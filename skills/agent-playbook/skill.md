---
title: Agent Playbook (Token-Efficient Agent Design)
version: 1.0
audience: Tech Lead, Backend Architect, Product Manager, any agent designing agents, prompts or workflows
frequency: Per new agent, prompt, skill, or orchestration change
complexity: Low
---

**Use when:** you are designing or changing an agent, prompt, skill, workflow or orchestration, or when token usage or cost is a concern.

---

## Purpose

Apply the distilled rules in `playbook/PLAYBOOK.md` (in a consuming repo this is
`.claude/agents/playbook/PLAYBOOK.md`) so agent systems spend fewer tokens without losing quality.
The playbook is refreshed weekly from public sources and reviewed by a human before rules change.

---

## Workflow

1. Read `playbook/PLAYBOOK.md`. It is short on purpose; do not load the digests.
2. Before adding an agent, check the "Single agent or many" rules. Default to one agent.
3. Before writing a prompt, `CLAUDE.md` section or skill, check the "Claude Code token levers" and "Skills" rules.
4. For long-running work, apply the "Long-running work" rules (progress file, feature list, evaluator only where needed).
5. Record any deliberate deviation from a rule in the pull request or plan, with a one-line reason.

---

## Quick checklist

- One agent unless context pollution, true parallelism, or specialization justifies more
- Context loaded just in time; durable notes in files, not in the window
- Cheapest model that does the job; sub-agents on smaller models for noisy work
- CLAUDE.md lean; workflow-specific instructions in skills
- Verification target defined before work starts
- Token spend measured (/usage, /context) before and after a change

---

## Proposing changes

- Playbook and source changes go through a pull request; a human merges.
- Changes to agents or skills in this library get their own pull request with before/after, expected token effect and evidence link.
- Consumers receive changes only when they bump the submodule pointer.

---

## Governance

- **Version**: 1.0
- **Owner**: Tech Lead
- **Refresh**: weekly digest, monthly source review
