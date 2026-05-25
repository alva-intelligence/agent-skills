# Runner Setup (Server-Side Contract)

`.github/workflows/review.yml` routes PR events to a self-hosted runner labelled `frndos-pr-review`. The runner executes two scripts: `pr-review.sh` (worker) and `lark-notify.sh` (notifier). This document is the **contract** between the workflow and those scripts — required env vars, stdout shape, side effects.

**Out of scope for this skill:** installing the runner, deploying the scripts, rotating bot tokens. Those are one-time infra tasks owned by the platform team. This document tells you what the scripts MUST do.

---

## Worker: `pr-review.sh`

### Required env vars (provided by the workflow)

| Var | Source | Use |
|-----|--------|-----|
| `PR_REPO` | `github.repository` | `owner/repo` slug |
| `PR_NUMBER` | `github.event.pull_request.number` | PR number |
| `PR_TITLE` | `github.event.pull_request.title` | Used in Lark card |
| `PR_URL` | `github.event.pull_request.html_url` | Used in Lark card |
| `PR_AUTHOR` | `github.event.pull_request.user.login` | Used in Lark card |
| `PR_BASE` | `github.event.pull_request.base.ref` | git fetch base |
| `PR_HEAD_SHA` | `github.event.pull_request.head.sha` | git fetch head |
| `PR_HEAD_REF` | `github.event.pull_request.head.ref` | git fetch head |

### Steps the script MUST execute

1. **Clone or fast-update** the PR repo to a workspace dir on the runner.
2. **Fetch** `origin/$PR_BASE` and `origin/$PR_HEAD_REF`.
3. **Locate** `REVIEW.md` and `KNOWLEDGE.md` at the PR head's repo root. If either missing → emit summary JSON with `error: "missing REVIEW.md"` / `"missing KNOWLEDGE.md"` and exit 0 (do NOT fail the workflow; missing docs are an onboarding issue, not a CI failure).
4. **Invoke `claude -p` twice**:
   - **Code review:** prompt = `REVIEW.md` content + `git diff origin/$PR_BASE...origin/$PR_HEAD_REF`. Capture stdout = single JSON object per REVIEW.md §A.1 (top-level fields: `verdict`, `diff_stats`, `counts`, `summary_markdown`, `inline_comments`).
   - **Knowledge review:** prompt = `KNOWLEDGE.md` content + same diff. Capture stdout = either `===SKIP===`/`===END SKIP===` sentinel per KNOWLEDGE.md §2.5 OR the §6 markdown template.
5. **Post comments via `gh api`** (bot account auth on the server):
   - Code review: each `.inline_comments[]` → `POST /repos/$PR_REPO/pulls/$PR_NUMBER/comments`. Then `.summary_markdown` → `POST /repos/$PR_REPO/issues/$PR_NUMBER/comments`.
   - Knowledge review: if `===SKIP===`, post nothing. Else, the §6 markdown → `POST /repos/$PR_REPO/issues/$PR_NUMBER/comments`.
6. **Print one JSON line to stdout** (consumed by the Lark notifier — see schema below). Exit 0 always.

### Stdout JSON schema (consumed by `lark-notify.sh`)

```json
{
  "repo": "owner/repo",
  "service": "api|web|ai|data|other",
  "pr_number": 127,
  "pr_title": "feat: super-admin module",
  "pr_url": "https://github.com/owner/repo/pull/127",
  "pr_author": "github-handle",
  "code_review": {
    "verdict": "approve|request-changes|comment-only",
    "counts": {"critical": 0, "high": 2, "medium": 4, "low": 1},
    "top_issues": ["one-line gist with path:line"],
    "summary_excerpt": "first 200 chars of summary"
  },
  "knowledge_review": {
    "emitted": true,
    "skip_reason": null,
    "tldr": "2-3 sentence TL;DR if emitted"
  },
  "error": null
}
```

When `===SKIP===` fired: `knowledge_review.emitted = false`, `skip_reason = <sentinel reason>`, `tldr = null`.

When the worker hits an unrecoverable error before reaching claude: emit JSON with `error: "..."` and all other fields best-effort. Notifier surfaces the error in the Lark card.

---

## Notifier: `lark-notify.sh`

### Input

Reads one JSON line from stdin (the worker's stdout, per schema above).

### Output

Posts a Lark interactive card to a fixed group via `lark-cli` (or direct webhook). Card layout:

- **Title:** `[<service>] PR #<pr_number>: <pr_title>`
- **Author:** `<pr_author>` (link to GitHub profile)
- **PR link:** `<pr_url>`
- **Code review section:**
  - Verdict with icon (✅ / 🔴 / 💬)
  - Counts row (icons + numbers)
  - Top issues (bullets, max 5 shown — link "see PR for more" when truncated)
  - Excerpt of summary
- **Knowledge review section:**
  - If `emitted: true` → TL;DR
  - If `emitted: false` → omit section entirely (per KNOWLEDGE.md §2.5)
- **Footer:** timestamp + commit SHA

Failure of the Lark post must NOT fail the workflow — the workflow step has `continue-on-error: true`. But the notifier should `>&2 echo` the error so it appears in the runner log.

---

## Bot auth on the server

The worker uses a long-lived `gh` token associated with a dedicated bot user (e.g. `alvaintelligent-owner`). Token scopes:

- `repo` (full — needed to post PR comments + review comments)
- `read:org` (to verify repo access)

Do NOT use `GITHUB_TOKEN` from the workflow — it's scoped to the actions context and lacks the right shape for posting bot-styled comments + creating reviews on behalf of the bot identity.

Rotate the token periodically. Store in the runner's keyring, not in plaintext on disk.

---

## Adding a new repo to the pipeline

1. Run this skill in the target repo: `/frndos-pr-review-setup +all`.
2. Push the generated `REVIEW.md`, `KNOWLEDGE.md`, `.github/workflows/review.yml`.
3. Enable the repo on the self-hosted runner group (org settings → Actions → Runner groups → add repo).
4. Confirm by opening a draft PR + marking ready-for-review; the workflow should fire and you should see both comments + a Lark card.

If the workflow fires but no comments appear: check the runner log for `gh api` errors (most likely the bot doesn't have access to the repo). Fix by adding the bot as a collaborator with write access.

If the workflow doesn't fire at all: check the repo is in the runner group + the workflow file is on the PR's base branch (workflows must exist on the base branch to run on PRs from that base).
