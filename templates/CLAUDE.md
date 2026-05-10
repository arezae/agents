# [Project Name] — Project Context for Claude Code

> **Where to look first.** Project-specific facts live in `PROJECT_CONTEXT.md`. This
> file (`CLAUDE.md`) is the routing layer: which agents exist, which skills are
> available, and how Claude Code should behave at the conversation level. Agents read
> `PROJECT_CONTEXT.md` for everything domain-specific.

## What this project is

[One paragraph. What does the product do. Who is it for. What is the business shape.]

## Core docs

- `PROJECT_CONTEXT.md` — every project-specific fact agents read (stack, schema, scope, conventions, commands).
- [`ROADMAP.md`] — strategic direction, vision, history.
- [`PRODUCT_EXECUTION_SPEC.md`] — the frozen execution plan currently being delivered.
- [`TEAM_STRUCTURE.md`] — agent org chart and routing.

If `PROJECT_CONTEXT.md` and any other doc disagree on a current-delivery fact, follow `PROJECT_CONTEXT.md`.

## Stack

See `PROJECT_CONTEXT.md` → Stack.

## Running the project

See `PROJECT_CONTEXT.md` → Commands.

## Agents

The agents in `.claude/agents/core/` come from the shared agents library
(`github.com/<your-org>/agents`). Pinned via git submodule at `.claude/agents/core/`.

| Agent | Path |
|-------|------|
| Tech Lead | `.claude/agents/core/engineering/tech-lead.md` |
| Backend Architect | `.claude/agents/core/engineering/backend-architect.md` |
| Frontend Engineer | `.claude/agents/core/engineering/frontend-engineer.md` |
| Code Reviewer | `.claude/agents/core/engineering/code-reviewer.md` |
| Code Fixer | `.claude/agents/core/engineering/code-fixer.md` |
| UX Designer | `.claude/agents/core/design/ux-designer.md` |
| Product Manager | `.claude/agents/core/product/product-manager.md` |
| Product Analyst | `.claude/agents/core/product/product-analyst.md` |
| Market Researcher | `.claude/agents/core/growth/market-researcher.md` |
| Growth Strategist | `.claude/agents/core/growth/growth-strategist.md` |

Product-specific agents live alongside core under `.claude/agents/product-specific/`. Add them as you discover the project needs domain expertise the core library doesn't cover.

## Skills

Reusable skills from the library at `.claude/agents/skills/`:

- `run-migrations` — required before any schema migration
- `validate-api-schemas` — required before any API surface change PR
- `format-admin-report` — admin command / digest formatting

Product-specific skills live at `.claude/skills/`.

### Mandatory skill use ("1% rule")

If there is even a 1% chance a skill applies to your current task, invoke it. Each skill
file has a `**Use when:**` line — read it. Skipping a relevant skill is not permitted.

## Routing

When the user's request matches a built-in slash command (e.g., `/ship`, `/plan-eng-review`,
`/investigate`, `/qa`, `/review`), invoke it via the Skill tool. See the gstack catalog
in `~/.claude/CLAUDE.md` for the full list and routing rules.

For project-internal skills, agents invoke them by reference per the skill registration
above.

## Coding conventions

See `PROJECT_CONTEXT.md` → Coding Conventions.

## Testing

See `PROJECT_CONTEXT.md` → Testing Workflow. Never commit code that lowers the green-test count.

## Plan & Spec gates

See `PROJECT_CONTEXT.md` → Plan File Convention and Spec Gate Convention.
