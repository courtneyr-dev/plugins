# pstack roles → Claude Code models

Every ported skill says "model per `~/.claude/skills/poteto-mode/references/models.md`, role <name>" instead of naming a model. Look the role up in the table and pass its `model` and `effort` on the `Agent` call. A role line here overrides the skill's inline default; `/setup-pstack` rewrites the table. Installed path: `~/.claude/skills/poteto-mode/references/models.md`. Skills read the file on each run; no restart.

`Agent` `model` accepts `sonnet`, `opus`, `haiku`, `fable`. `effort` accepts `low`, `medium`, `high`, `xhigh`, `max`. Two aliases are valid in any cell:

- `inherit` (upstream spellings `inherit-parent`, `auto`): omit `model` on the Agent call; the role runs on the parent session's model.
- `counselors`: the slot runs through the `counselors` skill (Codex and Gemini CLIs), not as an Agent call. Use it where the upstream role was a second opinion or adversarial review.

Panel roles (arena runners, arena cross-judge pool, architect runners, interrogate reviewers) hold ordered lists: one Agent call per entry, aliases included, so list length sets the fan-out. Arena picks one entry from its cross-judge pool. `swarm workers` is the default for every swarm worker unless a race names another per arm.

## Upstream string → port

| Upstream string | Port |
|---|---|
| `claude-fable-5-thinking-max`, `claude-fable-5-1-thinking-max` | `model: fable`, `effort: max` (judgment, prose, hardest tasks) |
| `claude-opus-5-5-max` (judgment, prose, hardest tasks since upstream 0.15.3) | `model: opus`, `effort: max` |
| `claude-opus-5-thinking-xhigh` | `model: opus`, `effort: xhigh` |
| `grok-4.6-fast-xhigh`, `grok-4.7-xhigh-fast` (fast code, exploration) | `model: sonnet`, `effort: high` |
| `gpt-5.6-sol-max` (tooling, second opinion) | not a Claude Code subagent model. A second-opinion or adversarial-review slot becomes `counselors`; any other role takes `model: opus`, `effort: xhigh`; a runner panel drops the slot (three Claude runners) |
| `inherit-parent`, `auto` | `inherit` (omit `model`) |

## Roles

| Role | Used by | `model` | `effort` | Upstream default |
|---|---|---|---|---|
| code | poteto-mode, Subagents section (the fast code model; trivial mechanical edits) | sonnet | high | `grok-4.7-xhigh-fast` |
| prose | poteto-mode, Subagents section | opus | max | `claude-opus-5-5-max` |
| judgment | poteto-mode, Subagents section | opus | max | `claude-opus-5-5-max` |
| hardest tasks | poteto-mode, Subagents section (strongest judgment model and strongest instruction-following model) | opus | max | `claude-opus-5-5-max` |
| feature | poteto-mode `playbooks/feature.md`, step 4 | sonnet | high | `grok-4.7-xhigh-fast` |
| refactoring | poteto-mode `playbooks/refactoring.md`, step 5 | sonnet | high | `grok-4.7-xhigh-fast` |
| bug-fix | poteto-mode `playbooks/bug-fix.md`, step 3 | fable | max | `claude-fable-5-1-thinking-max` |
| perf-issue | poteto-mode `playbooks/perf-issue.md`, step 3 | fable | max | `claude-fable-5-1-thinking-max` |
| hillclimb | poteto-mode `playbooks/hillclimb.md`, step 5 | fable | max | `claude-fable-5-1-thinking-max` |
| how explorer | how, Step 2a (`subagent_type: Explore`) | sonnet | high | `grok-4.7-xhigh-fast` |
| how explainer | how, Steps 2b and 3 (`subagent_type: Explore`) | opus | max | `claude-opus-5-5-max` |
| why investigators | why, Step 3 (`subagent_type: general-purpose`; needs MCP tools) | sonnet | high | `grok-4.7-xhigh-fast` |
| why synthesizer | why, Step 4 (`subagent_type: general-purpose`) | opus | max | `claude-opus-5-5-max` |
| interrogate reviewer A | interrogate, Step 3 (`subagent_type: Explore`) | opus | max | `claude-opus-5-5-max` |
| interrogate reviewer B | interrogate, Step 3; adversarial second opinion | counselors | | `gpt-5.6-sol-max` |
| interrogate reviewer C | interrogate, Step 3 (`subagent_type: Explore`) | sonnet | high | `grok-4.7-xhigh-fast` |
| reflect judgment | reflect, step 2 (`subagent_type: general-purpose`) | opus | max | `claude-opus-5-5-max` |
| reflect tooling | reflect, step 2 (`subagent_type: general-purpose`); a tooling lens, not a second opinion, so it takes the non-counselors port | opus | xhigh | `gpt-5.6-sol-max` |
| reflect divergent | reflect, step 2 (`subagent_type: general-purpose`) | opus | max | `claude-opus-5-5-max` (upstream line `reflect judgment, divergent, synthesizer`) |
| reflect synthesizer | reflect, step 3 (`subagent_type: general-purpose`) | opus | max | `claude-opus-5-5-max` (upstream line `reflect judgment, divergent, synthesizer`) |
| arena runners (panel) | arena, Phase B, one Agent call per entry (two by default, up to five) | 1. opus, max. 2. sonnet, high | | `claude-opus-5-5-max`, `gpt-5.6-sol-max`, `grok-4.7-xhigh-fast` (the gpt slot has no Agent model; add an `opus, xhigh` entry to keep three runners) |
| arena cross-judge pool (panel) | arena, Phase C; pick one entry, prefer one unlike the parent's model | 1. opus, max. 2. counselors. 3. sonnet, high | | same three upstream slugs |
| architect runners (panel) | architect, one Agent call per entry (two by default) | 1. opus, max. 2. sonnet, high | | same three upstream slugs (gpt slot dropped, as for arena runners) |
| swarm workers | swarm, step 4 (every worker unless a race names another per arm); poteto-mode `playbooks/multi-phase-plan.md` live lanes | sonnet | high | `grok-4.7-xhigh-fast` |
| recall miners | recall, step 3 ("a fast, cheap model" for transcript slices) | haiku | medium | none named upstream |
| trail reviewer | show-me-your-work, cross-model review of the trail | counselors; if unavailable, a Claude model other than the one that did the work | | none named upstream ("a different model family") |

The `interrogate reviewers` panel is reviewer A through C above, in that order. Shrink or extend the panel by deleting or adding reviewer lines; the skill relabels to match.

## Different model family

Claude Code subagents are all Claude models. Where a skill asks for "a different model family" (show-me-your-work's trail reviewer, orchestrate's per-unit verifier and tool-error retry, eval's blinded judge, arena's cross-judge, figure-it-out's arena judge), use a `counselors` entry, and fall back to a Claude model other than the one that did the work.

## Effort on `Explore` runs

`~/.claude/agents/Explore.md` pins `effort: low` in its frontmatter. Roles that run as `subagent_type: Explore` (how explorer, how explainer, interrogate reviewers A and C) need the Agent call's `effort` to override that pin; if the call's `effort` does not take, state the effort in the prompt or run the role as `general-purpose`.
