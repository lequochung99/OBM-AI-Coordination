# Prompt121 Addendum — TblPosRuntimeProfile is the single local startup source of truth

This addendum is binding and overrides any broader or conflicting migration-first, TblParameter, or five-readiness-object interpretation in prompt121 and its earlier addenda.

## Authoritative operator decision

The canonical local startup contract is intentionally simple:

```text
TblPosRuntimeProfile contains the current local installation/runtime status.
If the current local profile says installation is complete/ready, normal startup opens MainWindow.
If no current local profile exists, or the current profile explicitly says installation is incomplete, startup enters InstallationV0.
```

`TblPosRuntimeStateHistory` is history/audit only. It must never be required to decide whether MainWindow may open.

Do not use `TblParameter` as the installation-complete source. Do not create a new readiness framework. Do not require all five report120 candidate objects for normal startup.

The operator has provided physical DB evidence that these tables already exist:

```text
TblPosRuntimeProfile
TblPosRuntimeStateHistory
```

Therefore, before any migration, prove the exact schema, mapping, rows, columns, and current values of those existing tables in `obm_pos_dev_v0_pg`.

## Exact task order

### 1. Prove the existing physical tables

Using read-only inspection first, record:

```text
exact PostgreSQL schema for TblPosRuntimeProfile
exact PostgreSQL schema for TblPosRuntimeStateHistory
exact quoted physical names
column names and types
primary key / unique key
foreign keys
row count
current local profile row identity
current installation/runtime status value
last-updated timestamp
whether current startup queries the same schema/table/columns
```

Explicitly reconcile report120's expectation of `dbo."TblPosRuntimeProfile"` and `dbo."TblPosRuntimeStateHistory"` with the physical tables shown by the operator. Determine whether the defect is:

```text
wrong schema mapping
wrong quoted table name
wrong DbContext/model exclusion
wrong status column/value
missing current profile row
stale startup query
another exact mapping/state defect
```

Do not infer that a migration is required merely because the attached migration chain does not show these objects. Physical reality and the active startup query must be reconciled first.

### 2. Establish the single current-state contract

Audit the existing entity and service writers for `TblPosRuntimeProfile`. Reuse existing columns/status values when possible; do not invent a second status model.

Return the exact startup truth table using the current source names:

```text
current profile row exists + status means installed/ready
=> MainWindow

current profile row missing
=> InstallationV0

current profile row exists + status means incomplete/installing/failed-repair-required
=> InstallationV0 with precise recoverable status
```

Remote state must not participate in this local decision:

```text
WpfJwt 401
API offline
SignalR offline
sync unavailable
no refresh-token contract
```

These produce cloud/sync degraded status only after MainWindow opens when local profile is installed/ready.

### 3. Use the existing Phase2 transaction boundary

When Phase2 baseline installation completes successfully, the same canonical transaction/checkpoint boundary must set the current `TblPosRuntimeProfile` status to the existing installed/ready value.

If the existing design appends `TblPosRuntimeStateHistory`, that history row may be written in the same transaction, but startup must not depend on the history table.

Do not manually `INSERT` or `UPDATE` a status row solely to force PASS. Use the existing Phase2 completion service/writer. Correct that existing writer if it fails to persist the current profile state.

### 4. Smallest correction only

Preferred correction order:

```text
A. Fix wrong schema/table mapping or query.
B. Fix wrong current status predicate/value mapping.
C. Fix existing Phase2 writer so it creates/updates the current profile row.
D. Create one attached migration only if direct proof shows the required physical table or required status column truly does not exist.
```

Do not create or require these merely for startup unless separately proven necessary for another feature:

```text
TblSchemaVersion
TblSystemBaselineVersion
Phase2TrialCompletionMarker
TblPosRuntimeStateHistory as a readiness gate
```

Do not create five readiness tables to satisfy an assessor.

## Frozen work

Do not modify:

```text
API DB/schema
TblTenantPosDevice
sync routing
CompanionApp or terminal model
Price Weight
Category Weight
Booking Weight
refresh-token architecture
Pairing Code behavior
Firebase cleanup
.env cleanup
```

Do not reset/drop/recreate the WPF DB. Do not use EnsureCreated. Do not use manual SQL to fabricate installed state.

## Physical PASS gate

PASS requires the operator-equivalent normal WPF Development launch with visible label `prompt121` to prove:

```text
canonical ProductRoot selected
canonical DB = obm_pos_dev_v0_pg
TblPosRuntimeProfile physical table/schema proven
current local profile row and installed/ready value proven
startup reads exactly that current profile state
API port 7161 has no listener
MainWindow opens directly
InstallationV0 does not show or flash
WpfJwt/API failure is shown only as cloud/sync degraded
MainWindow remains alive and responsive for at least 60 seconds
close normally
launch a second time
MainWindow again opens directly while API remains offline
```

If InstallationV0 still appears, verdict must remain BLOCKED.

PASS verdict:

```text
OBM_WPF_RUNTIME_PROFILE_SINGLE_SOURCE_MAINWINDOW_OFFLINE_PHYSICALLY_RESTORED_READY_FOR_OPERATOR_SCREENSHOT
```

## Required report121 fields

```text
Exact TblPosRuntimeProfile schema/table
Exact current-state columns used
Current profile row exists before/after
Current installation/runtime status before/after
Startup reader class/method/line
Phase2 profile writer class/method/line
TblPosRuntimeStateHistory startup dependency count
Wrong schema/mapping/state defect classification
Migration created yes/no and why
Manual status write count = 0
API offline proof
MainWindow direct-open proof
InstallationV0 shown/flashed yes/no
60-second proof
Second-launch proof
Operator screenshot ready true/false
Manual POS1 test ready false
```
