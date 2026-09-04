---
name: swarm
description: "Fan out N parallel workers, drain them, and return one report. Use for /swarm, 'swarm this', or parallel coverage, races, gauntlets, and exploration."
---

# Swarm

Fan out N parallel background workers. They may cover separate slices, race the same brief, or mix both. The parent waits, aggregates, and returns one report.

## Start

Open a todolist with one entry per phase before launching anything.

1. Frame
2. Fan out
3. Aggregate
4. Report

## Phase A: Frame

1. State the done predicate and the artifact or report the swarm must return.
2. Choose the shape. Partition into slices, race N workers on identical briefs, or mix both. For a race or mixed shape, declare `first pass`, `rank all`, or `best-of` before spawning.
3. Set N from the user or derive it from the shape. N is total workers, not a concurrency limit. Cap N at 8 per wave unless the user names a larger number; past that, run waves of 8 and drain each before the next. Don't swarm when one worker could finish the brief inside its own budget: run one `Agent` call, or do it yourself.
4. Worker model and effort per `~/.claude/skills/poteto-mode/references/models.md`, role `swarm workers`. For a model race, name each arm's model and effort up front.
5. Give each worker its own writable output when it writes. Use `isolation: worktree` on the Agent call, a branch, or `/tmp/swarm-<slug>/worker-<n>/`.

## Phase B: Fan out

Spawn all N workers in one message, one `Agent` call each:

- `subagent_type: general-purpose`
- `model` and `effort` from the `swarm workers` role line
- `run_in_background: true`
- `isolation: worktree` when the worker writes; `isolation: remote` only when a cloud runner is available and the worker needs nothing on the user's computer

When a worker must start from a non-default branch, name the branch in its brief; the worker checks it out inside its own worktree.

While the wave runs, the lead keeps working: lay out the result table with one row per slice or arm, and read each terminal result as its notification arrives instead of waiting for the whole wave.

Every brief stands alone. Include the goal, scope, exact slice or race arm, how to verify, and what to report. Reports use `PASS`, `ISSUES`, or `BLOCKED` with evidence.

If a worker drops out, proceed with N-1 and note it.

## Phase C: Aggregate

Read the terminal results. For coverage, every required slice needs a result. For a race, apply the selection rule declared up front. Use first pass, rank all, or best-of. Do not paste raw worker dumps.

Keep a compact result table, one-line evidenced issues, and explicit gaps or dropouts.

## Phase D: Report

Return one consolidated in-chat report with the table, issue one-liners, gaps or dropouts, and the race rule when used.
