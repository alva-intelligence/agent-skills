<!--
========================================================================
TEMPLATE: REVIEW.md
========================================================================
This file is a TEMPLATE. The skill renders it into the target repo's root.

Placeholders to fill ({{NAME}}):
  - {{REPO_SLUG}}              e.g. frnd-api-php
  - {{STACK}}                  e.g. "Laravel 13 · PHP 8.4 · Pest 4 · Pint"
  - {{DEFAULT_BRANCH}}         e.g. develop
  - {{STYLE_TOOL}}             e.g. Pint
  - {{TYPE_TOOL}}              e.g. PHPStan/Larastan
  - {{TEST_TOOL}}              e.g. Pest 4 (tests/Feature/*)
  - {{OFFICIAL_SKILLS_TABLE}}  rows for §1.5 detection table (user-supplied)
  - {{OFFICIAL_SKILLS_NAMES}}  comma-joined for "Not found" log line
  - {{AUTH_PATHS}}             comma-joined repo-specific auth paths
  - {{FRAMEWORK_NAME}}         e.g. Laravel 13

Blocks to ADAPT per stack (see stack-profiles.md):
  - §3.1 Code Quality rubric table
  - §3.2 Code Quantity rubric table (mostly generic — usually unchanged)
  - §3.3 Code Security rubric table
  - §4 Framework Checklist paragraph

For unrecognised stacks: leave the Laravel rubric as a reference + add a
TODO HTML comment at the top of the file saying the rubric needs adapting.

Strip this entire HTML-comment header before writing to the target repo.
========================================================================
-->
# REVIEW.md — PR Review ({{REPO_SLUG}})

> PR review playbook. Read full before output.
> **Stack:** {{STACK}}
> **Default branch:** `{{DEFAULT_BRANCH}}`

## 1. Scope

Flag 3 dimensions only: **Quality** (readability, structure, idiom, dead code, error handling), **Quantity** (diff size, scope creep, mixed concerns, over-engineering), **Security** (auth, authz, validation, injection, secrets, supply chain, data exposure, OWASP API Top 10). Skip what tooling catches: {{STYLE_TOOL}} (style), {{TYPE_TOOL}} (types), CI ({{TEST_TOOL}}).

## 1.5 Pre-flight — Official Skills First (if exist)

Two official skills are pre-flighted before this doc's rubric runs. Both authoritative — this doc layers on top.

**Detection** (per skill, order):

{{OFFICIAL_SKILLS_TABLE}}

**Run order when found:** for each detected skill in declared priority order, (1) run on diff + capture output. (N) then run this doc's rubric with all skill outputs in context, reference don't duplicate. (N+1) merge — inline = concat to single `inline_comments` array, tag body `source: <skill-name>` / `source: repo-review`, dedupe `(path, line, dimension)` priority follows skill order (highest priority wins collisions). Summary = prepend `### <skill-name> review` block (verbatim findings) per skill in order, then this doc + deltas. Verdict precedence: any official skill's `request-changes` is binding, this doc can escalate (add critical) never downgrade.

**Not found:** log top of summary which skills missing — e.g. `{{OFFICIAL_SKILLS_NAMES}} not detected — running repo rubric only.` Proceed solo with whichever subset detected. No simulation. Detection per-invocation.

## 2. Output Format

**Two-comment model (Mode A):** runner posts ONE root summary + each inline finding as separate line comment anchored to code. Inline findings NOT in root body.

### Mode A — PR Review (webhook → runner → `claude -p`)

#### A.1 — Output: single JSON object to stdout. No sentinels. Top-level schema:

```
{
  "verdict": "approve" | "request-changes" | "comment-only",
  "diff_stats": { "added": int, "removed": int, "files": int },
  "counts": {
    "by_severity": { "critical": int, "high": int, "medium": int, "low": int },
    "by_dimension": { "security": int, "quality": int, "quantity": int }
  },
  "summary_markdown": "<root-comment markdown per §A.2 — posted as PR issue comment>",
  "inline_comments": [ <objects per schema below>, ... ]
}
```

**Inline-comment schema:** `{path, line, side: RIGHT|LEFT, severity: critical|high|medium|low, dimension: quality|quantity|security, source: <official-skill-name>|repo-review, title (≤80 chars, runner-log only — NOT posted), body (markdown ≤1200 chars per body template), suggestion? (drop-in, exact whitespace)}`. Never `nit`.

**Body template** (mandatory — `<details>` keeps comment compact):

```markdown
<icon-tag-line>

**<one-sentence problem with backticked symbols>**

<details><summary>🪲 Proposed fix</summary>
<2–4 sentences: what to change + why. Reference idiom or OWASP id.>
</details>

<details><summary>📝 Committable suggestion</summary>

```<lang>
<drop-in replacement>
```
</details>

<details><summary>🤖 Prompt for AI Agents</summary>
<paragraph AI agent executes verbatim: file, line range, exact change, what NOT to touch>
</details>
```

**Icon-tag line** — applicable tags ` | `-separated italic, order kind → severity → effort:

| Tag | Use for |
|-----|---------|
| `⚠️ _Potential issue_` | Bug/regression/latent defect |
| `🛡 _Security_` | §3.3 |
| `🧹 _Refactor_` | Quality/idiom |
| `🧩 _Scope_` | Quantity/scope-creep |
| `🛑 _Critical_`/`🔴 _High_`/`🟡 _Medium_`/`🔵 _Low_` | severity |
| `⚡ _Quick win_` | Fix ≤5 lines, no ripple |
| `🧵 _Needs thread_` | Needs author judgement |

Skip `🪲 Proposed fix` if problem statement describes fix in one line. Skip `📝 Committable suggestion` if no single-hunk drop-in. `🤖 Prompt for AI Agents` always.

#### A.2 — Root summary (rendered into `summary_markdown` field)

High-signal overview, NOT inline-finding dump. Posted verbatim as PR root issue comment.

```markdown
## 🔍 Code Review

**Verdict:** <✅ Approve | 🔴 Request changes | 💬 Comment-only>
**Diff:** +<added> −<removed> across <N> files

### Findings at a glance
🛑 Critical: `<n>` · 🔴 High: `<n>` · 🟡 Medium: `<n>` · 🔵 Low: `<n>`
🛡 Security: `<n>` · 🧹 Quality: `<n>` · 🧩 Quantity: `<n>`

### 🎯 Top issues to address first
1. <pointer: gist + `path:line` + "see inline comment">

(Every critical/high gets bullet, no cap. Skip if zero.)

### 🛡 Security notes
- <bullet, only if dependency manifests / lock files / auth / routes touched. Else: "_Nothing flagged._">

### ✨ What's good
- <bullet per nice pattern, no cap. Skip if nothing stands out; do NOT fabricate.>

### ➡️ Suggested next steps
1. <action>

_Source: `REVIEW.md` · Inline findings posted as line comments on the code._
```

**Hard rules:** no per-finding detail in root, no bullet-per-finding under dimensions (use counts table), top-issues = pointers only, bullets uncapped, never truncate findings.

### Caveman compression (if skill available)

If `caveman` skill detected, render bullets + list items in caveman-full: drop articles, fragments OK, short synonyms. Descriptions (verdict, tables, footer) stay plain English. Paths/errors/symbols/labels verbatim. Detection: (1) `caveman` in loaded skills, (2) `~/.claude/skills/caveman*`, (3) `.claude/skills/caveman*`. None → plain English.

### Mode B — Ad-hoc

Emit same JSON object. Local renderer prints `summary_markdown` first, then `inline_comments` as markdown table grouped by file. No PR posting.

## 3. Review Dimension Rubrics

<!-- ADAPT TABLES BELOW per stack. See stack-profiles.md.
     The embedded rows are the Laravel reference. For other stacks,
     replace rows with stack-appropriate checks. -->

### 3.1 Code Quality

| Issue | Sev | Notes |
|-------|-----|-------|
| Business logic in controller > 30 lines | medium | Extract to service / action layer. |
| Fat model — scopes + mutators + domain > 400 LOC | medium | Service or Repository split. |
| New N+1 query (loop over models calling relation accessor) | high | Use eager-load equivalent. Reference loop line. |
| Repeated literal strings/numbers as identifiers | medium | Move to enum / constants. |
| Debug calls left in non-test code (`dd()`, `console.log`, `print()`, etc.) | high | Remove before merge. |
| Empty catch swallowing exceptions | high | Rethrow, log to error reporter, or document. |
| New Job/Listener/Event without test on dispatch path | medium | Per repo's testing policy. |
| Repository bypass — direct ORM call where Repository exists | medium | Route through Repository. |
| Resource returning model directly without transform | low | Explicit fields; no full attribute leak. |

**Don't flag:** {{STYLE_TOOL}} auto-fix (spacing/imports/braces/quotes), naming when idiomatic + clear, "consider extracting" when fn < 30 lines + reads well.

### 3.2 Code Quantity

| Issue | Sev | Notes |
|-------|-----|-------|
| Diff > 800 net LOC across > 15 files | medium | Recommend split. State seams. |
| Two unrelated concerns in one PR | high | Extract drive-by to own PR. |
| Migration + backfill + controller + cron together, no feature flag | high | Cannot revert independently. Split or gate. |
| New abstraction (interface + abstract + 2 impls) for single caller | medium | YAGNI — inline until 2nd caller. |
| Half-finished (TODO, `not implemented` throws, commented-out blocks) | high | Complete or remove. |

**Don't flag:** mechanical large diffs (generated, lockfile, mass rename), long-but-cohesive PRs description justifies.

### 3.3 Code Security

Map every finding to OWASP API Top-10. Default **high** unless contained.

| Issue | Sev | OWASP | Check |
|-------|-----|-------|-------|
| New route without auth middleware + no explicit no-auth justification | critical | API2 | Read route. |
| Query interpolating user-supplied values without parameter binding | critical | API8 | Use bindings. |
| Raw SQL (`whereRaw`/`DB::raw`/`statement`/`$queryRawUnsafe`/string-concat SQL) | critical | API8 | Must use parameter bindings. |
| Storage write where path derives from user input without `basename()`/sanitisation | high | API1 | Path traversal. |
| File read / include / unserialize from user input | critical | API8/API10 | RCE / object injection. |
| Mass assignment without explicit field allowlist + validated request schema | high | API6 | Require validated DTO. |
| Missing authorization check in request / route guard | high | API1/API3 | Tie to policy. |
| Response returning sensitive fields (password hashes, tokens, internal IDs, payment, PII) not previously exposed | high | API3 | Diff against `$hidden` / existing schema. |
| Secret committed (`sk-…`, `xoxb-…`, `AKIA…`, `ghp_…`, JWT, PEM headers, `.env` content) | critical | — | Rotate immediately + rewrite history. |
| Manifest adds dependency from vendor not in lockfile | high | supply chain | Verify maintainer, release date, stars, CVEs. Org has prior supply-chain incidents — guilty until proven otherwise. |
| Lockfile changed without manifest change | high | supply chain | Silent transitive pin shift possible. Demand explanation. |
| `CORS` widened (`allowed_origins: ['*']` + `supports_credentials: true`) | high | API8 | Reject. |
| Rate-limiter removed from sensitive route (`login`, `register`, `password.*`, `otp.*`, auth controllers) | high | API4 | Restore. |
| Sensitive logged: full request payload, user object, auth tokens, payment, OTP | high | API3/GDPR | Mask or drop. |
| Env-var read directly outside config layer | medium | — | Move to config; breaks on config-cache. |
| Session/cookie config changed (`same_site`/`secure`/`http_only`/`domain`) | high | API2 | Verify intent. |
| Webhook endpoint without signature verification | critical | API2/API8 | Reject. |
| External HTTP without timeout | medium | API4 | Set timeout. |
| Background job public property holding full ORM model | medium | API3 | Pass ID; re-fetch in handler. |
| State-changing POST/PUT/DELETE bypassing CSRF when touching sessions | high | API8 | Verify. |

**Always:** grep diff for `password`, `token`, `secret`, `api_key`, `private_key` — flag unless safe by context. If any of {{AUTH_PATHS}} touched → verdict cannot be `approve` without explicit security paragraph. If lockfile changed, list each net-new package by name + one-line risk.

## 4. Framework Checklist ({{FRAMEWORK_NAME}})

<!-- ADAPT this paragraph per stack. The embedded text is Laravel — replace
     with framework-specific best-practice list. See stack-profiles.md. -->

Controllers thin → service / action layer. Request validation lives in dedicated request objects, no inline validation for non-trivial rules. Responses shaped by Resource/serializer (never raw models). Policies/guards gate authorised actions. Enums replace magic strings. Events/Listeners/Jobs/Observers in their respective dirs, wired in service provider / equivalent. Migrations reversible (down/rollback works) unless destructive; large-table columns use chunked backfills. Models declare explicit allowlists + casts + hidden sensitive fields. Background jobs use queue interface + sensible retry/backoff (no infinite retries). Error reporter receives exceptions (don't silence). Observability/dev tools env-gated, not enabled in prod.

## 5. Severity → Verdict

| Severity | Use |
|----------|-----|
| `critical` | Breaks prod, leaks data, account takeover. **Blocks merge.** |
| `high` | Significant defect; incident within weeks. **Blocks merge.** |
| `medium` | Real problem. **Request changes** but doesn't block if author justifies. |
| `low` | Worth fixing while code open; not blocker. |

Any `critical`/`high` → `request-changes`. Only `medium`/`low` → `comment-only`. Zero findings → `approve`.

## 6. Tone

Terse + specific. Problem one sentence, fix one sentence. Never "consider"/"perhaps"/"might". Never restate diff. Never invent findings (empty review valid). Never refactor outside diff scope. Quote errors/paths/lines verbatim. Suggesting framework idiom → link {{FRAMEWORK_NAME}} docs.

## 7. Execution Recipe

(1) `git fetch origin <base> <pr-branch>` (runner done). (2) `git diff origin/<base>...origin/<pr-branch>` = review surface. (3) pre-flight §1.5: detect each declared official skill, run each found, capture. (4) run rubric §3 with official outputs in context; merge per §1.5. (5) compute `verdict` + `counts` + `diff_stats`, render `summary_markdown` per §A.2, assemble `inline_comments` array, emit single JSON object to stdout, exit `0` always — runner reads `verdict` directly from JSON.

## 8. Out of Scope

Architectural challenge / feature-impact → `KNOWLEDGE.md`. Tests → CI. Style → {{STYLE_TOOL}}. Deployment → repo's release workflow.
