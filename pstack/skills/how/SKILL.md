---
name: how
description: "Use when Courtney asks 'how does X work', wants a code walkthrough before changing something, or asks where something should live, which package owns it, or whether this is the right layer. Motivation questions go to why."
---

# How

Explore the codebase to answer "how does X work?" questions.

If the scope is ambiguous, state your interpretation and explore. The user can redirect.

## 1. Explain

Spawn one Agent subagent with `subagent_type: Explore` (read-only agent) that explores and explains in one pass. Give it the user's question and this brief:

> You are writing an architectural explanation for a senior engineer. Explore the codebase and write an explanation that a senior engineer unfamiliar with this area could read and walk away with a solid mental model. They should understand the architecture well enough to start working in it confidently. You have read-only access to the codebase.

## 2. Present

Present the subagent's output. Light edits for clarity or context from the conversation are fine. Do not substantially rewrite it.
