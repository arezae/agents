# agents — A software-company-in-a-box

A library of reusable Claude Code agents and skills extracted from real product
delivery. Drop them into a new repo, fill in `PROJECT_CONTEXT.md`, and you have a
multi-role engineering team specialized to your project.

## What's in here

```
core/                       Universal roles every software product needs
├── engineering/            tech-lead, backend-architect, frontend-engineer, code-reviewer, code-fixer
├── product/                product-manager, product-analyst
├── design/                 ux-designer
└── growth/                 market-researcher, growth-strategist

extensions/                 Optional add-ons by domain
└── content/                multilingual-adapter

skills/                     Reusable workflows agents invoke
├── run-migrations/         Idempotent PostgreSQL schema migrations
├── validate-api-schemas/   Pre-PR validation gate for typed APIs
└── format-admin-report/    Parallel queries + sectioned message formatting

templates/                  Files to copy into a new product repo
├── PROJECT_CONTEXT.md      The keystone — every project-specific fact agents read
├── CLAUDE.md               Routing layer: which agents and skills exist for the project
└── agent-journals/         Per-agent continuity logs (host-state, not in this submodule)
```

Each library agent has a `## Working Memory` section pointing to its journal at
`.claude/agent-journals/<agent-name>.md`. Agents read their journal on entry and
append a dated entry before returning — that is how spawned agents carry context
across sessions in a consuming project.

## Two-layer design

Core agents are **generic role frames**. They contain none of your product's facts —
no schema, no domain model, no scope. They read `PROJECT_CONTEXT.md` in the consuming
repo for everything specific to your product.

This means:
- A new product gets the same Tech Lead and Backend Architect that LogiKid uses.
- The same agents specialize differently for an e-commerce platform, a fintech app, or a chat product.
- When the agents library improves, every project can pull updates.

## Bootstrapping a new product

See `BOOTSTRAP.md` for the full setup. Short version:

```bash
# In a new product repo:
git submodule add git@github.com:arezae/agents.git .claude/agents
cp .claude/agents/templates/PROJECT_CONTEXT.md ./PROJECT_CONTEXT.md
cp .claude/agents/templates/CLAUDE.md ./CLAUDE.md
# Fill in the placeholders. Add product-specific agents under .claude/agents/product-specific/.
```

## Origin

These agents were authored and refined while building LogiKid (an AI logic-puzzle
product for kids and teens). They are not synthetic templates — each role was shaped by
real delivery work across content, engineering, design, product, research, and SaaS
tracks. LogiKid-specific agents (question generators, explanation writers, the
adaptive engine, the SaaS spinoff plan) were intentionally excluded so this library
stays portable.

## Status

- 11 core agents, 1 extension agent, 3 skills.
- Used in production at LogiKid.
- Suitable for: greenfield product repos, internal tools, B2B SaaS, prototype-stage products.

## Contributing

When you discover an agent or skill is missing for a product:
1. First add it to your product repo as a product-specific agent.
2. After 2+ uses, evaluate whether it's reusable.
3. If yes, generalize and contribute it back here.

Three similar lines is better than a premature abstraction. Don't promote to core
until the same need has appeared in at least two unrelated projects.
