# prompt140 — Repair premature ApplicationReady/Activated state and resume missing local bootstrap without DB reset

## Context

The latest operator physical retest after prompt139 now routes directly into MainWindow successfully, but the local OBM-POS database has no expected baseline/bootstrap data visible in the application. The existing database was previously observed with:

- EF migration history present
- exactly one `TblPosRuntimeProfile` row
- current semantic state `Activated` (treated as `ApplicationReady`)
- runtime state history present
- Tenant/POS structural rows present

The operator now confirms the app opens directly but appears empty because the baseline/bootstrap data was not seeded/applied.

Current architecture authority is `INSTALLATION_RUNTIME_CANONICAL_V005.md`.

Canonical rule:

```text
ApplicationReady/Activated is valid only after:
- schema is current
- canonical Tenant/POS identity is valid
- required POS1 baseline or POS2/replacement bootstrap snapshot is committed
- required local runtime/setup rows are committed
```

The local DB must not be dropped, recreated, copied over, truncated, or manually forced into readiness.

## Objective

Determine exactly why `Activated/ApplicationReady` was committed before the required baseline/bootstrap data, correct the existing owner/transaction boundary, safely repair the current incomplete DB idempotently, and physically prove that the application opens with the expected baseline data.

## Mandatory Phase A — Read-only forensic classification before mutation

Inspect the active WPF source and the current target DB read-only.

### A1. Trace the exact write path to `Activated/ApplicationReady`

Identify:

- the method/service/repository that writes `DatabaseReady`
- the method/service/repository that writes `Activated/ApplicationReady`
- the baseline/bootstrap owner invoked before that write
- the transaction boundary around baseline/bootstrap + final state
- every branch that can write `Activated/ApplicationReady` without applying the required baseline/bootstrap

Produce a concise call-chain table:

| Concern | Active owner | Preconditions | Transaction boundary | Observed defect |
|---|---|---|---|---|

### A2. Inventory current DB state by counts only

Do not print business payloads, identities, credentials, or raw GUIDs.

Inspect at minimum:

- pending migration count
- `TblPosRuntimeProfile` row count and semantic state
- `TblPosRuntimeStateHistory` row count and transition sequence
- required Tenant/POS identity/setup row counts
- required settings count
- required parameters count
- required printer default count
- required default role count
- any existing bootstrap/version/watermark marker already owned by current source
- `TblLocalOutbox` count attributable to installation/bootstrap (must remain zero)
- employee/service/customer/invoice/output/booking counts only to confirm they were not seeded

Use the existing approved baseline manifest/owner in source. Do not invent a new manifest or new baseline table.

### A3. Classify the current DB

Choose exactly one:

- `B1_SCHEMA_READY_BOOTSTRAP_NOT_STARTED`
- `B2_BOOTSTRAP_PARTIALLY_APPLIED_STATE_PREMATURELY_ACTIVATED`
- `B3_BOOTSTRAP_COMPLETE_UI_DATA_SOURCE_MISMATCH`
- `B4_EXISTING_BUSINESS_DATA_REQUIRES_OPERATOR_REVIEW`

If `B4`, stop without mutation and report:

`BLOCKED_PROMPT140_EXISTING_BUSINESS_DATA_REPAIR_REQUIRES_OPERATOR_REVIEW`

## Mandatory Phase B — Minimal source correction

Reuse existing owners only. Do not create a second installer, second bootstrap service, second startup coordinator, second readiness framework, second sync path, or second state table.

Correct the logic so that:

```text
migrations current
→ write/retain DatabaseReady
→ execute approved POS1 baseline or POS2/replacement bootstrap
→ verify required baseline/bootstrap result through the existing owner
→ atomically commit required identity/setup/baseline + Activated/ApplicationReady + history
```

Requirements:

1. `Activated/ApplicationReady` must never be written before the required baseline/bootstrap work succeeds.
2. Baseline/bootstrap failure must leave the DB resumable at `DatabaseReady` or equivalent incomplete state.
3. Restart must resume only missing/idempotent work.
4. Existing complete baseline rows must not be duplicated.
5. Installation/bootstrap creates zero `TblLocalOutbox` rows.
6. No employee, service, customer, invoice, output, booking, queue, event, or delivery data is seeded unless already explicitly required by the approved baseline authority.
7. Do not use manual SQL inserts to force readiness.
8. Do not add a new bootstrap completion table merely for this prompt.
9. If the current `Activated` row is proven premature, repair it only through the existing runtime-profile owner and canonical state transition logic. Do not directly patch the table with ad hoc SQL.

## Mandatory Phase C — Safe repair of the current DB

For `B1` or `B2` only:

- preserve the current DB identity
- do not drop/recreate/reset
- do not copy from another DB
- resume the existing approved bootstrap idempotently
- ensure exactly one current runtime-profile row
- ensure final semantic state is `Activated/ApplicationReady` only after bootstrap commit
- append only the required valid state history transition; do not fabricate history
- prove installation/bootstrap outbox rows remain zero

If the source proves POS1/new-tenant baseline is the correct branch, use that branch. If the installation plan proves POS2/replacement, use the existing canonical snapshot/bootstrap branch instead. Do not guess installation mode.

## Mandatory tests

Add focused tests covering at least:

1. Schema current + baseline missing + state `DatabaseReady` → resume baseline then `ApplicationReady`.
2. Baseline failure → no `ApplicationReady/Activated` write.
3. Premature `Activated` + missing baseline → startup does not silently treat installation as complete; canonical repair path is selected.
4. Partial baseline → idempotent completion without duplicates.
5. Complete baseline + `Activated` → direct MainWindow route.
6. Installation bootstrap creates zero `TblLocalOutbox` rows.
7. Repeated resume does not duplicate settings/parameters/printer defaults/roles.
8. Existing business data + inconsistent state → recovery/blocker, no automatic reseed/reset.

## Physical proof required

Using the existing target DB only:

1. Run latest WPF build with visible label `prompt140`.
2. Confirm no DB drop/recreate/reset occurred.
3. Resume bootstrap through the corrected existing owner.
4. Prove pending migrations = 0.
5. Prove required baseline counts match the approved source manifest/owner.
6. Prove exactly one current runtime-profile row.
7. Prove current semantic state = `Activated/ApplicationReady` only after bootstrap completion.
8. Prove installation/bootstrap `TblLocalOutbox` rows = 0.
9. Open MainWindow and verify the expected baseline-backed screens are no longer empty where baseline data is required.
10. Keep MainWindow responsive for at least 60 seconds with API offline.
11. Restart twice; both launches must open MainWindow directly without InstallationV0 flashing.

Do not claim PASS based only on build/tests.

## Frozen scope

Do not modify:

- Category Weight
- Booking Weight
- Price Weight
- sync/outbox architecture
- SignalR
- Companion/payment terminal
- Firebase/.env
- refresh-token architecture
- PlatformApp authorization
- Pairing Code contract

## Required report

Create and push `report/report140.md` with:

- verdict
- root-cause classification
- exact premature-state call chain
- DB classification B1/B2/B3/B4
- sanitized pre/post row counts
- source files changed
- transaction boundary before/after
- tests/build totals
- destructive-operation counts
- physical MainWindow/restart/offline evidence
- private artifact version and SHA-256
- coordination commit SHA

PASS verdict only when all physical proof succeeds:

`OBM_WPF_V005_BOOTSTRAP_COMPLETED_BEFORE_APPLICATIONREADY_MAINWINDOW_DATA_PRESENT_OFFLINE_RESTART_PROVEN`
