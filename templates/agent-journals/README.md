# Agent Journals

This directory holds per-agent continuity logs for the consuming project. Each
file is the persistent working memory for one agent role. Without it, every
agent invocation starts cold — agents have no other way to know what their past
selves decided.

This directory lives in the **host project**, not in the agents library
submodule. Each agent's `## Working Memory` section in the library points here.

## How agents use these files

Every agent with a `## Working Memory` section in the library is required to:

1. **Read its journal** at the start of every invocation (skim the last ~5 entries).
2. **Append a dated entry** before returning, using the format below.

If an agent's journal file doesn't exist yet, the agent creates it on first
append. If this whole directory doesn't exist, the agent proceeds without one
(journals are optional — the convention only applies once the project adopts it).

## Entry format

```markdown
## YYYY-MM-DD — <one-line summary of the task>

**Did:** What you actually changed/decided (files, procedures, schemas).
**Learned:** Non-obvious findings, surprises, gotchas. Skip if none.
**Open:** Anything left undone or blocked, with enough context for next time.
```

## Rules

- Append-only. Never delete past entries; supersede with a new entry instead.
- Keep entries terse — one screen max per entry.
- Do not duplicate plan files or specs; link to them instead.
- If a journal grows past ~500 lines, archive the older half to
  `<agent-name>.archive.md` and start the active file fresh.
- The journal is for the agent role, not a status report for humans.

## Naming

One file per agent, matching the agent's filename in the library:

- `tech-lead.md`, `backend-architect.md`, `frontend-engineer.md`,
  `code-reviewer.md`, `code-fixer.md`, `ux-designer.md`,
  `product-manager.md`, `product-analyst.md`, `market-researcher.md`,
  `growth-strategist.md`
- Extension agents: same convention (e.g. `multilingual-adapter.md`)
- Product-specific agents: same convention (use the agent's filename)

## Adoption

To adopt journals in a new project, copy this template directory into your
project root:

```bash
cp -r .claude/agents/templates/agent-journals .claude/agent-journals
```

That's it. The library agents will start using their journals automatically on
their next invocation.
