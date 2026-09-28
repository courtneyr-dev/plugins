---
name: setup-pstack
description: Configure which model and effort pstack uses per role, and at what reasoning budget, by rewriting the pack's ~/.claude/skills/poteto-mode/references/models.md. Use for /setup-pstack, "configure pstack models", "pstack budget", or changing pstack's model choices.
---

# Setup pstack

Rewrite the role table in `~/.claude/skills/poteto-mode/references/models.md` (installed: `~/.claude/skills/poteto-mode/references/models.md`).

## Steps

1. **Detect.** The Agent tool's `model` set is `sonnet`, `opus`, `haiku`, `fable`; `effort` is `low`, `medium`, `high`, `xhigh`, `max`. Confirm both against the tool's schema in this session. Two aliases always pass: `inherit` (omit `model`; the role runs on the parent session's model) and `counselors` (the slot runs through the counselors skill). Never write a value outside these sets.
2. **Load.** Read the current budget line and table; they are the current state. A role that is not in the table the pack ships, such as `how critics`, is from a retired role. Drop it.
3. **Budget, map, and confirm.**

   **(a) Ask for a budget.** Ask with `AskUserQuestion`. Offer these four options with these exact labels, and name the current budget when the table records one.

   - `unlimited — keep max`
   - `large — xhigh reasoning`
   - `medium — high reasoning`
   - `small — medium reasoning`

   **(b) Apply it.** Build the working table from the role table the pack ships (the defaults), and on a re-run keep any role you changed by model, list, or alias (`inherit`, `counselors`). `unlimited` leaves every effort as in that table. `large`, `medium`, and `small` set the `effort` of every Agent entry, panel entries included, to `xhigh`, `high`, or `medium`. If the Agent tool rejects that effort for a model, use the highest effort at or below the target that it accepts, else mark the role as needing a choice. `inherit` and `counselors` entries do not change. So `small` turns `opus, max` into `opus, medium`, and `sonnet, high` into `sonnet, medium`.

   **(c) Show the roles and confirm.** Show every role with its model and effort. Also list each role step 2 dropped. Ask with `AskUserQuestion` whether to keep the table or change specific roles. Panel roles (arena runners, arena cross-judge pool, architect runners, interrogate reviewers) hold lists; one Agent call per entry, aliases included, so list length sets the fan-out. Arena picks one entry from its cross-judge pool. `swarm workers` is the default for every worker unless a race names another per arm.
4. **Validate.** Every model and effort must be in step 1's sets; aliases pass. Otherwise stop and ask again.
5. **Write.** Rewrite the whole table so re-runs stay idempotent, with a budget line above it that names the chosen label and its target effort (`Budget: unlimited (max)`); keep the header and notes.
6. **Confirm.** Report the written table. Skills read it on each run; no restart.
7. **Offer verification.** If the project has no `verify-*` skill or harness, offer once to generate one with `/create-verification-skill`.
