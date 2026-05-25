---
name: frndos-pr-review-setup
description: Scaffold PR review tooling (REVIEW.md, KNOWLEDGE.md, .github/workflows/review.yml) into any repo so it joins the central frndOS PR-review pipeline. Subcommands +review-md, +knowledge-md, +workflow, +all.
---

# frndos-pr-review-setup

Scaffold the three artefacts every repo needs to participate in the central PR-review pipeline:

| File | Purpose | Skill subcommand |
|------|---------|------------------|
| `REVIEW.md` | Repo's code-review playbook (quality + quantity + security). Stack-flavoured. Asks for official agent skills to pre-flight. | `+review-md` |
| `KNOWLEDGE.md` | Repo's high-level architecture / feature-impact challenge playbook. Stack-agnostic. | `+knowledge-md` |
| `.github/workflows/review.yml` | GitHub Actions workflow firing on PR events; routes to self-hosted runner labelled `frndos-pr-review`. | `+workflow` |

Run `+all` to scaffold all three in sequence.

---

## When to invoke

Trigger conditions:
- User says "set up PR review in this repo", "scaffold review docs", "add the review workflow", "/frndos-pr-review-setup".
- New repo onboarding into the frndOS PR-review pipeline.
- Existing repo missing one of the three files.

Do NOT invoke if the three files already exist AND the user hasn't asked for regeneration. Detect via `ls REVIEW.md KNOWLEDGE.md .github/workflows/review.yml`. If any exist, ask whether to overwrite, merge, or skip.

---

## Subcommand: `+review-md`

Generates `REVIEW.md` at repo root. Stack-flavoured + interactive.

### Steps

1. **Detect stack** from sentinel files at repo root:
   - `composer.json` + Laravel in dependencies → `php-laravel`
   - `package.json` + `next` in deps → `ts-nextjs`
   - `pyproject.toml` / `requirements.txt` + `fastapi` → `python-fastapi`
   - `pyproject.toml` + `clickhouse` / `pydantic` (no fastapi) → `python-data`
   - Else → ask user (free text: framework + language).

   Map to a stack profile in [`references/stack-profiles.md`](./references/stack-profiles.md).

2. **Detect default branch** via `git remote show origin | grep 'HEAD branch'` or `gh repo view --json defaultBranchRef --jq .defaultBranchRef.name`. Confirm with user.

3. **Ask the user for official skills/agents to pre-flight (§1.5).** This is mandatory — REVIEW.md's pre-flight section cannot be empty. Use the ask tool with multi-select. Offer:
   - Common per-stack defaults from `references/stack-profiles.md` (e.g. `laravel-simplifier`, `laravel-boost` review for php-laravel; `next-best-practices`, `tanstack-query` for ts-nextjs).
   - "Other" → free text for skill name + source (npx URL, github URL, or `skills.sh` slug).
   - "None" → §1.5 still rendered with detection logic but no pre-named skills.

   For each chosen skill, collect:
   - **Name** (slug used in detection match)
   - **Source** (one of: `skills.sh` URL, `npx skills add <repo>` URL, GitHub repo URL, "bundled with framework" e.g. laravel/boost)
   - **What it covers** (one-line, used to populate §1.5 table)
   - **Run order priority** (higher priority wins dedup collisions; default = list order)

4. **Load template** from [`references/review-md.template.md`](./references/review-md.template.md). Fill placeholders:
   - `{{STACK}}` — human-readable (e.g. `Laravel 13 · PHP 8.4 · Pest 4 · Pint`)
   - `{{DEFAULT_BRANCH}}` — e.g. `develop`
   - `{{REPO_SLUG}}` — e.g. `frnd-api-php`
   - `{{STYLE_TOOL}}` / `{{TYPE_TOOL}}` / `{{TEST_TOOL}}` — from stack profile
   - `{{OFFICIAL_SKILLS_TABLE}}` — table built from step 3
   - `{{OFFICIAL_SKILLS_RUN_ORDER}}` — ordered list
   - `{{RUBRIC_QUALITY}}` / `{{RUBRIC_QUANTITY}}` / `{{RUBRIC_SECURITY}}` — from stack profile
   - `{{FRAMEWORK_CHECKLIST}}` — from stack profile
   - `{{AUTH_PATHS}}` — repo-specific auth file paths (FormRequests + middleware + auth controllers — from stack profile + user confirmation)

5. **Write** to `<repo-root>/REVIEW.md`. If file exists, diff against template + ask: overwrite / merge / abort.

6. **Verify** line count ≤ 200 (the cap the source playbook uses). If over, the agent compressed the rubric tables too verbosely — re-trim.

7. **Print** one-line confirmation with file path + line count.

---

## Subcommand: `+knowledge-md`

Generates `KNOWLEDGE.md` at repo root. Stack-agnostic (KNOWLEDGE deals with concepts + diagrams, not framework idiom).

### Steps

1. **Detect stack + default branch** (same as `+review-md` step 1–2). KNOWLEDGE uses these for the header block + the §3.1 surface-area path hints (e.g. `routes/api.php` for Laravel vs `pages/api/*` for Next, vs `app/routes.py` for FastAPI).

2. **Load template** from [`references/knowledge-md.template.md`](./references/knowledge-md.template.md). Fill:
   - `{{STACK}}` / `{{DEFAULT_BRANCH}}` / `{{REPO_SLUG}}` — same as REVIEW
   - `{{SURFACE_AREA}}` — stack-specific path hints (from stack profile, §3.1 of template)
   - `{{EXTERNAL_DEPS}}` — list of external services the repo calls (from stack profile or repo's config — e.g. `Xendit, OpenAI, Pusher, ClickHouse` for the api repo). Ask user if unsure.
   - `{{STORAGE_BACKENDS}}` — MySQL / Postgres / ClickHouse / S3 / Redis etc. Ask user.

3. **Write** to `<repo-root>/KNOWLEDGE.md`. Diff/overwrite/abort prompt as in `+review-md`.

4. **Verify** line count ≤ 200.

5. **Print** confirmation.

---

## Subcommand: `+workflow`

Generates `.github/workflows/review.yml`.

### Steps

1. **Confirm runner config:**
   - `runs-on` labels (default: `[self-hosted, linux, frndos-pr-review]`)
   - Runner script path (default: `/var/www/pr-review.frndos.internal/bin/pr-review.sh`)
   - Lark notifier path (default: `/var/www/pr-review.frndos.internal/bin/lark-notify.sh`)
   - Timeout minutes (default: `20`)

   Ask user only if any value differs from defaults.

2. **Load template** from [`references/review-yml.template.yml`](./references/review-yml.template.yml). Fill:
   - `{{RUNNER_LABEL}}`, `{{RUNNER_SCRIPT_PATH}}`, `{{LARK_SCRIPT_PATH}}`, `{{TIMEOUT_MINUTES}}`
   - `{{STACK_SPECIFIC_IGNORES}}` — append `paths-ignore` patterns from the matched stack profile's "Workflow ignore patterns" block (see `stack-profiles.md`). Each pattern as one YAML list item indented 6 spaces: `      - 'pattern'`. Empty string OK if the stack has none.

3. **Verify** `.github/workflows/` exists; create if missing.

4. **Write** to `<repo-root>/.github/workflows/review.yml`. Diff/overwrite/abort prompt.

5. **Point user at** [`references/runner-setup.md`](./references/runner-setup.md) for the server-side script contract (what `pr-review.sh` must read + emit). This skill does NOT install the runner — that's a one-time infra task.

6. **Print** confirmation + reminder to enable the self-hosted runner on the repo.

---

## Subcommand: `+all`

Run `+review-md` → `+knowledge-md` → `+workflow` in sequence. Single confirmation prompt before starting. Stop on first failure / user abort.

---

## Hard rules

- **Never write without user confirmation** when the target file exists.
- **Never invent stack profiles.** If detection fails, ask the user.
- **Never invent official skills.** §1.5 in REVIEW.md is user-supplied (or "None") — do not hallucinate a `laravel-simplifier` if it isn't installed; the user must opt in by name + source.
- **Never modify** the source playbook in this repo (`references/*.template.md`). To fix a typo in the playbook, open a PR against `alva-intelligence/agent-skills` itself.
- **Line cap = 200** for both REVIEW.md + KNOWLEDGE.md. Compress rubrics if the generated file exceeds.
- **Stack profile additions** (new framework support) go in `references/stack-profiles.md` as a PR to this repo, not as an inline workaround.

---

## Output contract (for downstream agents)

When invoked non-interactively (e.g. by `orchestra` or a meta-skill), accept these env vars to skip questions:

| Env var | Effect |
|---------|--------|
| `FRNDOS_STACK` | Skip stack detection; one of `php-laravel`, `ts-nextjs`, `python-fastapi`, `python-data`, `custom:<name>` |
| `FRNDOS_DEFAULT_BRANCH` | Skip default-branch detection |
| `FRNDOS_OFFICIAL_SKILLS` | JSON array of `{name, source, covers, priority}` for §1.5 |
| `FRNDOS_RUNNER_LABEL` | Override runner label |
| `FRNDOS_RUNNER_SCRIPT_PATH` | Override runner script path |
| `FRNDOS_OVERWRITE` | `true` to skip overwrite confirmation |

When all required vars set, run silently. Emit JSON to stdout on success:

```json
{
  "wrote": ["REVIEW.md", "KNOWLEDGE.md", ".github/workflows/review.yml"],
  "skipped": [],
  "errors": []
}
```

---

## See also

- [`references/review-md.template.md`](./references/review-md.template.md) — REVIEW.md template + adaptation guide
- [`references/knowledge-md.template.md`](./references/knowledge-md.template.md) — KNOWLEDGE.md template
- [`references/review-yml.template.yml`](./references/review-yml.template.yml) — workflow template
- [`references/stack-profiles.md`](./references/stack-profiles.md) — supported stack registry
- [`references/runner-setup.md`](./references/runner-setup.md) — server-side runner contract
