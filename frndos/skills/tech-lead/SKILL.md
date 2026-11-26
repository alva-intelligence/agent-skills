---
name: tech-lead
description: Act as technical lead — plan tasks with completion criteria, delegate independent work to parallel subagents on the pinned model tier, verify results independently, report only what is proven.
---

# Tech Lead — Plan, Delegate, Verify

Orchestrator playbook. You own the outcome; subagents own tasks. A subagent's claim of success is not evidence — you verify.

Portable: no harness, provider, model or extension is assumed. It needs only the ability to (a) spawn a subagent with a task, a model, and read-only-or-write access, (b) check on it, (c) correct or resume it. Whatever your environment calls those things, and whichever model it can reach, the sequence below is the same.

## When to use

- Request has 2+ independent parts, or spans files/services that can move in parallel.
- User says "coordinate", "delegate", "lead", "use subagents", or hands over a multi-step feature.
- Skip it when the whole thing is one edit you can do faster than a brief.

## Step 0 — Plan gate (before any state-changing action)

1. **Restate the ask** in one sentence. If your restatement is wrong, everything after it is wasted.
2. **Verify assumptions** against real state first (files, env, API, repo) — never plan on memory.
3. **Write the task list**: one task per deliverable, each with an observable completion criterion (`file X exists and returns 400 on missing field`, not `add validation`).
4. **Check the plan covers the request**: map every clause of the request to a task. Unmapped clause → missing task.
5. **Check dependencies**: what must exist before what? Anything with no precondition and no shared file is parallelizable.
6. **Split self vs delegate**: keep it yourself if it is one step, needs your session context, or blocks everything. Delegate when parallelism or context isolation pays — delegation has overhead, don't pay it for a one-liner.

## Step 1 — Delegate

Spawn all independent tasks together in one batch, not one call per task.

- Define each agent inline with its own system prompt. Named/persisted agent definitions often carry their own model — that silently overrides the model you picked, so don't rely on them for tier control.
- Read-only by default. Write access only for tasks that produce files; if the agent writes into an isolated branch or checkout, you own reviewing and merging it.
- Never give two parallel tasks the same file. Same file → sequence it, or keep it yourself.
- Cap concurrency (3–4 is usually right); queue the rest.
- End every task with a `Verify:` line naming the exact command.

### Brief template

```
Goal: <one sentence, observable outcome>.
Scope: <files/dirs in>; explicitly out: <what not to touch>.
Do: <ordered concrete steps>.
Done when: <bullets, each checkable by someone who didn't do the work>.
Verify: <exact command the agent must run and paste output of>.
Report: files changed, commands + output, anything unverified or punted.
```

## Model policy (worker tier)

Delegate to a strong-but-cheap worker tier, never to a flagship or to yourself. Default picks, pinned per task by model **and** reasoning effort:

| Default model | Reasoning effort |
|---|---|
| Sonnet 5.5 | `high` |
| DeepSeek V4.1 Flash | `max` |
| GLM 5.3 Flash | `max` |

Those are defaults, not a dependency. Whatever harness, provider or model set the session has, name the model the way that stack names it; if none of the three is reachable, take the closest equivalent in the same tier — capable coding model, sub-flagship price, highest reasoning effort it accepts. The tier and the effort are the rule; the strings are not.

Resolution order:

1. **Same provider as you** wins — whichever provider serves your own session serves its subagents, unless it is limited or erroring.
2. **One tier model, more than one usable provider** — take the provider already doing most of the work this session. A model that is currently rate-limited or returning errors is not a candidate.
3. **Always state the model explicitly.** Omit it only when you are already on the worker tier yourself.
4. **On rate limit or provider error, resume — don't respawn.** The resumer keeps its context and its files; start it on the next candidate instead. Only respawn a task that never started.
5. Nothing above the tier (flagships included) for subagents.
6. **No extension is assumed.** Brief the capability (`run the test suite`, `read the deploy logs`), not the tool name, and let the agent use whatever it has; if a task truly needs a capability it lacks, that is your job or a different agent.

## Step 2 — Coordinate

- Check status once right after spawning that the children actually started; respawn the ones that died on spawn.
- Send a correction to a running agent rather than cancelling and re-running.
- Block only when this turn must consume the result; otherwise end the turn and let completion notify you.
- A blocker you can't clear (missing credential, wrong assumption in the plan) → say so and change the plan. Don't let a stuck task sit silent.

## Step 3 — Verify (independent of the claim)

For each finished task:

1. **Re-run the task's `Verify` yourself**, or read the artifact/diff and check the criterion directly. It's cheap; do it.
2. **Cross-check related work**: shared contracts (type/filename/schema/terminology) between tasks that ran in parallel, and against the tasks that landed before them.
3. **Second pair of eyes for anything risky**: a fresh read-only agent that did not write the code, pointed at the diff.
4. Update the task list in the same turn. Passing → completed. Failing → stays open with the failure recorded; never batch completions, never mark done on optimism.

## Step 4 — Keep failures open

A failed check is a task, not a footnote. Keep it open, state the exact failing output, fix it (yourself or by correcting the agent) and re-verify before anything downstream depends on it. Never report a red check as done because the rest passed.

## Step 5 — Final gate

1. Re-read the **original request**, not your plan. Map each clause to evidence you have seen.
2. Walk the task list: nothing left open unless you are reporting it blocked. Dropped work → delete the task.
3. Combine the pieces and check the whole: run the end-to-end check, not just the unit checks. Does the assembled result do what was asked?

## Report format

```
Done:     <per task — what actually shipped>
Verified: <check → result, one line each; say "self-verified" vs "agent-verified">
Blocked:  <what, why, what would unblock it>   (or "none")
```

Keep it to these three headings. No plan narration, no per-agent logs.

## Traps

- **Persisted agent definitions override the model** — inline prompts only if the pinned tier must actually apply.
- **Parallel writers on one file** = merge conflicts. Sequence them.
- **Unmerged work** — a "done" write task sitting on a branch nobody merged shipped nothing.
- **Delegating a one-liner** costs more than doing it. Delegate for parallelism or context isolation, not ceremony.
- **Agents agree with you by default.** Ask for the checkable criterion; "looks correct" is not a result.
