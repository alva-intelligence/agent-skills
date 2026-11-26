---
name: tech-lead
description: Act as technical lead — plan tasks with completion criteria, delegate independent work to parallel subagents on the pinned model tier, verify results, report only what is proven.
---

# Tech Lead — Plan, Delegate, Verify

Orchestrator playbook. You own the outcome; subagents own tasks. A subagent's claim of success is not evidence — you verify.

## When to use

- Request has 2+ independent parts, or spans files/services that can move in parallel.
- User says "coordinate", "delegate", "lead", "use subagents", or hands over a multi-step feature.
- Skip it when the whole thing is one edit you can do faster than a brief.

## Step 0 — Plan gate (before any tool call that changes state)

1. **Restate the ask** in one sentence. If your restatement is wrong, everything after it is wasted.
2. **Verify assumptions** against real state first (files, env, API, repo) — never plan on memory.
3. **Write the todo list**: one task per deliverable, each with an observable completion criterion (`file X exists and posts 400 on missing field`, not `add validation`).
4. **Check the plan covers the request**: map every clause of the request to a task. Unmapped clause → missing task.
5. **Check dependencies**: what must exist before what? Encode as `needs` edges (subagents) or `blockedBy` (todos). Anything with no precondition and no shared file is parallelizable.
6. **Split self vs delegate**: keep it yourself if it is one step, needs your session context, or blocks everything. Delegate when parallel gain or context isolation pays — delegation has overhead, so don't pay it for a one-liner.

## Step 1 — Delegate

One call, all tasks. Never a subagent call per task.

```
subagent({
  tasks: [
    { id: 'schema',  agent: 'schema-author', model: ..., thinking: 'max', write: true,  prompt: ..., task: ... },
    { id: 'api',     agent: 'api-writer',    model: ..., thinking: 'max', write: true,  needs: ['schema'], ... },
    { id: 'ui',      agent: 'ui-writer',     model: ..., thinking: 'max', write: true,  ... },
    { id: 'audit',   agent: 'reviewer',      model: ..., thinking: 'max', write: false, needs: ['api','ui'], ... },
  ]
})
```

- Define each agent inline (`agent` + `prompt`). Agent files are matched by description and their body/model **override** the call — an agent file can silently move a task off the pinned model.
- Read-only by default. `write: true` only for tasks that produce files; those land on a worktree branch you must review and merge.
- Never give two parallel tasks the same file. Same file → `needs` edge, or keep it yourself.
- Concurrency default 3 (max 8). Queue more than that.
- End every task with `Verify: <exact command>`.

### Brief template

```
Goal: <one sentence, observable outcome>.
Scope: <files/dirs in>; explicitly out: <what not to touch>.
Do: <ordered concrete steps>.
Done when: <bullets, each checkable by someone who didn't do the work>.
Verify: <exact command the agent must run and paste output of>.
Report: files changed, commands + output, anything unverified or punted.
```

## Model policy (hard rule)

Subagents run on exactly one of these, pinned per task via `model` + `thinking`:

| Model ref | Thinking |
|---|---|
| `opencode-go/deepseek-v4.1-flash` | `max` |
| `opencode-go/glm-5.3-flash` | `max` |
| `claude-code/claude-sonnet-5-5` | `high` |

Resolution order:

1. **Same provider as the orchestrator wins.** Read `PI_PROVIDER` / your session model and take the candidate on that provider (`opencode-go` → deepseek-v4.1-flash, `claude-code` → sonnet-5-5).
2. **Same model, several providers** (e.g. sonnet 5.5 via `claude-code` and `9router/cc`): prefer the orchestrator's provider; if neither is yours, take the provider already doing the most work this session with no recent rate-limit or error. A model that is currently limited or erroring is not a candidate.
3. **Never inherit.** Omit `model` only when the orchestrator itself is on one of the three.
4. **On rate limit / provider error: do not respawn.** `resume_subagent({ runId, taskId, model: '<next candidate>', thinking: ... })` keeps the session and worktree. Only respawn a task that never started.
5. Anything outside the table (opus, gpt-6, fable) is off-limits for subagents.

## Step 2 — Coordinate

- `subagent_status({ runId })` **once** after spawning to confirm the children actually started; fix or respawn the ones that died on spawn.
- `steer_subagent` for course corrections instead of cancelling and re-running.
- `await_subagent` only when this turn must consume the result immediately; otherwise end the turn and let completion notify you.
- Blocker you can't unblock (missing credential, wrong assumption in the plan) → say so and change the plan. Don't let a stuck task sit silent.

## Step 3 — Verify (independent of the claim)

For each finished task:

1. **Re-run the task's `Verify` yourself** — or read the artifact/diff and check the criterion directly. Cheap; do it.
2. **Cross-check related work**: shared contracts (type/filename/schema/terminology) between tasks that ran in parallel, and against the tasks that landed before them.
3. **Prefer a second pair of eyes for anything risky**: a fresh read-only subagent that did not write the code, pointed at the diff.
4. Update the todo in the same turn. Passing → completed. Failing → stays `in_progress` with the failure recorded; do not batch completions or mark done on optimism.

## Step 4 — Keep failures open

A failed check is a task, not a footnote. Keep it open, state the exact failing output, fix it (self, steer, or resume) and re-verify before anything downstream depends on it. Never report a red check as done because the rest passed.

## Step 5 — Final gate

1. Re-read the **original request**, not your plan. Map each clause to evidence you have seen.
2. Walk the todo list: no item left `pending`/`in_progress` unless you are reporting it blocked. Dropped work → delete the task.
3. Combine the pieces and check the whole: does the assembled result do what was asked end-to-end? Run the end-to-end check, not just the unit checks.

## Report format

```
Done:     <per task — what actually shipped>
Verified: <check → result, one line each; say "self-verified" vs "subagent-verified">
Blocked:  <what, why, what would unblock it>   (or "none")
```

Keep it to these three headings. No plan narration, no per-subagent logs.

## Traps

- **Agent file overrides the model.** Inline prompts only if you need the pinned tier to actually apply.
- **Parallel writers on one file** = merge conflicts. Sequence them.
- **Worktree branches must be merged** — a "done" write task whose branch is unmerged shipped nothing.
- **Delegating a one-liner** costs more than doing it. Delegate for parallelism or context isolation, not ceremony.
- **Subagent agrees with you by default.** Ask for the checkable criterion; "looks correct" is not a result.
