---
name: arena
description: "Spawn N parallel candidates at the same task, pick a base, graft the strongest parts of the losers into it. Use for /arena, 'arena this', 'throw it in the arena', or when one attempt at a non-trivial artifact would lock in the wrong shape."
---

# Arena

Fan out N parallel attempts at the same task. Read every candidate end to end. Pick the strongest as the base. Graft the best ideas from the others into it. Verify the synthesized result.

## Start

Open a todolist with one entry per phase before launching anything.

1. Frame
2. Fan out
3. Cross-judge
4. Pick
5. Graft
6. Verify

## Phase A: Frame

The N candidates will receive the same prompt, so the prompt is the contract.

1. State the artifact each candidate is producing.
2. Derive the rubric. State what success looks like for *this* task, then turn it into 3-6 concrete gradeable criteria. The rubric is the picker's tool in Phase D. Candidates only see the task.
3. Pick the runners. Runner model and effort per `~/.claude/skills/poteto-mode/references/models.md`, role `arena runners` (two by default, one Agent call per entry). An `inherit` entry in this role or the cross-judge pool means the parent model, so omit `model` for it. If the Agent tool rejects a configured entry, run that seat on `opus`, `max` and say so. If it rejects that too, use the closest valid model and effort from its error message. Spawn more when the arena covers multiple design directions, up to five; past that, reading every candidate end to end in Phase D costs more than the extra candidate adds. Same model N times when the work is generation-bound rather than judgment-sensitive. Don't run an arena when the task has one obvious shape or the rubric has fewer than three criteria; one `Agent` call, or doing it yourself, is cheaper than reading N candidates.
4. Assign output paths. Each candidate writes to its own location (a git worktree where possible, via `isolation: worktree` on the Agent call, otherwise `/tmp/arena-<slug>/candidate-<n>/`), per the **separate-before-serializing-shared-state** principle skill.

## Phase B: Fan out

Spawn all N candidates in one message, one `Agent` call each with `subagent_type: general-purpose`, `model` and `effort` from the role line, and `run_in_background: true`, each with the task, the path to the shared grounding, its own output path, and instructions to produce both the artifact and a short rationale. While they run, keep working: re-read the rubric and the grounding so Phase D starts the moment the last candidate lands.

Each rationale names the alternatives the candidate considered and what it rejected.

If a candidate fails to produce output, proceed with N-1 and note the dropout in the synthesis record.

## Phase C: Cross-judge

After all Phase B candidates complete, choose one entry from `~/.claude/skills/poteto-mode/references/models.md`, role `arena cross-judge pool`. Prefer a model different from the parent's; a `counselors` entry gives a read from another vendor (Codex or Gemini) and runs through the counselors skill instead of an Agent call. Otherwise spawn one judge with `subagent_type: Explore` (read-only), that model and effort, and `run_in_background: true`. It sees the rubric and the candidates by path label, scores each criterion, and recommends a base with rationale. It runs in parallel with the parent's reading in Phase D, not with the candidates themselves. Don't spawn the judge while candidates are still writing.

## Phase D: Pick a base

Read every candidate end to end before picking.

Score each candidate against the rubric criterion by criterion, not on holistic feel. Compare against the cross-judge. Agreement on the base confirms the pick. Disagreement means one of you is biased or the rubric was ambiguous. Read both rationales before deciding.

Pick the base on which candidate a future maintainer can extend most easily without breaking invariants. Prefer the cleaner boundary or smaller API when two feel tied, per the Laziness Protocol.

Record the pick and the reason in a short synthesis note alongside the base artifact, including the cross-judge's verdict.

## Phase E: Graft

Walk each losing candidate once more and identify what is worth porting into the base. The signal is usually one or two things per candidate, not most of it.

Fold each graft in by hand, per the **redesign-from-first-principles** principle skill. Don't paste mechanically. The result has to remain coherent under one mental model.

Record what was grafted, from which candidate, and what was rejected and why.

When N candidates converge on the same shape, that is a strong agreement signal. Note the convergence in the record and ship the consensus shape. No graft is needed. When N candidates wildly diverge, Phase A was under-specified. Reframe and re-run rather than averaging the divergence.

## Phase F: Verify

The synthesized artifact has to hold up under the same scrutiny as any other output, per the **prove-it-works** principle skill.

If verification surfaces a problem the arena did not catch, either Phase A was wrong (re-frame and re-run) or one candidate caught it and you missed the graft (go back to Phase E). Don't paper over.

## Outputs

One synthesized artifact. One short synthesis note alongside, naming the base, the grafts (with source candidate), the rejections, the dropouts if any, and the verification result.
