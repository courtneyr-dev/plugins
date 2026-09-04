---
name: setup-pstack
description: Configure which model and effort pstack uses per role by rewriting the pack's ~/.claude/skills/poteto-mode/references/models.md. Use for /setup-pstack, "configure pstack models", or changing pstack's model choices.
---

# Setup pstack

Rewrite the role table in `~/.claude/skills/poteto-mode/references/models.md` (installed: `~/.claude/skills/poteto-mode/references/models.md`). Every pstack skill reads it at start and falls back to its inline default when a role line is absent: an override layer, not a requirement.

## Steps

1. **Detect.** The Agent tool's `model` set is `sonnet`, `opus`, `haiku`, `fable`; `effort` is `low`, `medium`, `high`, `xhigh`, `max`. Confirm both against the tool's schema in this session. Two aliases always pass: `inherit` (omit `model`; the role runs on the parent session's model) and `counselors` (the slot runs through the counselors skill). Never write a value outside these sets.
2. **Load.** Read the current table; it is the current state.
3. **Map and confirm.** Show every role with its model and effort. Ask with `AskUserQuestion` whether to keep the table or change specific roles. Panel roles (how critics, arena runners, arena cross-judge pool, architect runners, interrogate reviewers) hold lists; one Agent call per entry, aliases included, so list length sets the fan-out. Arena picks one entry from its cross-judge pool. `swarm workers` is the default for every worker unless a race names another per arm.
4. **Validate.** Every model and effort must be in step 1's sets; aliases pass. Otherwise stop and ask again: a row the Agent tool rejects breaks every delegation that reads it.
5. **Write.** Rewrite the whole table so re-runs stay idempotent; keep the header and notes.
6. **Confirm.** Report the written table. Skills read it on each run; no restart.
7. **Offer verification.** If the project has no `verify-*` skill or harness, offer once to generate one with `/create-verification-skill`.
