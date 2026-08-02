# Prompt 120 — Restore the canonical WPF local Phase2 readiness state and physically open MainWindow with API offline

## Starting checkpoint

Prompt119 returned:

```text
BLOCKED_WPF_LOCAL_PHASE2_STATE_INVALID
```

Coordination references:

```text
report/report119.md
report119 commit:
71efaf4e6340516496550e4a5c9125a0e9d22544

prompt119 private artifact:
E:\Project2026\RecoveryReports\WpfLocalFirstRemoteAuthDegradedV001
aggregate SHA-256:
437e7ba8b2a47d6f392c6a4d619451452745e1307338b129a3b94970df16e9c7
```

Report119 proves:

```text
canonical ProductRoot resolved = yes
canonical WPF DB resolved = yes
local DB read-only access = yes
retained WpfJwt/checkpoint still exists and was not cleared
no new Pairing Code redeem occurred
API offline no longer crashes the WPF process
WPF remained alive/responding for 65 seconds in InstallationV0
MainWindow was not opened because the local gate returned SchemaMigrationRequired
local readiness tables/markers are missing
local Phase2 completion, local activation, and station identity are not yet proven
```

This is a real local-state blocker. Do not bypass it with token/API logic and do not open MainWindow blindly. Close only the canonical WPF Phase2 schema/baseline/activation boundary, then prove local-first MainWindow startup with the API completely offline.

## Authoritative architecture decisions

### Local-first

The local readiness decision is owned by local state:

```text
canonical PostgreSQL schema current
+ required minimal baseline present
+ local Phase2 completion/activation valid
+ local station/runtime identity valid
=> MainWindow may open
```

Remote state is separate:

```text
WpfJwt rejected/expired
API offline
SignalR offline
sync unavailable
=> remote/cloud degraded only
=> must not invalidate completed local installation
```

Do not add a refresh-token system in this task. Do not redeem a new Pairing Code. Do not delete, rotate, overwrite, or fabricate the retained credential/checkpoint.

### InstallationV0 remains two phases

```text
Phase 1:
Pairing Code -> redeem -> WpfJwt bootstrap credential -> protected local checkpoint

Phase 2:
local PostgreSQL migration/schema -> minimal baseline -> local completion/activation
```

This task closes only Phase 2 local readiness. It must not redesign Phase 1.

## Frozen work

Do not inspect, design, implement, or modify:

```text
Service Category Weight
Booking Weight
Price Weight save semantics
TblTenantPosDevice writer/schema/migration
TblPosLocal/TblTenantPosDevice routing
CompanionApp/terminal modeling
API destination routing
sync E2E/failure matrix
canonical Provider authentication behavior
POS1-POS10 PlatformAppV0 UI
Pairing Code API/UI
API database schema, role, grants, or data
```

Do not reset/drop/recreate either WPF or API database.

Do not use:

```text
EnsureCreated
manual application-table creation
manual completion-marker write
manual station-identity fabrication
copying protected state between ProductRoots
new DbContext/provider/startup lane
API-online requirement for Phase2 completion
```

## Strict scope

Execute only:

```text
1. Read and verify the complete prompt119 artifact.
2. Identify every exact WPF local readiness table, column, row, and marker required by the current startup gate.
3. Compare the canonical WPF model, attached migrations, model snapshot, and physical obm_pos_dev_v0_pg schema.
4. Classify exactly why startup returns SchemaMigrationRequired.
5. Correct the smallest source-of-truth defect in the existing WPF migration/Phase2 boundary.
6. Apply the accepted migration chain without DB reset.
7. Run the existing canonical Phase2 local-baseline/completion service when required.
8. Prove pending migrations = 0, required baseline exists, local activation is valid, and station identity is valid.
9. Start WPF with API port 7161 intentionally offline and prove MainWindow opens directly and remains stable.
10. Close and launch a second time and prove MainWindow opens again.
11. Run focused tests and build WPF.
```

## Required evidence intake

Read completely:

```text
OBM_POS_NewChat_Handoff_V001_2026-08-02.md when locally available
prompt/prompt107.md
report/report107.md
prompt/prompt117.md
report/report117.md
prompt/prompt118.md
report/report118.md
prompt/prompt119.md
report/report119.md
```

Read and verify:

```text
E:\Project2026\RecoveryReports\MainWpfDevResetExecutionV002
aggregate SHA-256:
47f68c634a5984611f3cb8b39ba3999f6005a558ad1e0d64bf998f7f4c2a0c58

E:\Project2026\RecoveryReports\WpfLocalFirstRemoteAuthDegradedV001
aggregate SHA-256:
437e7ba8b2a47d6f392c6a4d619451452745e1307338b129a3b94970df16e9c7
```

At minimum inspect complete current code for:

```text
startup readiness assessor returning SchemaMigrationRequired
all exact table/column existence probes
all Phase2 required-baseline probes
all Phase2 completion-marker readers/writers
all local runtime activation readers/writers
all local station-identity readers/writers
canonical WPF DbContext and design-time factory
all attached WPF migrations and model snapshot
existing Install Local Database Baseline command/service
minimal baseline manifest and transaction/checkpoint behavior
MainWindow eligibility predicate
InstallationV0 Open OBM-POS enablement predicate
```

Never expose tokens, passwords, complete connection strings, raw tenant/device identifiers, or business values.

Record before editing:

```text
PROMPT119_ARTIFACT_VERIFIED=true
CANONICAL_WPF_DB=obm_pos_dev_v0_pg
TASK_SCOPE=WPF_LOCAL_PHASE2_READINESS_ONLY
API_REQUIRED_FOR_PHASE2=false
NEW_REDEEM=FORBIDDEN
REFRESH_TOKEN_IMPLEMENTATION=FORBIDDEN
WPF_DB_RESET=FORBIDDEN
API_DB_MUTATION=FORBIDDEN
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
MANUAL_POS1_TEST_READY=false
OPERATOR_MAINWINDOW_SCREENSHOT_READY=false
```

## Phase 1 — Name the exact missing local contract

Do not report only `readiness tables missing` or `SchemaMigrationRequired`.

Produce an exact inventory:

```text
required object/table name
required columns/constraints/indexes
owning entity/DbSet/mapping
attached migration expected to create it
model snapshot presence
physical existence in obm_pos_dev_v0_pg
required row/marker semantics
startup assessor class/method/line
failure result code
```

Include every object used by:

```text
schema readiness
minimal baseline readiness
Phase2 completion
local runtime activation
local station identity
```

Capture the exact physical PostgreSQL errors/results. Use safe table/object names; do not expose private row values.

## Phase 2 — Classify the root cause exactly

Choose one primary classification:

```text
L1_ATTACHED_MIGRATION_EXISTS_BUT_PHYSICAL_DB_HISTORY_OR_APPLY_IS_INCOMPLETE
L2_REQUIRED_READINESS_ENTITY_EXISTS_IN_SOURCE_BUT_IS_MISSING_FROM_ATTACHED_MIGRATION
L3_PHYSICAL_SCHEMA_CURRENT_BUT_MINIMAL_BASELINE_ROWS_ARE_MISSING
L4_PHASE2_COMPLETION_OR_LOCAL_ACTIVATION_STATE_IS_MISSING
L5_LOCAL_STATION_IDENTITY_IS_MISSING_OR_INCONSISTENT
L6_READINESS_ASSESSOR_USES_STALE_OR_WRONG_TABLE_MAPPING
L7_MULTIPLE_EXACT_DEFECTS_WITH_PROVEN_ORDER
L8_OTHER_EXACTLY_PROVEN_LOCAL_PHASE2_DEFECT
```

Reconcile report107's prior `pending migrations = 0` result with report119's current missing-readiness result. State whether:

```text
report107 applied a migration chain that did not include the readiness schema
source migration chain changed later
readiness objects were never attached to the canonical migration
physical DB/object was altered after report107
startup assessor expects a stale table name
or another exact condition applies
```

Do not blame WpfJwt/API for a local schema/baseline failure.

## Phase 3 — Correct the source of truth narrowly

### When an attached migration is genuinely missing

Create or repair exactly one semantic WPF Npgsql migration in the accepted WPF migration assembly.

Requirements:

```text
model and migration agree
model snapshot updated
PostgreSQL/Npgsql only
no SQL Server/SQLite/InMemory fallback
no EnsureCreated
no manual physical-only patch
no speculative business tables
```

Apply through the existing WPF migration executor to `obm_pos_dev_v0_pg` without reset.

### When schema exists but baseline/completion is missing

Use only the existing canonical Phase2/local-baseline service.

Seed only the accepted minimal baseline required by current runtime, such as proven:

```text
settings
parameters
printer defaults
required roles
required runtime identity/setup markers
```

Do not seed:

```text
employees
services
customers
Invoice
TblOutputInfo
Booking
queue/Turn history
event/delivery history
```

Preserve the accepted seed rule:

```text
LOCAL_SEED_NO_OUTBOX
```

Installation seed does not create sync outbox rows. Real operator changes create outbox later.

### Completion and activation

Do not write completion/activation markers directly. The existing Phase2 service must write them only after proving:

```text
schema current
minimal baseline valid
local runtime identity valid
station identity valid
```

Use its existing transaction/checkpoint/rollback contract.

If the retained Phase1 checkpoint lacks a required local identity field, stop with a narrow blocker naming that exact field/boundary. Do not redeem again or fabricate identity.

## Phase 4 — Physical database proof

After correction prove:

```text
canonical DB = obm_pos_dev_v0_pg
provider = Npgsql/PostgreSQL
migration history contains the selected chain exactly once
pending migrations = 0
all exact readiness tables/columns/constraints/indexes exist
minimal baseline readiness = PASS
Phase2 completion = PASS
local activation = PASS
local station identity = PASS
no TblLocalOutbox rows created by Phase2 seed
no unrelated business rows seeded
```

Use rolled-back/non-persistent probes where appropriate.

Record any new schema contract for the later derived SQL-template export under:

```text
E:\Project2026\2SQL PostgreSQL
```

Do not finalize or overwrite the SQL templates in this task.

## Phase 5 — Physical local-first MainWindow proof

Build the latest WPF and display label:

```text
prompt120
```

Ensure the full API helper is stopped and port `7161` has no listener before launch.

Launch the exact operator-equivalent normal Visual Studio WPF profile.

Required proof:

```text
visible label = prompt120
canonical ProductRoot selected
canonical DB = obm_pos_dev_v0_pg
API port 7161 offline
retained WpfJwt/checkpoint preserved
no new redeem
MainWindow opens directly
InstallationV0 does not flash or replace MainWindow
local DB-backed UI remains responsive for at least 60 seconds
API/sync status may be offline/degraded
process does not exit/crash
```

Then:

```text
close normally
launch the same profile a second time
MainWindow opens directly again
InstallationV0 does not appear
retained credential/checkpoint/local activation remain preserved
```

The WpfJwt 401 or API outage must not invalidate the completed local Phase2 state.

## Phase 6 — Focused tests and build

Run focused tests for:

```text
exact readiness table/schema contract
migration-from-current application
pending migrations = 0
minimal baseline idempotency
LOCAL_SEED_NO_OUTBOX
Phase2 completion only after all prerequisites
local activation/station identity validation
MainWindow eligibility independent of remote API credential validity
API offline opens MainWindow after local completion
retained credential/checkpoint is not cleared
second-launch stability
prompt117 HttpRequestException regression
Firebase email/password remains absent
TblTenantPosDevice unchanged
```

Expected:

```text
all pass
0 skipped
WPF build errors = 0
```

Physical behavior overrides build/test results.

## End state

PASS requires:

```text
exact local readiness defect corrected at source
canonical WPF migration chain current
pending migrations = 0
minimal baseline valid
local Phase2 completion/activation valid
station identity valid
API offline
MainWindow physically opens directly for 60 seconds
second launch opens MainWindow directly again
retained WpfJwt/checkpoint preserved
no new redeem
InstallationV0 not shown on installed-local startup
Category Weight unchanged
Booking Weight unchanged
TblTenantPosDevice unchanged
OPERATOR_MAINWINDOW_SCREENSHOT_READY=true
MANUAL_POS1_TEST_READY=false
```

PASS verdict:

```text
OBM_WPF_LOCAL_PHASE2_READY_MAINWINDOW_OFFLINE_PHYSICALLY_RESTORED_READY_FOR_OPERATOR_SCREENSHOT
```

Narrow blockers only:

```text
BLOCKED_WPF_PHASE2_READINESS_CONTRACT
BLOCKED_WPF_PHASE2_MIGRATION_SOURCE
BLOCKED_WPF_PHASE2_MIGRATION_APPLY
BLOCKED_WPF_PHASE2_MINIMAL_BASELINE
BLOCKED_WPF_PHASE2_LOCAL_ACTIVATION
BLOCKED_WPF_PHASE2_STATION_IDENTITY
BLOCKED_WPF_MAINWINDOW_OFFLINE_PHYSICAL_PROOF
BLOCKED_WPF_PHASE2_FOCUSED_TESTS
```

Every blocked result must include the exact table/marker/service/method, sanitized error/SQLSTATE when available, migration/baseline/activation state, whether any local DB write occurred, and:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=false
MANUAL_POS1_TEST_READY=false
```

Do not return another generic `SchemaMigrationRequired` or `Phase2 incomplete` blocker.

## Required private artifact

Preserve previous artifacts unchanged. Create:

```text
E:\Project2026\RecoveryReports\WpfLocalPhase2ReadinessMainWindowV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ARTIFACT_VERIFICATION.md
PROMPT119_BLOCKER_INTAKE.md
EXACT_READINESS_OBJECT_INVENTORY.md
MIGRATION_CHAIN_RECONCILIATION.md
ROOT_CAUSE_CLASSIFICATION.md
READINESS_SOURCE_BEFORE.md
READINESS_SOURCE_AFTER.md
MIGRATION_PROOF.md
PHYSICAL_SCHEMA_PROOF.md
MINIMAL_BASELINE_PROOF.md
LOCAL_SEED_NO_OUTBOX_PROOF.md
PHASE2_COMPLETION_PROOF.md
LOCAL_ACTIVATION_PROOF.md
STATION_IDENTITY_PROOF.md
API_OFFLINE_MAINWINDOW_PROOF.md
SECOND_LAUNCH_PROOF.md
FOCUSED_TEST_OUTPUT.txt
FINAL_STATE.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

## Public report

Create and push only:

```text
report/report120.md
```

Include:

```text
Verdict
Prompt119 artifact SHA verified yes/no
Root-cause classification L1-L8
Exact missing readiness table/marker names
Readiness assessor class/method
Attached migration created/repaired yes/no and identifier
Migration applied exact-once yes/no
WPF pending migrations count
Minimal baseline readiness yes/no
Phase2 completion proof yes/no
Local activation proof yes/no
Station identity proof yes/no
LOCAL_SEED_NO_OUTBOX proof yes/no
Phase2 outbox rows created count
Unrelated business seed rows created count
Canonical WPF DB proof yes/no
API port 7161 offline during acceptance yes/no
Retained credential/checkpoint preserved yes/no
New redeem performed yes/no
MainWindow opens directly yes/no
InstallationV0 shown on installed-local launch yes/no
60-second stability yes/no
Second-launch MainWindow proof yes/no
WPF focused test totals
WPF build totals
WPF DB reset performed yes/no
API DB mutated yes/no
TblTenantPosDevice changed yes/no
Category Weight changed yes/no
Booking Weight changed yes/no
Operator MainWindow screenshot ready true/false
Manual POS1 test ready false
Production/customer/reference DB mutated yes/no
Private artifact yes/no
Aggregate SHA-256
```

Do not expose tokens, passwords, complete connection strings, raw identities, or private business data.
