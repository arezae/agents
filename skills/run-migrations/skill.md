---
title: Run Migrations (Sequenced PostgreSQL)
version: 1.0
audience: Backend Architect, Tech Lead
frequency: Per schema change (1-2 per feature)
complexity: Medium
---

**Use when:** you are about to apply any PostgreSQL schema change — new table, column,
index, constraint, trigger, or data type modification — to any environment.

---

## Purpose

Apply PostgreSQL schema migrations safely and idempotently. This skill:
- Executes migrations in order (prevents schema conflicts)
- Validates `IF NOT EXISTS` checks (prevents overwriting existing data)
- Provides dry-run capability (preview changes before applying)
- Logs all changes for audit trail
- Prevents data loss through explicit confirmation gates for destructive ops

---

## When to Use

- ✅ Adding a new table or column
- ✅ Modifying indexes or constraints
- ✅ Changing data types or defaults
- ✅ Creating triggers or functions

Do **not** use when:
- ❌ The schema change is already applied (verify with the project's DB shell)
- ❌ Making data-only changes (use direct SQL or a one-off script)
- ❌ Testing exploratory queries (use interactive psql)

---

## Inputs

### Environment

| Variable | Type | Purpose |
|----------|------|---------|
| `DATABASE_URL` | string | Connection string |
| `ENVIRONMENT` | enum | `development` / `staging` / `production` — gates confirmation checks |

### Migration File Layout

The exact paths live in `PROJECT_CONTEXT.md` → Database. A common shape:

```
postgres/init/
├── 01-schema.sql          # Initial schema (immutable baseline)
├── 02-schema.sql          # Working snapshot (kept in sync with 01 + applied migrations)
└── 04-migrations.sql      # Sequential migration steps
```

If the project uses a real migration tool (Flyway, sqitch, knex migrations, etc.), defer
to that tool's invocation — the safety patterns below still apply inside each migration.

---

## Idempotent Migration Pattern

Every migration must be safe to re-run. Use `IF NOT EXISTS` (or `DO $$ ... END $$` blocks
that check `information_schema`) so accidental re-runs don't fail.

### Adding a column

```sql
DO $$ BEGIN
  IF NOT EXISTS (
    SELECT 1 FROM information_schema.columns
    WHERE table_name = 'orders' AND column_name = 'status'
  ) THEN
    ALTER TABLE orders ADD COLUMN status TEXT DEFAULT 'pending' NOT NULL;
    RAISE NOTICE 'Added column: orders.status';
  END IF;
END $$;
```

### Adding an index

```sql
CREATE INDEX IF NOT EXISTS idx_orders_status ON orders(status);
```

### Adding a check constraint

```sql
DO $$ BEGIN
  IF NOT EXISTS (
    SELECT 1 FROM information_schema.constraint_column_usage
    WHERE table_name = 'orders' AND constraint_name = 'orders_status_check'
  ) THEN
    ALTER TABLE orders ADD CONSTRAINT orders_status_check
      CHECK (status IN ('pending', 'paid', 'cancelled'));
  END IF;
END $$;
```

### Renaming a column (breaking — coordinate first)

```sql
DO $$ BEGIN
  IF EXISTS (
    SELECT 1 FROM information_schema.columns
    WHERE table_name = 'users' AND column_name = 'name'
  ) AND NOT EXISTS (
    SELECT 1 FROM information_schema.columns
    WHERE table_name = 'users' AND column_name = 'full_name'
  ) THEN
    ALTER TABLE users RENAME COLUMN name TO full_name;
  END IF;
END $$;
```

A rename is a breaking change. Coordinate with all consumers before applying.

---

## Workflow

```
1. Parse DATABASE_URL and authenticate
2. List pending migrations
3. For each migration:
   a. Run inside a transaction
   b. The IF NOT EXISTS gate skips already-applied work
   c. Validate the affected tables/indexes/constraints exist after
   d. Log the result
4. Summarize: X applied, Y skipped, Z errors
5. On failure: transaction rolls back automatically
```

### Dry-run before applying

If your tooling supports it, always preview first. Read the migration file, parse the
intended changes, and produce output like:

```
🔍 Migration Dry-Run

1️⃣  Add `status` column to `orders`
    Status: ✓ Can apply (table exists, column doesn't)

2️⃣  Add index on `orders.status`
    Status: ✓ Can apply (index doesn't exist)

3️⃣  Rename `users.name` → `users.full_name`
    Status: ⚠️  Breaking — verify all consumers updated

Summary:
  Can apply: 3
  Risk level: Medium (1 breaking change)
```

### Apply

Run the migration command declared in `PROJECT_CONTEXT.md` → Commands. Common shapes:

```bash
make db-migrate                                  # Makefile target
npm run migrate                                  # Node toolchain
docker compose exec api npm run migrate          # Containerized
psql "$DATABASE_URL" -f postgres/init/04-migrations.sql   # Direct
```

### Verify

```bash
# In psql:
\d orders             # column list, types, indexes for one table
\dt                   # all tables
```

```sql
-- Confirm the migration landed:
SELECT column_name, data_type, is_nullable
FROM information_schema.columns
WHERE table_name = 'orders';
```

---

## Error Handling

| Failure | Cause | Action |
|---------|-------|--------|
| `IF NOT EXISTS` skip | Migration already applied | Safe — continue |
| Syntax error | Bad SQL | Transaction rolls back; fix and re-run |
| Permission denied | Wrong DB role | Check `DATABASE_URL`; do not retry blindly |
| Lock timeout | Long-running query holding the table | Investigate before retrying |

---

## Best Practices

1. **Always test locally first** — run on dev/local Docker before staging or production.
2. **Backup before production** — snapshot the database before any schema change in prod.
3. **Use `IF NOT EXISTS`** — every migration must have an idempotency guard.
4. **Document breaking changes** — add a comment in the migration explaining code impact.
5. **Deploy schema first, code second** — additive schema before code that needs it; destructive schema after the code that depended on it has been removed.
6. **Coordinate destructive changes** — drops, renames, and tightened constraints can break running consumers. Notify them before merge, not after.

---

## Known Limitations

- This pattern uses `IF NOT EXISTS` rather than a tracked `applied_migrations` table. If your project has many migrations or multiple environments out of sync, switch to a real migration tool with version tracking.
- Down migrations are manual — write them explicitly when the work is reversible.
- Migrations run sequentially. Parallelism is not safe for schema work.

---

## Governance

- **Version**: 1.0
- **Owner**: Backend Architect
- **Testing**: Manual verification + integration tests after each migration
