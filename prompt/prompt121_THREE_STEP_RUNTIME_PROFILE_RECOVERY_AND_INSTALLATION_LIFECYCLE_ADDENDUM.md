# Prompt121 Addendum — Three-step runtime-profile recovery, startup routing, and installation writer lifecycle

This addendum is binding and overrides any conflicting migration-first, five-readiness-table, parameter-table, or generic readiness-framework interpretation in `prompt/prompt121.md` and earlier prompt121 addenda.

## Authoritative operator decision

Use the two existing local WPF tables as the installation/runtime state contract:

```text
TblPosRuntimeProfile
= the single current local runtime/installation state
= the only installation-state source used for startup routing

TblPosRuntimeStateHistory
= append-only audit of state transitions
= never a MainWindow eligibility gate
```

The current canonical Development database contains both physical tables, but they are empty because the installation flow does not write the runtime-profile lifecycle. Correct that lifecycle directly.

Do exactly three things:

```text
1. Backfill the current canonical Development database through the application-owned runtime-state writer so the already-installed local environment is represented as completed.
2. Make normal WPF startup read TblPosRuntimeProfile first and route to InstallationV0 or MainWindow from that current state.
3. Wire the real installation flow so future installs write database-complete and application-complete transitions through TblPosRuntimeProfile and TblPosRuntimeStateHistory at the correct transaction boundaries.
```

Do not create a broader readiness framework.

## Important transaction interpretation

Do not keep one database transaction open across machine-side/application installation work.

Use two explicit, idempotent local state transitions:

```text
Transition A — Database ready
one local PostgreSQL transaction:
  required schema/migration state already current or applied by the existing Phase2 boundary
  + required baseline seed
  + upsert current TblPosRuntimeProfile state = existing source equivalent of DatabaseReady
  + append TblPosRuntimeStateHistory transition when the state actually changes
  + commit

Transition B — Application ready
only after all required local machine/application persistence succeeds:
one local PostgreSQL transaction:
  upsert current TblPosRuntimeProfile state = existing source equivalent of ApplicationReady/Installed/Ready
  + append TblPosRuntimeStateHistory transition when the state actually changes
  + commit
```

Use the exact existing enum/status/property names when they already exist. Do not invent duplicate status vocabularies. If the current source has no way to distinguish database-ready from application-ready, make the smallest explicit model change needed and prove why it is required.

Remote availability is not part of ApplicationReady after local installation completion:

```text
API offline
WpfJwt expired/rejected
protected hello 401
SignalR unavailable
sync unavailable
```

must not downgrade ApplicationReady and must not route an installed local POS back to InstallationV0.

## Step 0 — Prove the physical model and existing ownership

Before editing or writing data, inspect and report:

```text
exact schema/table names
all columns and types for TblPosRuntimeProfile
all columns and types for TblPosRuntimeStateHistory
primary keys and unique constraints
foreign keys
current EF entity mappings
current DbSet ownership
existing runtime-status enum/constants
existing startup reader call chain
existing Phase2 transaction owner/service/method
existing local activation/application-completion owner/service/method
all existing writers or attempted writers for both tables
```

Return one exact prior defect classification:

```text
W1_NO_RUNTIME_PROFILE_WRITER_EXISTS
W2_WRITER_EXISTS_BUT_NOT_INVOKED
W3_WRITER_OUTSIDE_TRANSACTION_OR_ROLLED_BACK
W4_WRITER_USES_WRONG_DB_SCHEMA_CONTEXT_OR_KEY
W5_WRITER_STATUS_DOES_NOT_MATCH_STARTUP_PREDICATE
W6_STARTUP_READS_WRONG_DB_SCHEMA_KEY_OR_STATUS
W7_MULTIPLE_PROVEN_DEFECTS_IN_SEQUENCE
W8_OTHER_EXACTLY_PROVEN_RUNTIME_PROFILE_LIFECYCLE_DEFECT
```

Do not return only `table empty`.

## Step 1 — Controlled backfill of the current Development installation

The operator authorizes a one-time recovery of:

```text
obm_pos_dev_v0_pg
```

because the local baseline/app were previously installed but the runtime-profile writer was missing.

Requirements:

```text
- Do not use ad-hoc psql/manual SQL as the final recovery mechanism.
- Reuse the existing runtime-state owner if present; otherwise add the smallest application-owned writer methods to the existing Phase2/local-activation boundary.
- Invoke the same production writer methods that future installation will use.
- Verify the existing baseline/local prerequisites before marking DatabaseReady or ApplicationReady.
- Do not fabricate tenant/station identity; use the already persisted canonical local identity only.
- Preserve retained protected credential/checkpoint records.
- Create/update exactly one current runtime-profile row for the canonical local installation identity.
- Append history only for actual state transitions.
- Do not create TblLocalOutbox rows.
- Do not call API.
```

For the already-installed Development environment, drive the writer through the valid transition sequence:

```text
missing/current incomplete
-> DatabaseReady
-> ApplicationReady
```

Record row counts and sanitized state names before and after. Do not expose raw tenant/device identities or secret values.

## Step 2 — Startup routing from TblPosRuntimeProfile

Normal WPF startup must resolve the canonical ProductRoot and canonical local database, then read exactly the one current runtime-profile row.

Required routing semantics:

```text
no current profile row
  -> InstallationV0 begins/resumes at the earliest incomplete local phase

current state before DatabaseReady
  -> InstallationV0 Phase2/database work

current state DatabaseReady but before ApplicationReady
  -> InstallationV0 resumes only the local application-activation/finalization step

current state ApplicationReady/Installed/Ready
  -> open the real production MainWindow directly

corrupt, duplicate, or identity-mismatched current state
  -> precise recoverable repair state; no crash and no blind MainWindow bypass
```

`TblPosRuntimeStateHistory` must have startup query/dependency count `0` except optional diagnostics after routing has already been decided.

The following must be orthogonal status only:

```text
remote API credential valid/invalid
API reachable/offline
SignalR connected/disconnected
sync available/unavailable
```

They may show Connected/Degraded/Offline after MainWindow opens, but may not influence installation routing once the current profile is ApplicationReady.

## Step 3 — Correct the real installation lifecycle

Wire the existing installation flow so every future install performs:

```text
Phase1 local protected pairing/identity persistence
-> Phase2 schema/baseline work
-> Transition A: DatabaseReady in the same local transaction as baseline completion
-> local application activation/finalization
-> Transition B: ApplicationReady in its own short local transaction
-> next normal startup reads ApplicationReady and opens MainWindow
```

Requirements:

```text
- Current profile upsert must be idempotent.
- Re-running the same transition must not create duplicate current rows.
- History is append-only but must not duplicate a history entry for an idempotent no-op retry.
- A failed baseline transaction must leave no DatabaseReady state.
- A failed application-finalization transaction must leave DatabaseReady, not ApplicationReady, so restart resumes correctly.
- No transaction may span network calls, API waits, UI waits, or machine setup that occurs outside PostgreSQL.
- No remote hello/API success is required to preserve ApplicationReady after initial local completion.
- No installation transition creates TblLocalOutbox.
```

Add focused integration tests proving rollback and idempotency. Use an isolated disposable Development test database or existing test harness; do not reset the canonical Development database and do not touch production/customer/reference databases.

## Schema and migration lock

Both target tables physically exist. Do not create a migration merely because they are empty.

A migration is allowed only if direct schema/model evidence proves a required missing or incompatible:

```text
column
primary/unique key
constraint
mapping
```

Do not create:

```text
TblSchemaVersion
TblSystemBaselineVersion
Phase2TrialCompletionMarker
another runtime-profile table
another history table
a second startup/readiness service
```

## Physical acceptance

Update the visible WPF label to:

```text
prompt121
```

Stop the full API and prove port `7161` has no listener.

PASS requires all of the following:

```text
Current Development recovery:
- writer methods, not ad-hoc SQL, populated the two existing tables
- exactly one current profile row exists
- current state is ApplicationReady/Installed/Ready using the exact accepted source value
- expected transition history exists
- TblLocalOutbox rows created = 0

Startup:
- normal WPF Development launch uses the canonical ProductRoot and obm_pos_dev_v0_pg
- startup reads TblPosRuntimeProfile as the current source of truth
- MainWindow opens directly
- InstallationV0 does not appear or flash
- API is offline
- MainWindow remains alive and responsive for at least 60 seconds
- close normally and launch a second time
- second launch again opens MainWindow directly with API offline

Lifecycle tests:
- baseline rollback cannot write DatabaseReady
- successful baseline + profile transition commit together
- application-finalization failure remains DatabaseReady
- successful finalization writes ApplicationReady
- retries are idempotent
- history never gates startup
```

If MainWindow does not open directly, verdict remains BLOCKED.

PASS verdict:

```text
OBM_WPF_RUNTIME_PROFILE_TWO_STAGE_LIFECYCLE_AND_MAINWINDOW_OFFLINE_PHYSICALLY_RESTORED_READY_FOR_OPERATOR_SCREENSHOT
```

Status after PASS:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=true
MANUAL_POS1_TEST_READY=false
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
```

## Frozen work

Do not modify:

```text
Category Weight
Booking Weight
Price Weight semantics
TblTenantPosDevice
API destination routing
canonical sync Provider
CompanionApp or payment-terminal modeling
API database/schema
.env cleanup or Firebase cleanup
refresh-token design
```

Stop immediately after the runtime-profile lifecycle and physical MainWindow acceptance are complete.

## Required report fields

Update `report/report121.md` with:

```text
Verdict
Prior defect classification W1-W8
Exact current-profile table schema/mapping
Exact history table schema/mapping
Existing status names reused or minimal status change made
Current-profile unique key/cardinality
Startup reader class/method/query
Phase2 transaction owner/service/method
Application-finalization owner/service/method
Development backfill performed through production writer yes/no
Ad-hoc manual SQL used count
Current profile rows before/after
History rows before/after
DatabaseReady transition proof
ApplicationReady transition proof
Baseline/profile atomic commit proof
Application-finalization/profile atomic commit proof
Rollback proof
Idempotent retry proof
History startup dependency count
TblLocalOutbox rows created count
API port 7161 offline proof
MainWindow opens directly yes/no
InstallationV0 shown/flashed yes/no
60-second MainWindow proof
Second-launch MainWindow proof
WPF build/test totals
Operator MainWindow screenshot ready true/false
Manual POS1 test ready false
Private artifact path/version and aggregate SHA-256
```
