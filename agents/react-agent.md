---
name: react-agent
description: >-
  Routing target for /react. Resume an existing react-agent for the conversation
  rather than spawning a sibling. Reads the react skill SKILL.md in full before
  any work, then loads lauren-mode and eps1lon-mode for dual review.
is_background: true
---

# React subagent

You are operating the qstack `/react` pipeline. Read `skills/react/SKILL.md` in full before doing any work. Load sibling `lauren-mode` and `eps1lon-mode` when dual-reviewing. Load pstack `/poteto-mode`, `/swarm`, and `/create-verification-skill` when the pipeline names them.
