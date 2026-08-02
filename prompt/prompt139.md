# prompt139 — Diagnose and fix `POS_RUNTIME_ROUTE_UNKNOWNFAILURE` after local DB verification

## Context

The operator physically reached the following state in the latest WPF build after prompt138:

```text
Verify Local Database Baseline: enabled
Open OBM-POS: enabled
Open OBM-POS state: Failed
StageId=InstallationV0OpenObmPos
ResultCode=POS_RUNTIME_ROUTE_UNKNOWNFAILURE
```

This proves prompt138 command enablement is active. The new blocker is no longer button state. It is the runtime/startup route returned when `Open OBM-POS` invokes the existing startup owner.

Current canonical authority remains:

```text
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL_V005.md
```

Do not change V005.

## Goal

Find the exact cause of `POS_RUNTIME_ROUTE_UNKNOWNFAILURE`, prove the physical local DB/runtime-profile state, and make the smallest safe correction so that:

```text
ApplicationReady or proven existing equivalent Activated
-> existing LocalPosStartupService selects MainWindow
-> MainWindow opens
```

Do not bypass runtime readiness and do not delete/recreate the existing database.

## Scope

### Phase A — Read-only physical diagnosis first

Before any source or DB mutation, inspect the exact call chain from:

```text
InstallationV0Window Open OBM-POS command
-> existing open/resume owner
-> LocalPosStartupService / route assessor
-> runtime profile repository
-> route/result mapping
-> MainWindow launch owner
```

Capture safe structural evidence only:

1. Exact enum/result type carrying `POS_RUNTIME_ROUTE_UNKNOWNFAILURE`.
2. Exact exception or unmatched branch that produces `UnknownFailure`.
3. Current target DB classification:
   - absent;
   - existing empty;
   - partially migrated;
   - schema current but profile missing;
   - DatabaseReady;
   - Activated/ApplicationReady;
   - inconsistent existing business DB.
4. Whether these objects physically exist:
   - EF migration history;
   - `TblPosRuntimeProfile`;
   - `TblPosRuntimeStateHistory`.
5. Pending migration count.
6. Current runtime-profile row count and semantic state, without exposing raw GUIDs or private values.
7. Whether the `Open OBM-POS` button was enabled from real `ApplicationReady/Activated` state or from a stale/incorrect predicate.
8. Exact inner exception/result that the UI currently hides behind `POS_RUNTIME_ROUTE_UNKNOWNFAILURE`.

Do not print passwords, full connection strings, JWTs, Pairing Codes, raw identity GUIDs, or private business data.

### Phase B — Classify the root cause

Use one exact classification:

```text
R1_PROFILE_STATE_MAPPING_DRIFT
R2_PROFILE_ROW_MISSING_OR_DUPLICATE
R3_SCHEMA_OR_MIGRATION_INCOMPLETE
R4_STARTUP_ROUTE_RESULT_MAPPING_DRIFT
R5_MAINWINDOW_LAUNCH_EXCEPTION_HIDDEN_AS_UNKNOWN
R6_OPEN_COMMAND_ENABLED_BEFORE_APPLICATIONREADY
R7_DATABASE_CONFIGURATION_OR_RUNTIME_CREDENTIAL_LOAD_FAILURE
R8_OTHER_PROVEN_ROOT_CAUSE
```

Do not guess. Include the exact source owner and safe evidence.

### Phase C — Minimal correction only

Apply only the correction required by the proven classification.

Preferred corrections:

- reuse the existing semantic mapping between V005 `ApplicationReady` and established `Activated`;
- reuse the existing `LocalPosStartupService` and runtime-profile repository;
- fix a missing route/result mapping rather than adding a second startup coordinator;
- surface a safe specific result code instead of collapsing known failures into `UnknownFailure`;
- if the DB is still only `DatabaseReady`, keep `Open OBM-POS` disabled and resume bootstrap rather than opening MainWindow;
- if migrations/bootstrap are incomplete, resume them idempotently through existing owners;
- if the existing DB contains user/business data with inconsistent profile state, stop in Recovery UI.

Do not manually insert `ApplicationReady`/`Activated` merely to make the button work.

## Non-negotiable prohibitions

Do not:

```text
DROP DATABASE
DROP SCHEMA
TRUNCATE
EnsureDeleted
recreate obm_pos_dev_v1_pg
copy another DB over it
manual CREATE TABLE
manual insert/update of runtime profile solely to force readiness
open MainWindow when local state is not ApplicationReady/Activated
create a second startup coordinator
create a second runtime-profile table/repository
change pairing/API/sync/SignalR/PlatformApp flows
```

## Required tests

Add focused tests for the proven failure boundary, including at minimum:

1. `ApplicationReady` maps to MainWindow route.
2. Existing equivalent `Activated` maps to MainWindow route.
3. `DatabaseReady` does not open MainWindow.
4. Missing profile does not become unknown failure; it returns the correct recoverable route/result.
5. Duplicate/inconsistent profile does not open MainWindow.
6. Known repository/config/migration failure returns a specific safe result code, not generic unknown when classification is available.
7. Open command enablement agrees with the same canonical route/readiness owner.

## Physical verification

Use the existing target DB. Do not delete it.

Prove in order:

1. Run the latest WPF build with visible label `prompt139`.
2. Load the existing protected pairing checkpoint.
3. Verify/resume local DB safely.
4. Pending migrations = 0.
5. Exactly one current runtime-profile row.
6. Current state is `ApplicationReady` or proven canonical equivalent `Activated`.
7. Click `Open OBM-POS` once.
8. MainWindow opens and remains responsive for at least 60 seconds.
9. Stop/leave API offline and confirm MainWindow remains usable.
10. Restart WPF twice; both launches open MainWindow directly without InstallationV0 flashing.

If physical verification cannot be completed, do not claim PASS.

## Build and report

Run:

- InstallationV0 build;
- focused InstallationV0 tests;
- any existing startup/runtime-profile focused tests needed for the exact owner.

Create and push:

```text
report/report139.md
```

The report must include:

- exact root-cause classification;
- exact call chain and owner reused;
- DB structural state and pending migration count;
- runtime-profile row count and semantic state;
- before/after route behavior;
- destructive-operation counts;
- build/test totals;
- physical MainWindow/restart results;
- private artifact version and SHA-256;
- coordination commit SHA.

PASS verdict only if all physical acceptance criteria succeed:

```text
OBM_WPF_V005_RUNTIME_ROUTE_APPLICATIONREADY_MAINWINDOW_OFFLINE_PHYSICALLY_PROVEN
```

Otherwise use one precise blocker verdict.