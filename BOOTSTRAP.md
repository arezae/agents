# Bootstrap — Starting a new product with this agent library

This guide turns an empty repository into a product workspace with a full agent team
specialized to your project.

---

## Prerequisites

- Claude Code installed and authenticated.
- A new repo (or one you can add `.claude/` to).
- Decisions on:
  - Product name and one-line description
  - Target audiences
  - Initial stack (web framework, database, language)
  - Scope guardrails (what is in / out for this phase)

If you don't have these yet, run `/office-hours` (gstack) to pressure-test the idea
before bootstrapping. Saves rework later.

---

## Step 1 — Add the agents library as a submodule

```bash
git submodule add git@github.com:arezae/agents.git .claude/agents
git commit -m "Add agents library as submodule at .claude/agents"
```

This pins your project to a specific commit of the agents library. To pull updates
later: `git submodule update --remote .claude/agents`.

If you don't want the submodule overhead and prefer to fork-and-customize the entire
library, use GitHub's "Use this template" button instead and skip Step 1.

---

## Step 2 — Copy the templates

```bash
cp .claude/agents/templates/PROJECT_CONTEXT.md ./PROJECT_CONTEXT.md
cp .claude/agents/templates/CLAUDE.md ./CLAUDE.md
```

`PROJECT_CONTEXT.md` is the keystone. Every borderline agent reads it for
project-specific facts. Without it filled in, the agents will fall back to generic
defaults that may not match your project.

---

## Step 3 — Fill in PROJECT_CONTEXT.md

Open `PROJECT_CONTEXT.md` and replace every `[…]` placeholder. Sections to nail down:

1. **Product** — what it does, who pays, current stage.
2. **Audiences** — every persona the product serves, with their goals and pain points.
3. **Scope Guardrails** — what is explicitly out of scope for this phase.
4. **Stack** — the actual frameworks, services, and infrastructure choices.
5. **Architecture** — an ASCII diagram of how services fit together.
6. **Database** — critical tables, append-only tables, schema location.
7. **External Service Interfaces** — every paid or third-party service, with logging and cost guards.
8. **Testing Workflow** — the exact commands tests run with.
9. **Commands** — the cheat sheet of project commands.
10. **Coding Conventions** — strict mode, validation layer, server-authority rules, AI call logging.

If a section doesn't apply to your project, **delete it** rather than leaving it
empty. Empty sections invite agents to invent answers.

### Sections you can defer

- **Contract Protocol** — only fill in if the project has multiple consumers depending on a typed API surface (frontend + mobile + partners). Single-consumer projects can skip this.
- **Plan File Convention** and **Spec Gate Convention** — useful when work spans multiple agents. Solo work doesn't need them.
- **Active Execution Spec** — only relevant once the project has a frozen execution plan.

---

## Step 4 — Customize CLAUDE.md

Open `CLAUDE.md` and:
1. Replace the project name and "What this project is" paragraph.
2. Update the "Core docs" list to point to your project's actual roadmap and execution spec files (if any).
3. Confirm the agent table matches your roster — keep core agents you want, remove ones the project will never use.
4. List any product-specific agents you've added under `.claude/agents/product-specific/`.

---

## Step 5 — Set up agent journals (recommended)

Spawned agents have no shared memory between invocations. Plan files and specs
cover per-feature work, but per-agent learnings, recurring findings, and
in-progress state need somewhere to live across sessions.

```bash
cp -r .claude/agents/templates/agent-journals .claude/agent-journals
git add .claude/agent-journals
git commit -m "Add agent-journals directory for cross-session continuity"
```

Each library agent has a `## Working Memory` section that points to its journal
at `.claude/agent-journals/<agent-name>.md`. Agents read their journal on entry
and append a dated entry before returning. The directory is host-state — it
lives in your project, not in the submodule.

If you skip this step, agents check for the directory and proceed without a
journal if it's missing. You can adopt journals later by running the copy
command above.

---

## Step 6 — Add product-specific agents (when needed)

Don't pre-emptively create agents you might need. Wait until a real task surfaces a
gap, then write the agent.

When a product-specific agent emerges:

```bash
mkdir -p .claude/agents/product-specific/<category>
$EDITOR .claude/agents/product-specific/<category>/<agent-name>.md
```

Use any core agent as a template. Strip the `PROJECT_CONTEXT.md` indirection — your
product-specific agent can hardcode product details since it lives in this repo.

After you've used the same agent on 2+ unrelated projects, consider promoting it to
the agents library.

---

## Step 7 — Add product-specific skills (when needed)

Skills are deterministic, repeatable workflows. Add one when you find yourself doing
the same multi-step thing 3+ times.

```bash
mkdir -p .claude/skills/<skill-name>
$EDITOR .claude/skills/<skill-name>/skill.md
```

Use the format in `.claude/agents/skills/run-migrations/skill.md` as a starting point.

---

## Step 8 — Verify the setup

Open the project in Claude Code. Test:

```
"Show me the Tech Lead agent and confirm it's reading PROJECT_CONTEXT.md."
```

The Tech Lead should reference your project's stack, scope guardrails, and active
execution spec — not LogiKid's. If it's still talking about logic puzzles, the agent
file isn't being read or `PROJECT_CONTEXT.md` isn't filled in.

---

## Maintenance

### Pulling updates from the agents library

If your project uses a submodule:

```bash
git submodule update --remote .claude/agents
git diff .claude/agents       # review what changed in the agents
git add .claude/agents
git commit -m "Pull agents library updates"
```

### Diverging from upstream

If you've customized a core agent in your project, override it by copying it to
`.claude/agents/product-specific/` with the same path structure. The version in
`product-specific/` takes precedence (Claude Code scans both).

Avoid editing files inside the submodule directly — your changes will be overwritten
on the next update.

### Contributing improvements back

When a product-specific agent or skill turns out to be reusable across projects:
1. Fork the agents library.
2. Generalize the agent (replace product specifics with `PROJECT_CONTEXT.md` references).
3. Open a PR.

---

## Troubleshooting

**Agents reference "LogiKid" or "Phase 2".** The submodule is pulling old files, or
you copied a LogiKid-specific agent by mistake. Re-run `git submodule update --remote`
and confirm only generic agents are present.

**Agents ignore PROJECT_CONTEXT.md.** Confirm the file lives at the project root (not
inside `.claude/`). Confirm the relevant agent's "Project context" header still says
"see PROJECT_CONTEXT.md at the project root" — if it's been edited away, restore it.

**Submodule won't clone for collaborators.** Make sure the agents repo is public, or
the collaborator has SSH access to your fork. Add a note to your project README:
`git clone --recurse-submodules <your-repo>` for first clone, or
`git submodule update --init` after a regular clone.
