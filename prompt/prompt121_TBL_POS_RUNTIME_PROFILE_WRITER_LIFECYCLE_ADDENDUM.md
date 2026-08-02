# Prompt121 Addendum — Implement the missing TblPosRuntimeProfile writer lifecycle inside the existing Phase2 transaction

This addendum is binding and overrides any broader or conflicting migration-first/readiness-framework interpretation in `prompt/prompt121.md` and earlier prompt121 addenda.

## Authoritative operator evidence

The physical WPF database already contains:

```text
TblPosRuntimeProfile
TblPosRuntimeStateHistory
```

However, the operator has verified that the installation flow currently writes no row into `TblPosRuntimeProfile`; the table is empty. Therefore the primary defect is not that the runtime-profile table is absent. The primary defect is that the installation/Phase2 completion flow has no effective writer lifecycle for the current runtime profile.

Do not create a replacement readiness framework or treat five candidate readiness objects as automatically required.

## Canonical startup and installation contract

Use exactly this model unless current source proves an existing equivalent contract that is narrower and production-valid:

```text
TblPosRuntimeProfile
= the single current-state source used by startup

TblPosRuntimeStateHistory
= append-only/audit history of state transitions
= never an independent MainWindow startup prerequisite
```

Startup decision:

```text
canonical local DB reachable
+ current TblPosRuntimeProfile row exists
+ current installation/runtime status is the existing source-defined Installed/Ready equivalent
=> open MainWindow

no current row, or current status is not installation-complete
=> open InstallationV0/Phase2 recovery
```

Remote API reachability, WpfJwt validity, SignalR, and sync availability do not participate in this local installation-complete predicate.

## Exact root cause to prove

Before editing, trace and document:

```text
1. The exact Phase2/baseline service and transaction boundary.
2. The exact successful final commit branch.
3. Whether any current source method attempts to create/update TblPosRuntimeProfile.
4. Why no row exists after the prior installation/seed attempt.
5. The exact entity fields, key/cardinality, existing status enum/constants, and unique constraint.
6. The exact startup reader/query/predicate for TblPosRuntimeProfile.
7. Whether TblPosRuntimeStateHistory has an existing writer and whether it incorrectly gates startup.
```

Return one primary classification:

```text
W1_PHASE2_HAS_NO_RUNTIME_PROFILE_WRITER
W2_WRITER_EXISTS_BUT_IS_NOT_CALLED
W3_WRITER_CALLED_OUTSIDE_TRANSACTION_AND_ROLLED_BACK_OR_FAILED
W4_WRITER_USES_WRONG_DB_SCHEMA_CONTEXT_OR_KEY
W5_WRITER_STATUS_VALUE_DOES_NOT_MATCH_STARTUP_PREDICATE
W6_STARTUP_READER_USES_WRONG_SCHEMA_CONTEXT_OR_KEY
W7_OTHER_EXACTLY_PROVEN_RUNTIME_PROFILE_LIFECYCLE_DEFECT
```

## Required implementation

Repair the smallest existing Phase2 boundary.

On successful Phase2 baseline installation, in the same local PostgreSQL transaction as the required baseline seed:

```text
1. Upsert exactly one current TblPosRuntimeProfile row for the local POS/runtime identity using the existing source-defined key.
2. Set the existing source-defined installation-complete/runtime-ready status; do not invent a new status when an existing enum/constant exists.
3. Set/update only fields genuinely required by the existing entity contract.
4. Append one TblPosRuntimeStateHistory transition row when the existing history contract requires it.
5. Commit baseline rows + current profile + history atomically.
```

If any operation fails:

```text
rollback all baseline/profile/history changes
startup must continue to classify installation as incomplete
```

Idempotency requirements:

```text
re-running completed Phase2 does not create duplicate current profiles
same completion state does not create unbounded duplicate history rows
existing valid current profile is preserved or updated through the canonical owner
```

## Startup reader correction

After the writer lifecycle is correct, make startup read `TblPosRuntimeProfile` as the current local installation-state source.

Requirements:

```text
TblPosRuntimeProfile current Ready/Installed-equivalent row => MainWindow eligible
TblPosRuntimeProfile empty/not-ready => InstallationV0
TblPosRuntimeStateHistory => audit only; startup dependency count must be 0
API offline/401 => cloud degraded only; must not change the local profile or reopen InstallationV0
```

Do not open MainWindow merely because any arbitrary row exists. Use the exact existing status/identity predicate proven from source.

## Migration prohibition by default

Because both physical tables already exist, do not create a migration unless direct schema comparison proves that the exact required column, key, constraint, or mapping is missing.

Before any migration, record:

```text
PHYSICAL_TABLE_EXISTS=true
CURRENT_COLUMNS=<names only>
MODEL_EXPECTED_COLUMNS=<names only>
SCHEMA_DIFF=<exact>
MIGRATION_REQUIRED=yes/no
```

A migration is forbidden when the only defect is an empty table or missing writer invocation.

## Prohibited shortcuts

Do not:

```text
manually INSERT/UPDATE TblPosRuntimeProfile using psql or ad hoc SQL
generate a fake completion marker
create TblSchemaVersion/TblSystemBaselineVersion/Phase2TrialCompletionMarker merely to satisfy old assessor logic
make TblPosRuntimeStateHistory a startup gate
reset/drop/recreate the WPF database
use EnsureCreated
redeem a new pairing code
add refresh-token behavior
modify API/schema/sync/TblTenantPosDevice/Category Weight/Booking Weight
```

## Required physical proof

Use the actual canonical Development WPF DB and existing Phase2 UI/service.

### Before Phase2 completion

```text
TblPosRuntimeProfile current-row count = 0 or not-ready
startup routes to InstallationV0
```

### Execute existing Phase2 completion

```text
one canonical transaction
baseline seed succeeds
TblPosRuntimeProfile current row written by application service, not manual SQL
status = exact existing Ready/Installed equivalent
history transition written when required
TblLocalOutbox rows created = 0
```

### After completion with API offline

```text
API port 7161 has no listener
normal WPF startup reads TblPosRuntimeProfile
MainWindow opens directly
InstallationV0 does not appear or flash
MainWindow remains responsive for at least 60 seconds
close normally
launch again
second launch again opens MainWindow directly
WpfJwt 401/offline does not clear or downgrade the local profile
```

## PASS gate

PASS is forbidden unless all are true:

```text
TblPosRuntimeProfile writer owner/service/method proven
writer invoked by existing Phase2 completion path
profile upsert committed atomically with baseline
current profile row exists and is Ready/Installed-equivalent
TblPosRuntimeStateHistory is audit-only
no manual DB state edit used
API offline
MainWindow opens directly
InstallationV0 shown/flashed = no
60-second MainWindow proof = yes
second-launch MainWindow proof = yes
```

PASS verdict:

```text
OBM_WPF_RUNTIME_PROFILE_WRITER_COMPLETED_MAINWINDOW_OFFLINE_PHYSICALLY_RESTORED_READY_FOR_OPERATOR_SCREENSHOT
```

Otherwise return a narrow blocker identifying the exact writer/transaction/reader boundary.

## Required report fields

`report/report121.md` must include:

```text
Verdict
Primary classification W1-W7
Physical TblPosRuntimeProfile exists yes/no
Before current-row count/status
Phase2 writer owner/service/method
Writer previously absent/not-called/wrong reason
Transaction owner and atomicity proof
Current-profile key/cardinality
Exact Ready/Installed status value from source
After current-row count/status
History writer and startup dependency count
Migration created yes/no and exact justification
Manual DB insert/update count 0
TblLocalOutbox rows created 0
API offline proof
MainWindow opens directly yes/no
InstallationV0 shown/flashed yes/no
60-second proof yes/no
Second-launch proof yes/no
Operator MainWindow screenshot ready true/false
Manual POS1 test ready false
Private artifact and aggregate SHA-256
```
