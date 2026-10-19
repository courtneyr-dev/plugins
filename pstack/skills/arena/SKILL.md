---
name: arena
description: "Spawn N parallel candidates at the same task, pick a base, graft the strongest parts of the losers into it. Use for /arena, 'arena this', 'throw it in the arena', or when one attempt at a non-trivial artifact would lock in the wrong shape."
---

# Arena

Open a todolist with one entry per phase (Frame, Fan out, Cross-judge, Pick, Graft, Verify) before launching anything.

## Phase A: Frame

Every candidate gets the same prompt, so the prompt is the contract.

1. State the artifact each candidate is producing.
2. Derive the rubric, 3-6 concrete gradeable criteria for what success looks like on *this* task. It is the picker's tool in Phase D. Candidates only see the task.
3. Pick the runners. Runner model and effort per `~/.claude/skills/poteto-mode/references/models.md`, role `arena runners` (two by default, one Agent call per entry). An `inherit` entry in this role or the cross-judge pool means the parent model, so omit `model` for it. If the Agent tool rejects a configured entry, run that seat on `opus`, `xhigh` and say so. If it rejects that too, use the closest valid model and effort from its error message. Spawn more runners when the arena covers multiple design directions, up to five; past that, reading every candidate end to end in Phase D costs more than the extra candidate adds. Use the same model N times when the work is generation-bound rather than judgment-sensitive. Don't run an arena when the task has one obvious shape or the rubric has fewer than three criteria; one `Agent` call, or doing it yourself, is cheaper than reading N candidates. Pick each subagent's `effort` for the job, per the Subagents section of the **poteto-mode** skill.
4. Give each candidate its own output location (a git worktree where possible, via `isolation: worktree` on the Agent call, otherwise `/tmp/arena-<slug>/candidate-<n>/`), per the **separate-before-serializing-shared-state** principle skill.

## Phase B: Fan out

Spawn all N candidates in one message, one `Agent` call each with `subagent_type: general-purpose`, `model` and `effort` from the role line, and `run_in_background: true`, each with the task, the path to the shared grounding, its own output path, and instructions to produce the artifact and a short rationale naming the alternatives it considered and what it rejected. While they run, keep working: re-read the rubric and the grounding so Phase D starts the moment the last candidate lands.

If a candidate produces no output, proceed with N-1 and note the dropout.

## Phase C: Cross-judge

After all Phase B candidates complete, choose one entry from `~/.claude/skills/poteto-mode/references/models.md`, role `arena cross-judge pool`. Prefer a model different from the parent's; a `counselors` entry gives a read from another vendor (Codex or Gemini) and runs through the counselors skill instead of an Agent call. Otherwise spawn one judge with `subagent_type: Explore` (read-only), that model and effort, and `run_in_background: true`. It sees the rubric and the candidates by path label, scores each criterion, and recommends a base with rationale. It runs in parallel with your reading in Phase D. Don't spawn it while candidates are still writing.

## Phase D: Pick a base

Read every candidate end to end before picking. Score each against the rubric criterion by criterion, not on holistic feel, and compare with the cross-judge. Agreement on the base confirms the pick. Disagreement means one of you is biased or the rubric was ambiguous. Read both rationales before deciding.

Pick the base a future maintainer can extend most easily without breaking invariants. When two feel tied, prefer the cleaner boundary or smaller API, per the Laziness Protocol.

## Phase E: Graft

Walk each losing candidate once more for what is worth porting into the base, usually one or two things per candidate, not most of it. Fold each graft in by hand, per the **redesign-from-first-principles** principle skill, so the result stays coherent under one mental model. Don't paste mechanically.

If the candidates converge on one shape, that is strong agreement. Note it and ship the consensus shape. No graft is needed. If they wildly diverge, Phase A was under-specified. Reframe and re-run rather than averaging the divergence.

## Phase F: Verify

Verify the synthesized artifact like any other output, per the **prove-it-works** principle skill. If verification finds a problem the arena missed, either Phase A was wrong (re-frame and re-run) or a candidate caught it and you missed the graft (go back to Phase E). Don't paper over it.

## Outputs

One synthesized artifact, with a short synthesis note alongside naming the base and why, the cross-judge's verdict, each graft with its source candidate, what was rejected and why, any convergence or dropouts, and the verification result.
