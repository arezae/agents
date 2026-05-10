---
title: Validate API Schemas (Pre-PR Check)
version: 1.0
audience: Code Reviewer, Backend Architect
frequency: Every API change (1+ per feature)
complexity: Low
---

**Use when:** you are adding or modifying a typed API procedure (tRPC, FastAPI, NestJS,
GraphQL, etc.), changing any input/output schema, or reviewing an API PR. This skill is
a required gate before any API change is approved.

---

## Purpose

Validate API input/output schemas **before code review** to catch contract violations
early. This skill:
- Extracts schemas from the project's validation layer (Zod, Pydantic, class-validator, GraphQL SDL)
- Confirms input schemas have the right checks: ID validation, enums, numeric ranges, optional defaults
- Confirms output schemas match declared return types
- Generates a checklist of validation tests that must exist
- Detects breaking changes against a committed contract snapshot when the project keeps one

---

## When to Use

- ✅ Adding or modifying any typed API procedure
- ✅ Changing the input or output schema of an existing endpoint
- ✅ Approving an API PR (required gate)
- ✅ Validating third-party webhook payloads (Stripe, GitHub, etc.)

Do **not** use when:
- ❌ The schema lives only in a comment, not in the validation layer
- ❌ The function is internal and not exposed across a trust boundary
- ❌ You are generating tests rather than validating (use the `generate-tests` skill if the project has one)

---

## Validation Checks

| Check | What to look for | Example (Zod) |
|-------|------------------|---------------|
| **Required fields** | No `.optional()` on fields the handler reads unconditionally | `userId: z.string().uuid()` |
| **ID validation** | UUIDs use `.uuid()`, opaque IDs match a regex | `z.string().uuid()` / `z.string().regex(/^[A-Z0-9]{10}$/)` |
| **Enum validation** | All enum-shaped fields use `.enum([...])`, not free strings | `status: z.enum(['pending', 'paid', 'cancelled'])` |
| **Numeric ranges** | All numbers have `.min()` / `.max()` where bounds exist | `limit: z.number().int().min(1).max(100)` |
| **String bounds** | All strings have `.min()` / `.max()` where bounds exist | `name: z.string().min(1).max(120)` |
| **Optional defaults** | `.optional()` paired with `.default()` so handlers can rely on the value | `limit: z.number().optional().default(10)` |
| **Output schema** | Declared return type matches the actual response | `output(z.object({ id: z.string(), … }))` |

The same patterns apply in Pydantic (`Field(min_length=1, max_length=120)`),
class-validator (`@IsUUID()`, `@IsIn(['…'])`), and GraphQL (enum types vs `String`).

---

## Workflow

1. List the API surface files changed in the diff.
2. For each procedure:
   - Extract the input schema and the output schema (if declared).
   - Walk every input field and check it against the table above.
   - Confirm there is a unit test exercising at least one valid case and one invalid case per field that has a constraint.
3. Produce a violation report.
4. Block the PR if any **critical** violation is present (no ID validation, no enum validation, missing required field). Warn for **non-critical** violations (missing output schema, undocumented default).

### Example output

```
🔍 Validating API schemas…

✅ services/api/src/routers/sessions.ts
   ✓ sessions.list: Input schema valid
     - childProfileId: UUID validation ✓
     - limit: optional number with default ✓

⚠️  services/api/src/routers/orders.ts
   ⚠️  orders.create: Missing enum validation for `status`
       Suggestion: z.enum(['pending', 'paid', 'cancelled'])

Summary:
  Procedures: 12
  Valid: 11 ✓
  Violations: 1 ⚠️
```

---

## Test Checklist Generator

For every validated procedure, the skill emits a list of tests that should exist:

```markdown
### orders.create

- [ ] Valid input → 200 OK with declared output shape
- [ ] Invalid `status` enum → 400 error
- [ ] Missing required `customerId` → 400 error
- [ ] `quantity` below min → 400 error
- [ ] `quantity` above max → 400 error
- [ ] Authenticated as wrong user → 403 forbidden
```

If a corresponding test is missing in the validation test file, treat it as a violation
(severity: warn → block depending on the project's policy).

---

## Breaking Change Detection (mandatory pre-PR gate)

If `PROJECT_CONTEXT.md` declares a contract snapshot mechanism, run the project's
breaking-change check before opening any API PR. Common shape:

```bash
# Replace with the command from PROJECT_CONTEXT.md → Commands
npm run contracts:check
```

| Change | Classification | Action |
|--------|---------------|--------|
| Procedure removed or renamed | 🔴 BREAKING | Block PR |
| Input field removed or renamed | 🔴 BREAKING | Block PR |
| Output field removed or renamed | 🔴 BREAKING | Block PR |
| New required input field added | 🔴 BREAKING | Block PR |
| New procedure added | ✅ Non-breaking | Proceed |
| New optional input field with default | ✅ Non-breaking | Proceed |
| New output field added | ✅ Non-breaking | Proceed |

### On a breaking change

1. Do **not** merge.
2. Notify the Tech Lead with the printed impact report.
3. Tech Lead notifies all affected consumers (frontend, mobile, partner integrations) listed in `PROJECT_CONTEXT.md`.
4. Each consumer adapts and confirms readiness (typically a clean type-check or build).
5. Tech Lead merges in the order specified in `PROJECT_CONTEXT.md` and updates the committed snapshot afterward.

If the project does not keep a snapshot, skip this gate but call out the change in the
PR description so consumers can react.

---

## Authority & Usage

- **Required for**: every API surface PR (Code Reviewer enforces).
- **Blocking**: missing ID validation, missing enum validation, missing required-field guard, breaking changes without sign-off.
- **Warnings**: missing output schema, undocumented default, no validation test for a field with constraints.
- **Advisory**: response shape doesn't precisely match output schema (non-blocking when the actual shape is a strict subset).

---

## Known Limitations

1. Static check — does not test runtime behavior. Pair with integration tests.
2. Cannot catch data-shape mismatches across two endpoints unless both are inspected.
3. Enum values aren't auto-synced with database enums; project-specific tooling is needed for that.
4. Requires the validation layer to be consistent — if some endpoints use Zod and others use ad-hoc checks, this skill only catches the Zod ones.

---

## Governance

- **Version**: 1.0
- **Owner**: Code Reviewer agent
- **Frequency**: every API change
- **Testing**: unit tests over the validation rules + integration runs over each router file in the project
