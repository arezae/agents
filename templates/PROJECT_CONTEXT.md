# PROJECT_CONTEXT.md

> **What this file is.** The single source of truth for every project-specific fact your
> agents need: stack, schema, scope, conventions, commands, audiences. The agents in
> `.claude/agents/core/` (and any extensions you add) read this file to specialize.
>
> **Replace every "[…]" placeholder.** When a section doesn't apply, delete it rather
> than leaving it empty — empty sections invite agents to invent answers.

---

## Product

- **Name:** [Product name]
- **One-line description:** [What it is in plain language. Who pays. What value.]
- **Stage:** [Pre-launch / MVP / Growth / Mature]
- **Status:** [What is shipped, what is in progress, what is next]

## Audiences

List every distinct persona the product serves. Designers and product agents will read
this to tailor flows.

| Persona | Goal | Pain points | Tone / register |
|---------|------|-------------|-----------------|
| [Persona A] | [What they want from the product] | [What frustrates them] | [Formal / playful / etc.] |
| [Persona B] | […] | […] | […] |

## Scope Guardrails

- **In scope (current phase):** [Bulleted list — be concrete]
- **Explicitly out of scope:** [The tracks agents must not touch — leaderboards, B2B, teacher tools, social, etc.]
- **Pricing model:** [Free tier? Paid plans? Trial length?]

## Stack

- **Web:** [Framework + styling + state]
- **API:** [Language, framework, validation layer]
- **Database:** [Engine, hosting]
- **Cache / Queue:** [Redis / SQS / etc.]
- **Background work:** [Worker process, job queue]
- **External services:** [AI API, payment processor, email, search, analytics]
- **Infrastructure:** [Docker compose / K8s / Fly / Vercel / etc.]

## Architecture

```text
[Replace with an ASCII diagram of how the services fit together.
 Mark every external boundary clearly.]
```

## Repository Layout

```
[Top-level directory tree, only the parts agents need to navigate.]
```

## Branch & Workflow Conventions

- **Default branch:** [main / master / production]
- **Feature branches:** [Convention, e.g. `team/feature-name`]
- **Multi-agent work:** [How agents share or split branches]
- **PR rules:** [Who reviews, when to use Tech Lead, merge order]

## Database

- **Critical tables:** [List the tables agents should treat with care, with one line each]
- **Append-only tables:** [Tables that must never be `UPDATE`d or `DELETE`d. Used for audit, training, or telemetry.]
- **Schema files:** [Where the source-of-truth schema lives, where migrations go]

## External Service Interfaces

Document every external service, with the contract agents must respect.

### [Service A — e.g. AI API]
- **Purpose:** [What we use it for]
- **Logging:** [Where every call must be logged — table or log stream]
- **Cost guard:** [Daily spend limit env var, what to do when hit]
- **Caching:** [Cache key shape — keep this stable so producer and consumer agree]

### [Service B — e.g. Payments]
- […]

## Contract Protocol (optional)

If your project keeps a typed API contract that consumers depend on (web frontend,
mobile bot, partners), declare it here. Otherwise delete this section.

- **Snapshot file:** [`services/api/src/contracts/snapshot.json` — the committed baseline]
- **Detector commands:**
  - `[npm run contracts:check]` — flags breaking API surface changes
  - `[npm run contracts:check-db]` — flags schema drift for DB consumers
- **What counts as breaking:** [Removing a procedure, removing/renaming a field, adding a required input]
- **Consumers:**
  - `[services/web]` — frontend, owns `[services/web/consumer-manifest.json]`
  - `[services/telegram-bot]` — bot, owns `[services/telegram-bot/db-contract.json]`
- **Merge sequence on a breaking change:**
  1. Backend reports the impact to Tech Lead.
  2. Tech Lead notifies each consumer.
  3. Consumers adapt and confirm clean type-checks.
  4. Tech Lead merges in order: backend → web → bot.
  5. Tech Lead promotes `snapshot-new.json` to `snapshot.json` and commits.

## Plan File Convention

Use this if a task spans multiple files AND involves more than one agent:

- **Path:** `docs/plans/YYYY-MM-DD-feature.md`
- **Format:**
  ```markdown
  # Plan: [Feature name]
  **Date:** YYYY-MM-DD
  **Branch:** [branch]
  **Spec:** [link to spec if any]

  ## Goal
  One sentence.

  ## Tasks
  - [ ] Task 1 (owner: agent-name) — ~N min
  - [ ] Task 2 (owner: agent-name) — ~N min

  ## Verification gate
  What command proves this is done?
  ```
- **Rules:** Tasks must be small enough to verify (typically 2–5 min). Single-agent / single-file fixes are exempt.

## Spec Gate Convention (optional)

If a task spans two or more agents or two or more service boundaries, the responsible
agent writes a design spec:

- **Path:** `docs/specs/YYYY-MM-DD-topic.md`
- **Approver:** Tech Lead.
- **Self-approval window:** 24 hours.
- **Frozen at approval** — the spec becomes the Stage 1 source of truth for PR review.

## Testing Workflow

- **Test runner:** [`npm test` / `pytest` / etc.]
- **Required suites:**
  - `[npm run test:run]` — full unit + integration
  - `[npm run test:schema]` — schema validation
  - `[npm run test:e2e]` — end-to-end (when relevant)
- **Coverage rules:**
  - Schema change → schema test required.
  - New input validation → validation test required.
  - New API procedure → integration test required.

## Commands (cheat sheet)

```bash
[Replace with the actual commands agents run most often:]
make dev                 # Start local stack
make db-migrate          # Apply migrations
make logs-api            # Tail API logs
npm run test:run         # Run tests
npm run contracts:check  # Breaking-change detection
```

## Environment Variables

```
[List the env vars agents need to know exist. Mark which are required and which have safe defaults.]
[VAR_NAME]              # purpose, required / default
```

## Generated Files (do not edit by hand)

| File | Generator |
|------|-----------|
| `[services/web/src/types/api-router.ts]` | `[npm run contracts:export-types]` |
| `[…]` | `[…]` |

## Coding Conventions

- **Language strictness:** [TypeScript strict mode / Python type-checked, etc.]
- **Validation:** All external inputs validated with [Zod / Pydantic / etc.]
- **Server authority:** [What the client must never decide — e.g., "answer correctness, pricing, permissions"]
- **AI calls:** Logged via [`logAiCall()`] before every call. Daily spend gate via [`AI_DAILY_SPEND_LIMIT_EUR`].

## Active Execution Spec

The current frozen execution plan agents follow:

- **Path:** [`PRODUCT_EXECUTION_SPEC.md` or similar]
- **Phase:** [Phase 2 / MVP / etc.]
- **Owner:** [Tech Lead / Product Manager]

## Agent Roster

The agents this project uses, by category. Mark each as core (from the agents library)
or product-specific (defined in this repo).

| Agent | Source | Notes |
|-------|--------|-------|
| Tech Lead | core | — |
| Backend Architect | core | — |
| Frontend Engineer | core | — |
| Code Reviewer | core | — |
| Code Fixer | core | — |
| UX Designer | core | — |
| Product Manager | core | — |
| Product Analyst | core | — |
| Market Researcher | core | — |
| Growth Strategist | core | — |
| [Domain agent A] | product-specific | [Why this product needs it] |
| [Domain agent B] | product-specific | […] |

## Skill Roster

Skills used by agents in this project.

| Skill | Source | Use when |
|-------|--------|----------|
| run-migrations | library | Any schema change |
| validate-api-schemas | library | Any API surface change |
| format-admin-report | library | Admin command / digest |
| [domain-skill] | product-specific | […] |
