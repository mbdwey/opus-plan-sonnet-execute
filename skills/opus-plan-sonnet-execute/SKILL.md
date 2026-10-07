---
name: opus-plan-sonnet-execute
description: Use when a request is large or multi-part - touches many files or modules, bundles several features or fixes, needs a long build-out, a big refactor, broad research, or would take dozens of tool calls - or when the user asks to save tokens, cut cost, or delegate work to Sonnet. Do NOT use for small, single-file, or quick-answer requests.
---

# Opus Plans, Sonnet Executes

## Core principle
Opus tokens cost about 2x Sonnet's. Spend Opus on judgment: scoping, planning, writing briefs, reviewing. Spend Sonnet on volume: reading, editing, running, searching.

## Gate: delegate or do it inline?
Delegate when BOTH are true:
- The work splits into 2+ tasks that can each be described in a brief, OR one task needs more than ~15 tool calls.
- Each task can own its files without editing the same files as another task.

Shared files that every task would touch (package exports, registries, routes, config) are not assigned to any worker. Opus edits them in the Integrate step.

Otherwise do it inline. Each worker starts cold and rebuilds context, so delegating small work costs more than it saves.

## Workflow
1. **Scope cheaply.** Read only what the plan needs. If mapping the codebase itself takes more than a few reads, dispatch one `sonnet-worker` to map it and report back.
2. **Plan.** Split the work into tasks. Each task gets an owner of disjoint files, its dependencies, and a way to check it is done. Post the plan to the user as a short numbered list, then proceed. Stop and ask only when a requirement is ambiguous or a step is destructive.
3. **Brief.** Write one brief per task using the template below. Every field is REQUIRED.
4. **Dispatch.** Call Agent with `subagent_type: "sonnet-worker"`. Send independent tasks in ONE message so they run in parallel; dispatch dependent tasks after their inputs land. If `sonnet-worker` is unavailable, use `subagent_type: "general-purpose"` with `model: "sonnet"`. Pass `effort: "medium"` for mechanical tasks (renames, boilerplate, docs); leave the default (high) otherwise. Never pass `max`.
5. **Review.** Check each report against its acceptance criteria. Spot-check the diff; do not redo the work. If something is wrong, SendMessage the same worker with the specific fix, since it keeps its context.
6. **Integrate.** Run the full verification once (build, tests, lint) and report the outcome to the user.

## Brief template
```
GOAL: <one sentence: the outcome, not the steps>
CONTEXT: <what the worker cannot know: relevant decisions, conventions, paths, APIs, with file:line refs>
OWNED FILES: <files this task may create or edit; everything else is read-only>
STEPS: <numbered, concrete; name functions, commands, expected outputs>
ACCEPTANCE: <commands to run and what passing looks like>
OUT OF SCOPE: <adjacent things NOT to touch>
REPORT: status (done/blocked), files changed, verification output (last lines only), beyond the brief (anything done that the brief did not ask for, or "none"), open questions. Under 200 words.
```

## Common mistakes
| Mistake | Fix |
|---|---|
| Brief says "see above" or "as discussed" | Workers never saw the conversation. Paste the facts in. |
| Two tasks edit the same file | Merge them, or sequence them. |
| Opus re-reads every changed file in review | Trust passing acceptance checks; spot-check only. |
| Opus fixes a worker's bug itself | SendMessage the worker with the fix. |
| Delegating a 3-line change | Do it inline. |
| Worker returns a long narrative | The REPORT field caps it; restate the cap if it's ignored. |
