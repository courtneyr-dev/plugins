# pstack for Claude Code

A port of [pstack](https://github.com/cursor/plugins/tree/main/pstack) (Lauren Tan's Cursor plugin, v0.14.8) to Claude Code vocabulary. Prose, playbooks, and principles are upstream's; only dispatch syntax, model strings, paths, and frontmatter changed. Per-group porting notes are in `NOTES-*.md`.

## Vocabulary

| Cursor | Claude Code |
|---|---|
| `Task` tool, `subagent_type: generalPurpose` | `Agent` tool, `subagent_type: general-purpose` |
| `readonly: true` | `subagent_type: Explore` |
| `"poteto-agent"`, `"Comment Sicko"` | `poteto-agent`, `comment-sicko` in `agents/` |
| `AskQuestion` | `AskUserQuestion` |
| model slugs per skill | `model` + `effort` per role in `~/.claude/skills/poteto-mode/references/models.md` |
| `gpt-5.6-sol-max` as second opinion | the `counselors` skill (Codex, Gemini) |
| `~/.cursor/rules/pstack-models.mdc` | `~/.claude/skills/poteto-mode/references/models.md`, rewritten by `/setup-pstack` |
| `~/.cursor/skills/…` | `~/.claude/skills/…` |
| Cursor agent transcripts | `~/.claude/projects/<slug>/<uuid>.jsonl` |

## Install with skill-sync

Push this tree to a GitHub repo, then register one entry per skill and adopt it:

```bash
SYNC=~/.claude/skills/skill-sync/sync_skills.py
python3 $SYNC add how <owner>/<repo> --path skills/how && python3 $SYNC adopt how
```

Register `references` and `agents` the same way with `--dest ~/.claude/skills/pstack/references` and `--dest ~/.claude/skills/pstack/agents`, then symlink the two agent files into `~/.claude/agents/`. Never point `--dest` at `~/.claude/agents` itself: adopt and apply wipe the destination before copying. The weekly job then tracks the fork per entry.

## Invocation

`poteto-mode`, `how`, `why`, `interrogate`, `figure-it-out`, `architect`, `swarm`, and `arena` are model-triggerable (no `disable-model-invocation` key). Every other skill, the `principle-*` leaves included, keeps `disable-model-invocation: true` and runs only when you type `/<name>`.
