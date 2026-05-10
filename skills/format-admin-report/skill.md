---
title: Format Admin Report (Parallel Queries + Messaging)
version: 1.0
audience: Tech Lead, Backend Architect, anything wiring up admin commands
frequency: 5+ distinct reports per project
complexity: Low
---

**Use when:** you need to fetch multiple admin metrics and format them into a
human-readable report, digest, or admin command response.

---

## Purpose

Fetch admin metrics via parallel database queries, then format into human-readable
reports with fallback handling. This skill:
- Runs independent queries in parallel (faster than sequential)
- Aggregates results into a clean object the formatter consumes
- Formats data into a multi-section message with consistent structure
- Handles missing data gracefully (fallback messages instead of empty sections)
- Eliminates copy-pasted message-building code across admin commands

---

## When to Use

- ✅ Admin commands that fetch and format data
- ✅ Weekly digests, daily progress reports
- ✅ Monitoring dashboards or status pages
- ✅ Cost / budget summaries

Do **not** use when:
- ❌ A one-off text response (no formatting needed)
- ❌ User-facing transactional messages (those need user-tailored copy, not admin telemetry)
- ❌ Real-time notifications where DB query latency would block the path

---

## Pattern

The skill is three small pieces stitched together:

### 1. Parallel queries

Independent queries run concurrently. Sequential is the default for new code, and the
default is wrong — make this idiomatic.

```typescript
const [
  { rows: overviewRows },
  { rows: featureRows },
  { rows: costRows },
  { rows: alertsRows },
] = await Promise.all([
  db.query(OVERVIEW_QUERY),
  db.query(FEATURE_QUERY),
  db.query(COST_QUERY, [range]),
  db.query(ALERTS_QUERY),
]);

return {
  overview: overviewRows[0],
  features: featureRows,
  costs: costRows[0],
  alerts: alertsRows,
};
```

In Python (asyncpg + asyncio):

```python
overview, features, costs, alerts = await asyncio.gather(
    pool.fetchrow(OVERVIEW_QUERY),
    pool.fetch(FEATURE_QUERY),
    pool.fetchrow(COST_QUERY, range),
    pool.fetch(ALERTS_QUERY),
)
```

### 2. Aggregation

Don't pass raw query rows into the formatter. Build a typed summary object so the
formatter is purely about layout.

```typescript
type AdminStatsSummary = {
  totalUsers: number;
  activeToday: number;
  features: { key: string; planned: number; done: number }[];
  costsToday: { calls: number; estimatedEur: number };
  alerts: { kind: string; detail: string }[];
};
```

### 3. Formatting with fallbacks

```typescript
function buildAdminStatsMessage(s: AdminStatsSummary): string {
  const featureLines = s.features.length
    ? s.features.map(f => `${f.key}: planned ${f.planned} | done ${f.done}`)
    : ['No feature activity recorded yet.'];

  const alertLines = s.alerts.length
    ? s.alerts.map(a => `⚠️  ${a.kind}: ${a.detail}`)
    : ['No alerts.'];

  return [
    '📊 Admin Stats Summary',
    '',
    '👥 Members',
    `   Total users: ${s.totalUsers}`,
    `   Active today: ${s.activeToday}`,
    '',
    '🚧 Features',
    ...featureLines.map(l => `   ${l}`),
    '',
    '💰 Costs Today',
    `   Calls: ${s.costsToday.calls}`,
    `   Estimated cost: €${s.costsToday.estimatedEur.toFixed(2)}`,
    '',
    '🚨 Alerts',
    ...alertLines.map(l => `   ${l}`),
  ].join('\n');
}
```

---

## Workflow

```
1. Parse input (range, limit, filters)
2. Run independent queries in parallel
3. Aggregate raw rows into a typed summary object
4. Format into a sectioned message:
   a. Header line
   b. Each section with a fallback if its data is empty
   c. Join with \n
5. Return the formatted string
```

---

## Example Output

```
📊 Admin Stats Summary

👥 Members
   Total users: 412
   Active today: 38

🚧 Features
   AUTH-1: planned 1 | done 1
   ONBOARD-2: planned 2 | done 0

💰 Costs Today
   Calls: 142
   Estimated cost: €3.48

🚨 Alerts
   ⚠️  Low inventory: easy/junior bucket has 3 items
```

---

## Range Parsing Helper

When commands accept a range (`today`, `week`, `month`):

```typescript
function parseRange(text: string | undefined): 'today' | 'week' | 'month' {
  const normalized = (text ?? '').trim().toLowerCase();
  if (normalized.includes('week')) return 'week';
  if (normalized.includes('month')) return 'month';
  return 'today';
}
```

---

## Performance Targets

- **Query execution**: ~1 second for 4–8 parallel queries on a healthy database.
- **Formatting**: <100ms for typical message sizes.
- **End-to-end**: 1–2 seconds typical, well under chat-platform timeouts (Telegram: 60s, Slack: 3s for synchronous response).
- **Message size**: keep under platform limits (Telegram: 4096 chars; Slack: 40 KB; Discord: 2000 chars per message).

---

## Testing

### Unit tests

Test the formatter with empty inputs:

```typescript
it('builds message with fallback for empty data', () => {
  const msg = buildAdminStatsMessage({
    totalUsers: 0,
    activeToday: 0,
    features: [],
    costsToday: { calls: 0, estimatedEur: 0 },
    alerts: [],
  });
  expect(msg).toContain('No feature activity recorded yet.');
  expect(msg).toContain('No alerts.');
});
```

### Integration test

Run end-to-end against a real database (or test fixture) and assert:
- Total wall-clock time < 2s
- Output contains every section header
- Output handles a populated case correctly

---

## Known Limitations

1. **No caching.** Queries run on every invocation. If the report is expensive and called frequently, add a TTL cache layer in the project.
2. **No pagination.** Large result sets are truncated to the per-section limit. For long lists, link out to a dashboard.
3. **Platform-specific formatting.** Markdown vs HTML vs Slack mrkdwn vs Discord differ — keep the formatter parametric, or fork it per platform.

---

## Governance

- **Version**: 1.0
- **Owner**: Backend Architect (or whoever owns admin tooling)
- **Frequency**: per admin command (on demand)
