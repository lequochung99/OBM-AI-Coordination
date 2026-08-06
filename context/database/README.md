# Reusable Database Context

This directory stores durable, reviewed knowledge about OBM database tables and seed/runtime contracts so future agents do not rediscover the same facts.

## Rules

- One logical table/domain gets its own directory.
- `CURRENT.md` points to the latest verified version.
- Versioned DB context must record exact table/entity spelling, source commit, schema/key facts, ownership, seed behavior, sync/outbox behavior, tests, and invalidation conditions.
- Distinguish `FACT`, `INFERENCE`, `UNKNOWN`, and `CONFLICT`.
- Never store passwords, connection strings, credentials, API keys, customer data, merchant credentials, private tokens, or production secrets.
- Values copied from a legacy/source database into canonical seed data must be explicitly classified as safe non-secret configuration before they are committed.
- Source code and physical DB evidence remain authoritative; this context is a reusable navigation and contract summary.

## Required final artifact pattern

```text
context/database/<LogicalTableName>/
  CURRENT.md
  V001/
    DB_CONTEXT.md
```

Future tasks must read the relevant `CURRENT.md` before reinvestigating the table.
