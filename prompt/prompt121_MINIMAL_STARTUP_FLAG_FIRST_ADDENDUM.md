# Prompt121 Addendum — Prove the exact local installation-complete startup flag before creating any migration

This addendum is binding and overrides any broader interpretation in `prompt/prompt121.md`.

## Authoritative operator hypothesis

The operator believes the normal startup decision should be simple:

```text
open WPF
-> read local DB installation-complete/runtime-ready state
-> if complete, skip InstallationV0 and open MainWindow
-> if incomplete, stay in InstallationV0
```

The current behavior may therefore be caused by one wrong/missing local row or marker rather than by a need to add every readiness object listed in report120.

Do not create a migration first. Prove the exact startup predicate and exact local DB facts first.

## Exact first question

Answer with direct source and database evidence:

```text
Which exact table, row/key, column/value, and class/method decide whether normal startup opens MainWindow or InstallationV0?
```

Trace the complete direct call chain only:

```text
App startup
-> startup coordinator
-> LocalPosStartupService.AssessAsync
-> exact repository/query
-> exact predicate/result
-> MainWindow or InstallationV0 branch
```

Record exact class, method, file, and line for each boundary.

## Read-only DB inspection first

Before any source or schema mutation, inspect `obm_pos_dev_v0_pg` read-only and report for every object actually queried by the startup decision:

```text
object/table name
exists yes/no
expected row/key
row exists yes/no
expected column/value
actual sanitized value
last-write timestamp when safe
owning service that should create/update it
whether it is a schema requirement, baseline marker, activation marker, or station identity
```

The five objects from report120 are candidates, not an automatic mandate:

```text
dbo."TblSchemaVersion"
dbo."TblSystemBaselineVersion"
dbo."TblPosRuntimeProfile"
dbo."TblPosRuntimeStateHistory"
dbo."Phase2TrialCompletionMarker"
```

For each candidate, prove one of:

```text
A. directly queried and required for MainWindow eligibility
B. indirectly required by the existing Phase2 service
C. historical/test-only/not used by the normal startup gate
D. duplicate or stale contract
```

Do not add a table merely because a source type exists.

## Required classification

Choose exactly one primary result:

```text
F1_SINGLE_EXISTING_TABLE_ROW_OR_VALUE_WRONG
F2_SINGLE_REQUIRED_ROW_MISSING_IN_EXISTING_TABLE
F3_EXISTING_PHASE2_SERVICE_FAILED_TO_WRITE_COMPLETION_STATE
F4_STARTUP_READS_WRONG_TABLE_OR_STALE_STATE_SOURCE
F5_REQUIRED_TABLE_TRULY_MISSING_FROM_ATTACHED_MIGRATION
F6_MULTIPLE_READINESS_OBJECTS_TRULY_REQUIRED
F7_OTHER_EXACTLY_PROVEN_LOCAL_STARTUP_FLAG_DEFECT
```

## Minimal repair order

### If F1 or F2

Do not create a migration.

Use the existing canonical Phase2/completion/activation service that owns that row. Repair the service only if it fails to write its owned state. Do not execute manual SQL `UPDATE`/`INSERT` merely to force PASS.

### If F3

Fix only the exact existing Phase2 service transaction/write defect, rerun that service once, and prove the expected row/value was created through the owning code path.

### If F4

Fix only the startup query/state-source selection so it reads the canonical existing local state. Do not duplicate markers.

### If F5

Create one attached Npgsql migration containing only the exact table/object proven necessary for the startup predicate or its canonical owning Phase2 service.

Do not automatically create all five candidate objects.

### If F6

Only after direct proof that multiple objects are all required may one attached migration include those proven objects. Explain the dependency order and why a single completion flag is insufficient under the existing production contract.

## Hard prohibitions

```text
no WPF DB reset/drop/recreate
no EnsureCreated
no manual psql table creation
no manual completion-marker write
no fabricated activation/station identity
no new startup coordinator
no new readiness framework
no API change
no token/refresh/redeem change
no TblTenantPosDevice work
no Category Weight or Booking Weight work
```

## Physical proof remains mandatory

After the smallest correction:

```text
visible label = prompt121
API port 7161 offline
canonical ProductRoot selected
canonical DB = obm_pos_dev_v0_pg
exact installation-complete predicate = true through canonical local state
MainWindow opens directly
InstallationV0 does not appear or flash
MainWindow remains responsive for at least 60 seconds
close normally
launch again
second launch opens MainWindow directly with API still offline
```

If InstallationV0 remains visible, the task is BLOCKED. Build success, migration success, or a corrected DB row without MainWindow physical proof is not PASS.

## Required report additions

`report/report121.md` must state:

```text
Exact startup decision class/method/line
Exact deciding table/row/key/column/value
Primary classification F1-F7
Which of the five report120 candidates are actually required
Read-only DB facts before correction
Owning writer/service for the deciding state
Migration created yes/no
If yes, exact objects included and why each is necessary
Direct manual SQL used = no
MainWindow opens directly yes/no
InstallationV0 shown/flashed yes/no
API-offline 60-second proof yes/no
Second-launch proof yes/no
```

PASS verdict remains:

```text
OBM_WPF_PHASE2_READINESS_MIGRATION_COMPLETED_MAINWINDOW_OFFLINE_PHYSICALLY_RESTORED_READY_FOR_OPERATOR_SCREENSHOT
```

Even when no migration is needed, use the same PASS verdict only when MainWindow physical proof passes. In the report, explicitly state `MIGRATION_NOT_REQUIRED_MINIMAL_FLAG_REPAIR` when applicable.
