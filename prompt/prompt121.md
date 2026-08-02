# Prompt121 — Attach the existing WPF Phase2 readiness contract to the canonical Npgsql migration chain, complete local Phase2 once, and physically open MainWindow with API offline

## Starting checkpoint

Read completely:

```text
prompt/prompt119.md
report/report119.md
prompt/prompt120.md
prompt/prompt120_MAINWINDOW_PHYSICAL_PASS_GATE_ADDENDUM.md
report/report120.md
```

Verify the prompt120 private artifact before editing:

```text
E:\Project2026\RecoveryReports\WpfLocalPhase2ReadinessMainWindowV001
aggregate SHA-256:
894d8a903fc0fbb68051975fbee289166aca13d523d02115b7282c43c9809907
```

Accepted report120 verdict:

```text
BLOCKED_WPF_PHASE2_READINESS_CONTRACT
```

Accepted exact root-cause classification:

```text
L2_REQUIRED_READINESS_ENTITY_EXISTS_IN_SOURCE_BUT_IS_MISSING_FROM_ATTACHED_MIGRATION
```

The exact missing canonical WPF local readiness objects reported by the active startup/Phase2 code are:

```text
dbo."TblSchemaVersion"
dbo."TblSystemBaselineVersion"
dbo."TblPosRuntimeProfile"
dbo."TblPosRuntimeStateHistory"
dbo."Phase2TrialCompletionMarker"
```

The active assessor is:

```text
LocalPosStartupService.AssessAsync
```

The operator's application is still unusable because the normal WPF startup remains in InstallationV0 and does not open MainWindow. Build success, InstallationV0 stability, API health, or migration counts that omit these readiness objects are not PASS.

## Authoritative goal

Perform only this sequence:

```text
attach the five already-required readiness entities/tables to the one canonical WPF EF Core/Npgsql model
-> create exactly one attached migration
-> apply it exact-once to obm_pos_dev_v0_pg through the existing canonical migration boundary
-> execute the existing Phase2 local baseline/completion boundary once
-> prove local readiness/activation/station identity
-> stop API
-> normal WPF launch opens the real OBM-POS MainWindow directly
```

This is a narrow repair of an omitted migration contract. Do not redesign InstallationV0, authentication, sync, devices, or Turn Settings.

## Binding architecture and scope locks

### Local-first

After a successful pairing/redeem has already produced a durable protected Phase1 checkpoint and local Phase2 is valid:

```text
API offline or WpfJwt rejected/expired
=> remote auth/sync degraded only
=> MainWindow still opens
```

A remote `ProtectedHello` 401 must not erase or invalidate an already valid protected local Phase1 completion checkpoint. It must not prevent normal local startup after Phase2 completion.

Do not create a refresh-token contract in this task. Do not redeem again.

### Frozen work

Do not inspect, design, implement, or modify:

```text
Category Weight
Booking Weight
Price Weight save semantics
TblTenantPosDevice
TblPosLocal/device routing
CompanionApp registration
payment-terminal modeling
API destination routing
API grouped-sync happy path
canonical Provider behavior
POS1-POS10 PlatformAppV0 UI
Pairing Code API/UI
ApiServer schema/migrations/data
.env.local/.env.production cleanup
Firebase cleanup
```

The configuration/legacy cleanup requested by the operator will be a separate task only after MainWindow is physically restored.

### Prohibited actions

```text
no WPF DB drop/reset/recreate
no API DB mutation
no EnsureCreated
no manual CREATE TABLE/ad-hoc psql schema creation
no manual insertion of readiness/completion/activation markers
no copying protected state between ProductRoots
no fabricated station identity
no new ProductRoot
no second DbContext or migration lane
no second Phase2 service
no WpfJwt bypass
no new redeem
no refresh-token implementation
no business seed expansion
no TblLocalOutbox creation from installation seed
```

## Strict execution plan

## Phase 1 — Verify the existing entity and mapping contract before migration

Inspect the complete active WPF source for each of the five reported readiness objects.

For every object, record:

```text
entity CLR type
current source file
DbSet presence/absence
OnModelCreating/configuration registration presence/absence
schema/table mapping
primary key
required properties
indexes/unique constraints
foreign keys
read/write owner service
startup/readiness usage
Phase2 write usage
current model snapshot presence/absence
current migration-chain presence/absence
```

Do not invent new business semantics. Reuse the exact existing entity/property contract already read and written by current production startup/Phase2 services.

If an object is referenced only by raw SQL and no canonical entity exists, first prove that exact mismatch and add only the minimum canonical entity/configuration needed to represent the already-required table contract. Do not redesign it.

Resolve the exact status of `Phase2TrialCompletionMarker`: whether it is an entity/table contract or another mapped persistence object. Its attached migration representation must match the active reader/writer exactly.

Return one mapping matrix containing all five objects before editing.

## Phase 2 — Attach the five objects to the canonical WPF model

Use the existing canonical WPF PostgreSQL/Npgsql DbContext and design-time factory only.

Make the smallest source correction required so the canonical EF model includes all five objects.

Requirements:

```text
same canonical DbContext used by normal WPF runtime
same dbo schema convention already used by WPF
no duplicate entity/table mapping
no alternate context
no shadow replacement table
no renaming active table contracts without direct proof
no unrelated entity/schema changes
```

Run an EF model diff and prove the only intended schema additions are the five readiness objects plus their directly required keys/indexes/constraints.

If EF proposes unrelated drops, renames, column changes, or table rebuilds, stop and correct the model/snapshot mismatch before applying anything.

## Phase 3 — Create exactly one attached Npgsql migration

Create exactly one new canonical WPF EF Core migration with a clear identifier, for example conceptually:

```text
AttachPhase2ReadinessContract
```

Use the actual repository naming/timestamp convention.

The migration must:

```text
create exactly the missing readiness tables/objects
create only directly required indexes/constraints
update the canonical model snapshot
have a valid Down path
contain no business seed
contain no token/credential values
contain no tenant/pos hardcoding
contain no API schema objects
```

Do not hand-create tables outside EF migration execution.

Before applying, generate and review the SQL script. Record sanitized DDL object names and prove:

```text
unexpected DROP count = 0
unexpected ALTER existing business table count = 0
unexpected DELETE/UPDATE count = 0
unrelated created table count = 0
```

## Phase 4 — Apply migration exact-once to canonical WPF Development DB

Target only:

```text
provider: PostgreSQL/Npgsql
database: obm_pos_dev_v0_pg
environment: Development
canonical existing ProductRoot
```

Use the existing canonical WPF migration executor/Phase2 boundary. Do not use an alternate admin or fallback database lane.

Record before and after:

```text
DB_SAFE_NAME
MIGRATION_HISTORY_BEFORE
PENDING_MIGRATIONS_BEFORE
NEW_MIGRATION_ID
APPLY_COUNT
MIGRATION_HISTORY_AFTER
PENDING_MIGRATIONS_AFTER
```

Required:

```text
new migration apply count = 1
pending migrations after = 0
all five readiness objects exist
all five objects match the EF model
second migration execution is idempotent/no-op
```

Do not equate this migration success alone with installation PASS.

## Phase 5 — Execute the existing Phase2 local completion boundary once

After schema readiness is proven, invoke only the existing canonical `Install Local Database Baseline` / Phase2 service that owns:

```text
minimal baseline readiness
schema/system baseline version rows
local runtime profile
runtime state history
Phase2 completion marker
local activation/station identity state
```

The durable protected Phase1 checkpoint from the successful prior redeem is the local authorization source for completing the already-started installation. Preserve it.

Do not require a new redeem. Do not require the API to remain online merely to write local Phase2 state when the protected successful Phase1 checkpoint is valid.

If the existing Phase2 service still performs a remote hello, a 401 may be recorded as remote-auth degraded, but it must not erase the protected Phase1 checkpoint or prevent local Phase2 completion when the existing local checkpoint contract proves successful pairing.

Minimal baseline policy remains:

```text
required settings
required parameters
printer defaults
required default roles
readiness/version/runtime/activation records
```

Do not seed:

```text
employees
services
customers
Invoice
TblOutputInfo
Booking
turn/queue history
EventLog/EventDelivery history
transactional/runtime business data
```

Required installation invariant:

```text
LOCAL_SEED_NO_OUTBOX=true
TblLocalOutbox rows created by Phase2 = 0
```

Record exact row counts before/after only for the readiness/baseline tables needed to prove completion; do not expose business data.

Prove:

```text
LocalPosStartupService.AssessAsync result = Ready (or exact existing ready enum)
Phase2 completion = valid
local runtime activation = valid
station identity = valid
canonical ProductRoot identity matches DB runtime identity
```

Do not manually write any completion marker to force this result.

## Phase 6 — Physical offline MainWindow acceptance

Build the actual normal WPF executable and set the visible runtime label to:

```text
prompt121
```

Stop only the API helper/listener used for proof. Verify:

```text
127.0.0.1:7161 listener count = 0
```

Launch the exact operator-equivalent normal Visual Studio WPF Development profile/executable.

PASS requires all fields below to be physically true:

```text
visible label = prompt121
canonical ProductRoot selected = yes
canonical DB = obm_pos_dev_v0_pg
provider = Npgsql/PostgreSQL
pending migrations = 0
all five readiness objects present = yes
Phase2 completion valid = yes
local activation valid = yes
station identity valid = yes
retained protected Phase1 checkpoint preserved = yes
new redeem performed = no
API 7161 online = no
MainWindow opens directly = yes
InstallationV0 shown or flashed = no
MainWindow responsive for at least 60 seconds = yes
process crash/unhandled exception = no
```

Then close the app normally and launch the same profile a second time.

Second launch requires:

```text
MainWindow opens directly again = yes
InstallationV0 shown/flashed = no
API remains offline = yes
local readiness remains valid = yes
no new seed/migration replay side effects = yes
```

InstallationV0 remaining open or stable is not PASS. An enabled `Open OBM-POS` button is not a substitute for direct normal MainWindow startup.

## Phase 7 — Focused regression tests

Run focused tests for at minimum:

```text
all five readiness entities are part of the canonical EF model
migration creates all five objects
migration produces no unrelated schema operations
migration second run is no-op
Phase2 writes through the existing service only
Phase2 creates zero TblLocalOutbox rows
Phase2 does not seed operational/business tables
ready local state opens MainWindow with API offline
WpfJwt 401 does not invalidate completed local readiness
missing readiness schema remains InstallationV0
normal second startup remains MainWindow
```

Run WPF build after all changes.

Expected:

```text
focused tests pass
0 skipped
WPF build errors = 0
```

Physical MainWindow behavior overrides test/build success.

## Required verdict discipline

PASS verdict is allowed only as:

```text
OBM_WPF_PHASE2_READINESS_MIGRATION_COMPLETED_MAINWINDOW_OFFLINE_PHYSICALLY_RESTORED_READY_FOR_OPERATOR_SCREENSHOT
```

PASS is forbidden if any of these is not satisfied:

```text
MainWindow opens directly = yes
InstallationV0 shown/flashed = no
60-second MainWindow proof = yes
second-launch MainWindow proof = yes
API offline during both launches = yes
all five readiness objects attached and present = yes
Phase2 completed through existing service = yes
```

Narrow blocker verdicts only:

```text
BLOCKED_WPF_READINESS_ENTITY_MAPPING
BLOCKED_WPF_READINESS_MIGRATION_DIFF
BLOCKED_WPF_READINESS_MIGRATION_APPLY
BLOCKED_WPF_PHASE2_EXISTING_SERVICE
BLOCKED_WPF_LOCAL_ACTIVATION
BLOCKED_WPF_STATION_IDENTITY
BLOCKED_WPF_MAINWINDOW_ROUTING
BLOCKED_WPF_MAINWINDOW_PHYSICAL_PROOF
```

Do not return another generic schema/installation blocker.

## Required private artifact

Preserve all earlier artifacts unchanged. Create:

```text
E:\Project2026\RecoveryReports\WpfPhase2ReadinessMigrationMainWindowV001
```

Required evidence files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
PROMPT120_ARTIFACT_VERIFICATION.md
READINESS_ENTITY_MAPPING_MATRIX.md
CANONICAL_DB_CONTEXT_PROOF.md
MODEL_DIFF_BEFORE.md
MODEL_DIFF_AFTER.md
MIGRATION_ID_AND_FILES.md
MIGRATION_SQL_REVIEW.md
MIGRATION_APPLY_EXACT_ONCE.md
READINESS_OBJECT_EXISTENCE.md
PHASE2_SERVICE_CALL_CHAIN.md
PHASE2_BASELINE_AND_COMPLETION_PROOF.md
LOCAL_SEED_NO_OUTBOX_PROOF.md
LOCAL_ACTIVATION_AND_STATION_IDENTITY.md
API_OFFLINE_PROOF.md
MAINWINDOW_FIRST_LAUNCH_60_SECONDS.md
MAINWINDOW_SECOND_LAUNCH.md
FOCUSED_TEST_OUTPUT.txt
WPF_BUILD_OUTPUT.txt
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
FINAL_STATE.md
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

## Public report

Create and push:

```text
report/report121.md
```

The report must include:

```text
Verdict
Prompt120 artifact SHA verified yes/no
Root-cause L2 verified yes/no
Five readiness objects exact list
Entity/mapping attached yes/no per object
New migration identifier
Migration apply count
Pending migrations before/after
Unexpected schema operation counts
Phase2 existing service used yes/no
Phase2 completion valid yes/no
Local activation valid yes/no
Station identity valid yes/no
Phase2 TblLocalOutbox rows created count
Unrelated business seed rows created count
API 7161 offline proof yes/no
Retained checkpoint preserved yes/no
New redeem performed yes/no
Visible label
MainWindow opens directly yes/no
InstallationV0 shown/flashed yes/no
60-second MainWindow proof yes/no
Second-launch MainWindow proof yes/no
Focused test totals
WPF build totals
WPF DB reset performed no
API DB mutated no
TblTenantPosDevice changed no
Category Weight changed no
Booking Weight changed no
Operator MainWindow screenshot ready true/false
Manual POS1 test ready false
Private artifact path and aggregate SHA-256
```

Until the physical PASS gate succeeds:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=false
MANUAL_POS1_TEST_READY=false
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
```
