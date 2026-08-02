# Prompt121 Addendum — One authoritative local installation-completion source; do not build a five-table readiness framework

This addendum is binding and overrides any broader migration-first interpretation in `prompt/prompt121.md` and prior prompt121 addenda.

## Authoritative operator architecture correction

The operator's established POS model is intentionally simple:

```text
baseline seed is committed in one local database transaction
-> if the required baseline data exists after commit, local installation is complete
-> if the baseline data is absent/empty, local installation is incomplete
```

In the legacy/current-market application, startup uses an existing local parameter/settings table (operator refers to `TblParameter`; confirm the exact active table/class name in the current source) as the durable baseline-completion evidence. Because the baseline seed commits atomically, the app does not require a graph of independent readiness tables to decide whether MainWindow may open.

The current WPF implementation appears to have lost this whole-system view and introduced separate readiness entities/markers that are not attached to the migration chain. Do not automatically create all five objects merely because source classes reference them.

## Canonical design intent

Local startup needs one authoritative installation-completion source of truth.

The canonical startup decision must be equivalent to:

```text
canonical local PostgreSQL database reachable
+ supported/current schema
+ required minimal baseline transaction is durably complete
=> open MainWindow
```

Otherwise:

```text
required minimal baseline absent/incomplete
=> show InstallationV0 / Phase2 completion flow
```

Remote state is independent:

```text
WpfJwt missing/expired/rejected
API offline
SignalR unavailable
sync unavailable
station cloud registration unavailable
```

These may set cloud/sync to degraded, but after local baseline completion they must not route the app back to InstallationV0 and must not block MainWindow.

## Required first investigation — exact current startup predicate

Before any schema/model/migration edit, trace the complete normal startup call chain and return the exact predicate that selects MainWindow versus InstallationV0.

At minimum record:

```text
STARTUP_ENTRY_CLASS_METHOD
STARTUP_ASSESSOR_CLASS_METHOD
EXACT_DB_QUERY_OR_EF_QUERY
EXACT_TABLE_NAME
EXACT_ROW_KEY_OR_FILTER
EXACT_COLUMN_OR_VALUE_CHECK
CURRENT_VALUE_OR_ROW_COUNT
MAINWINDOW_TRUE_CONDITION
INSTALLATION_TRUE_CONDITION
WRITER_SERVICE_FOR_THIS_STATE
TRANSACTION_BOUNDARY_FOR_THIS_STATE
```

Do not return only `SchemaMigrationRequired`, `Phase2 incomplete`, or `readiness objects missing`.

Inspect the current canonical WPF DB read-only and establish:

```text
which existing parameter/settings/baseline tables contain rows
which exact rows were inserted by the accepted baseline seed
whether the expected baseline set is complete
whether those rows were committed atomically
whether an existing completion/version row already exists
whether the current startup ignores valid baseline evidence because it expects unattached readiness entities
```

Confirm the exact current table name rather than assuming the legacy name. Likely candidates include existing parameter/settings tables already used by the product, but source and DB evidence decide.

## Five readiness objects are candidates, not mandatory schema

The following objects identified by report120 must be classified individually:

```text
dbo."TblSchemaVersion"
dbo."TblSystemBaselineVersion"
dbo."TblPosRuntimeProfile"
dbo."TblPosRuntimeStateHistory"
dbo."Phase2TrialCompletionMarker"
```

For each return exactly one classification:

```text
A_REQUIRED_FOR_LOCAL_STARTUP_SOURCE_OF_TRUTH
B_REQUIRED_ONLY_FOR_PHASE2_EXECUTION_OR_AUDIT
C_REQUIRED_ONLY_FOR_CLOUD_SYNC_OR_REMOTE_IDENTITY
D_HISTORY_OR_DIAGNOSTIC_ONLY
E_TEST_TRIAL_OR_STALE_CONTRACT
F_DUPLICATE_OF_EXISTING_PARAMETER_BASELINE_STATE
G_OTHER_EXACTLY_PROVEN_ROLE
```

Do not create a table for classifications B-G merely to make startup green.

`TblPosRuntimeStateHistory` must not gate startup if it is history/audit only.

Cloud station/runtime profile must not gate local MainWindow unless direct business-data safety evidence proves it is essential to read the single-tenant local DB. Missing cloud identity should normally degrade API/sync only.

A table named `Phase2TrialCompletionMarker` must not become a permanent production startup dependency merely because a trial implementation referenced it. Prove whether it is test/trial residue.

## Preferred correction order

### Outcome 1 — Existing baseline rows already prove installation completion

If the canonical DB already contains the complete required baseline set from one committed transaction:

```text
repair LocalPosStartupService/startup predicate to use the existing authoritative baseline evidence
remove/de-scope unattached readiness objects from MainWindow eligibility
create no migration
write no manual marker
perform no new seed
```

This is the preferred minimal correction.

### Outcome 2 — Existing table is present but one canonical baseline/completion row is missing

Use only the existing Phase2/baseline service and its existing transaction to complete the required baseline. The completion evidence must be written by that service in the same transaction as the baseline seed, not manually.

```text
BEGIN
  required minimal baseline writes
  one authoritative baseline version/completion evidence
COMMIT
```

If anything fails, rollback leaves installation incomplete.

Do not create `TblLocalOutbox` rows for installation seed.

### Outcome 3 — A single durable completion/version row is genuinely required but no existing table can own it

Only after proving Outcomes 1 and 2 impossible may source add one narrowly scoped completion/version object. Prefer one existing product-owned parameter/system table over a new framework. If a new table is unavoidable, create only the single object required for the authoritative completion source, through one attached Npgsql migration.

Do not create all five report120 objects by default.

### Outcome 4 — Current readiness framework is stale/overengineered

Retire or de-scope it from startup. Preserve audit/history code only where useful, but it must not be a prerequisite for local MainWindow startup.

## Minimal baseline definition

Use the existing accepted product baseline policy. Installation completion may rely only on the required minimal baseline, such as the exact current product-owned set of:

```text
settings/parameters
printer defaults
required default roles
other directly proven mandatory setup rows
```

Do not seed:

```text
employees
services
customers
Invoice
TblOutputInfo
Booking
queue/turn history
event/delivery history
```

Required marker:

```text
LOCAL_SEED_NO_OUTBOX=true
```

## Local-first startup truth table

The final startup behavior must be:

```text
baseline complete + API online + token valid
=> MainWindow, cloud connected

baseline complete + API offline
=> MainWindow, cloud/sync offline

baseline complete + WpfJwt 401/expired/missing
=> MainWindow, cloud authentication degraded

baseline incomplete
=> InstallationV0 Phase2 flow
```

Do not erase local installation completion when remote authentication fails.

## Physical acceptance

Use visible label:

```text
prompt121
```

Stop full API and prove port `7161` has no listener.

PASS requires:

```text
canonical ProductRoot selected
canonical DB = obm_pos_dev_v0_pg
provider = Npgsql/PostgreSQL
exact authoritative baseline-completion source documented
required baseline evidence valid
API offline
MainWindow opens directly
InstallationV0 does not appear or flash
local DB operation succeeds
MainWindow remains alive/responsive for at least 60 seconds
close normally
launch a second time
second launch again opens MainWindow directly while API remains offline
```

The following are not PASS:

```text
InstallationV0 stays alive
build succeeds
five tables exist but MainWindow does not open
manual marker/row edit forces MainWindow
API must be online to open MainWindow
```

PASS verdict:

```text
OBM_WPF_SINGLE_BASELINE_COMPLETION_GATE_MAINWINDOW_OFFLINE_PHYSICALLY_RESTORED_READY_FOR_OPERATOR_SCREENSHOT
```

## Frozen work

Do not modify:

```text
Category Weight
Booking Weight
Price Weight semantics
TblTenantPosDevice
API destination routing
CompanionApp/BookingConsole handoff
API schema/config cleanup
refresh-token design
sync E2E
```

Do not reset/drop/recreate the WPF DB.

## Required report121 fields

```text
Verdict
Exact startup entry and assessor method
Exact authoritative table/row/key/value or baseline signature
Current baseline row counts/signature
Baseline transaction writer service/method
Startup gate before
Startup gate after
Each of the five readiness object classifications A-G
Readiness objects created count
Attached migration created yes/no and exact reason
Manual SQL/data edit count = 0
Phase2 service invoked yes/no
LOCAL_SEED_NO_OUTBOX proof
API port 7161 offline proof
MainWindow opens directly yes/no
InstallationV0 shown/flashed yes/no
60-second MainWindow proof yes/no
Second-launch MainWindow proof yes/no
Operator MainWindow screenshot ready true/false
Manual POS1 test ready false
Private artifact and aggregate SHA-256
```
