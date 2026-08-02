# Prompt124 — Rebuild and physically prove the canonical WPF self-provisioned fresh-install lifecycle

## Supersession and starting checkpoint

This prompt supersedes the execution direction of `prompt/prompt123.md`. Do **not** pre-create the new WPF database outside the product installation flow.

Starting evidence:

```text
report/report122.md
coordination commit:
85e2143ac1adfbfd410859cec0e1dd69f7daa2e1

report122 verdict:
BLOCKED_WPF_RUNTIME_PROFILE_PHYSICAL_SCHEMA

private artifact:
E:\Project2026\RecoveryReports\WpfRuntimeProfileLifecycleRebuildV001
aggregate SHA-256:
5a4a487f9ea4a89a0ba9fa0831c961649d547a81e9c543618c1776dcd531827b
```

Report122 proves that the inspected current local DB does not physically contain:

```text
dbo.TblPosRuntimeProfile
dbo.TblPosRuntimeStateHistory
```

It also proves no source change, no DB mutation, no token/header change, and no physical MainWindow proof were performed.

## Authoritative operator correction

The canonical new-install contract is:

```text
PlatformApp selects/provides the safe local POS database name as installation-plan metadata.
WPF receives that database name through the existing Pairing Code -> redeem/bootstrap handoff.
WPF itself creates the local PostgreSQL database during Phase2.
WPF applies the canonical attached migration chain.
WPF seeds the required minimal baseline.
WPF writes local runtime installation state.
WPF completes local application finalization.
WPF then opens MainWindow.
```

Do not create the target DB manually before WPF installation. Do not use an external pre-provisioned empty DB as acceptance proof.

For this Development proof, use the safe target name:

```text
obm_pos_dev_v1_pg
```

The name must come through the existing PlatformApp/pairing installation contract and be consumed by WPF. Do not hardcode it into general WPF production source.

Preserve `obm_pos_dev_v0_pg` unchanged as historical evidence. Do not drop, rename, reseed, truncate, repair, or copy business data from v0 into v1.

## Product architecture locks

### Local-first behavior

After local application installation is complete:

```text
local DB valid
+ local runtime profile complete
=> MainWindow opens
```

The following affect cloud/sync availability only and must not route an installed app back to InstallationV0:

```text
API offline
WpfJwt expired or rejected with 401
no refresh-token contract
SignalR offline
sync unavailable
```

### Installation phases

Preserve exactly the existing two installation phases:

```text
Phase1:
Pairing Code -> redeem -> WpfJwt/bootstrap state -> protected checkpoint

Phase2:
WPF local PostgreSQL creation -> migrations -> baseline seed -> runtime-profile transitions -> MainWindow
```

Do not add a third authentication or installation framework.

### Runtime state ownership

Use:

```text
TblPosRuntimeProfile
= one current local POS runtime/installation row
= startup source of truth

TblPosRuntimeStateHistory
= append-only transition/audit history
= never a startup eligibility dependency
```

Required lifecycle semantics:

```text
No target DB / no current profile
-> installation required

DatabaseReady equivalent
-> DB schema and baseline transaction committed
-> resume local application finalization

ApplicationReady equivalent
-> local app finalization committed
-> open MainWindow directly
```

First inspect the current enum/state vocabulary such as `Activated`. Reuse existing canonical state names when they already express these two semantics. Add explicit `DatabaseReady` and `ApplicationReady` states only if the source genuinely lacks equivalent states. Do not create duplicate synonyms.

## Frozen work

Do not implement or modify:

```text
Category Weight
Booking Weight
Price Weight save semantics
TblTenantPosDevice
API destination routing
canonical sync Provider behavior
SignalR architecture
CompanionApp or payment-terminal modeling
Firebase cleanup
.env cleanup
refresh-token architecture
```

Do not modify production/customer/reference DBs.

## Strict task scope

Execute only:

```text
1. Audit and prove the existing PlatformApp -> pairing-code -> redeem -> WPF installation-plan database-name handoff.
2. Repair that existing handoff only if the database name is not actually persisted or delivered.
3. Repair the WPF migration/model chain so a fresh WPF-created DB includes TblPosRuntimeProfile and TblPosRuntimeStateHistory.
4. Repair the canonical WPF Phase2 database-creation and installation lifecycle.
5. Make startup route from TblPosRuntimeProfile before remote API validation.
6. Run a genuine clean installation where obm_pos_dev_v1_pg does not exist beforehand and WPF creates it.
7. Complete Phase1 and Phase2 through production code and physically prove MainWindow.
8. Stop API and prove the completed app restarts directly into MainWindow offline.
```

## Required evidence intake

Read completely:

```text
OBM_POS_NewChat_Handoff_V001_2026-08-02.md when locally available
prompt/prompt118.md and all prompt118 addenda
report/report118.md
prompt/prompt119.md
report/report119.md
prompt/prompt120.md and addendum
report/report120.md
prompt/prompt121.md and all prompt121 addenda
prompt/prompt122.md
report/report122.md
prompt/prompt123.md for superseded-context awareness only
```

Verify the report122 private artifact and aggregate SHA before editing.

At minimum inspect complete source for:

```text
PlatformAppV0 POS1-POS10 create/select UI
pairing-code request entity/DTO/store/controller
safe target DB-name field and validation
pairing redeem response DTO/controller/service
WPF protected checkpoint/persistence model
WPF InstallationV0 Phase1 resume/redeem
WPF Phase2 DB-name resolver
WPF local PostgreSQL provisioning credential resolver
WPF DB create/provision service
Npgsql runtime connection creation
DbContext/model/migrations/snapshot
PosRuntimeProfileModels
PosRuntimeState
PostgresPosRuntimeProfileRepository
PostgreSqlPhase2ReferenceSeedExecutor
Phase2StartupHydrationService
LocalPosStartupService
App.xaml.cs startup routing
all EnsureCreated/EnsureDeleted/drop/reset helpers
```

Never expose token values, passwords, complete connection strings, passfile contents, OAuth secrets, raw private identities, or customer/business data.

## Phase 1 — Prove the actual installation-plan handoff

Map the complete active call chain:

```text
PlatformApp operator selects POS and target local DB name
-> pairing code record/request
-> API redeem response/bootstrap metadata
-> WPF protected checkpoint
-> Phase2 DB-name resolver
```

Record exact:

```text
PlatformApp UI/component
controller/service/method
DTO/property name
validation rules
storage owner
redeem response property
WPF persistence property
WPF Phase2 consuming method
```

Required contract:

```text
DB name only, not DB password
no complete connection string in pairing payload
safe PostgreSQL identifier validation
Development target = obm_pos_dev_v1_pg
```

If the active contract already works, preserve it. Do not create a second pairing endpoint or second installation-plan path.

If the DB-name field is missing or dropped in one active boundary, repair the smallest existing DTO/store/mapping path.

## Phase 2 — Repair the fresh-DB migration chain

The current source contains runtime-profile classes/repository but report122 proves the physical inspected DB lacks the two tables.

Inspect the canonical WPF DbContext model, entity mappings, migrations, and model snapshot.

Required outcome for every **fresh WPF-created DB**:

```text
dbo.TblPosRuntimeProfile exists
dbo.TblPosRuntimeStateHistory exists
correct keys/indexes/foreign-key relationships from the proven source contract
```

Preferred correction:

```text
attach the existing entities to the canonical WPF DbContext model
create exactly one new attached Npgsql EF migration for the missing runtime-profile schema
update the model snapshot
```

Do not edit historical applied migration files merely to hide the omission. Do not use `EnsureCreated`. Do not create tables through manual psql/SQL scripts.

Do not create these older readiness objects merely to satisfy stale code:

```text
TblSchemaVersion
TblSystemBaselineVersion
Phase2TrialCompletionMarker
```

Remove them from MainWindow eligibility if they are stale/test-only dependencies. EF migration history remains the schema-version source.

## Phase 3 — Canonical WPF self-provisioning

### Precondition

Before launching the clean installation proof, physically prove:

```text
TARGET_DB=obm_pos_dev_v1_pg
TARGET_DB_EXISTS=false
```

Do not create the DB manually.

### Database creation

WPF must use the existing protected local PostgreSQL provisioning boundary. Audit and preserve its security model.

Required sequence:

```text
resolve safe target DB name from protected installation plan
resolve local provisioning credential through the existing protected/local mechanism
connect to PostgreSQL maintenance DB
CREATE DATABASE obm_pos_dev_v1_pg with canonical UTF8/owner settings
connect to the new target DB using the canonical runtime role
apply all attached Npgsql migrations
verify pending migrations = 0
```

PostgreSQL `CREATE DATABASE` is outside a target-DB transaction. Do not pretend it is atomic with seed operations.

Rules:

```text
no hardcoded admin password
no password in pairing payload
no user-secrets dependency added
no interactive hidden fallback
no EnsureCreated
no drop/recreate when the target DB already exists
no automatic deletion of any non-disposable DB
```

If the target DB already exists on retry, inspect its runtime profile/migration state and resume idempotently; never drop it automatically.

## Phase 4 — Two-stage local installation lifecycle

### Transaction A — DatabaseReady

After database creation and migrations succeed, execute the existing canonical minimal baseline seed inside one target-DB transaction:

```text
BEGIN
  seed required settings
  seed required parameters
  seed required printer defaults
  seed required roles/runtime essentials only
  upsert exactly one TblPosRuntimeProfile current row to DatabaseReady-equivalent
  append exactly one TblPosRuntimeStateHistory DatabaseReady-equivalent transition
COMMIT
```

Requirements:

```text
baseline + current profile + history are atomic
failure rolls all three back
TblLocalOutbox rows created = 0
no employees/services/customers/invoices/bookings/queue/history/event-delivery business seed
idempotent retry creates no duplicate baseline/current row/history transition
```

### Local application finalization

After Transaction A, perform only the existing required local application finalization/activation work. Do not require API online after the protected local Phase1 checkpoint is already valid.

If the process exits here, next startup must see DatabaseReady-equivalent and resume finalization without recreating DB, reseeding baseline, or redeeming again.

### Transaction B — ApplicationReady

After local finalization succeeds:

```text
BEGIN
  update/upsert the one current TblPosRuntimeProfile row to ApplicationReady-equivalent
  append exactly one TblPosRuntimeStateHistory ApplicationReady-equivalent transition
COMMIT
```

No remote API call may be part of Transaction B.

## Phase 5 — Rewrite startup routing around local runtime profile

The normal WPF startup order must be:

```text
1. resolve canonical ProductRoot and safe DB name
2. determine whether target DB exists/reachable
3. query TblPosRuntimeProfile current row
4. route locally
5. only after MainWindow eligibility is known, start remote API/token/SignalR/sync as nonfatal background/degraded work
```

Required routing:

```text
DB absent
-> InstallationV0

DB exists but migrations/profile table absent
-> precise recoverable Phase2 migration/install state

profile missing
-> InstallationV0 Phase2/recovery

DatabaseReady-equivalent
-> resume local finalization

ApplicationReady-equivalent
-> MainWindow directly
```

Explicitly remove these as MainWindow blockers after ApplicationReady-equivalent:

```text
ProtectedHello success
WpfJwt validity/401
refresh token availability
API reachability
SignalR connectivity
sync availability
TblPosRuntimeStateHistory contents
legacy trial markers
```

Remote failures may update cloud status only.

## Phase 6 — Fresh physical installation proof

Use the real Development operator flow, not a test harness.

### Step A — Start full API and PlatformApp

Use the accepted full API runtime at:

```text
http://127.0.0.1:7161
```

Verify health/readiness. Use the existing authorized PlatformApp POS1-POS10 UI and Pairing Code path.

Create/select the Development POS and specify:

```text
obm_pos_dev_v1_pg
```

Generate a fresh pairing code through the real existing UI/API. No fabricated token or direct database insert.

### Step B — Run WPF clean installation

Use visible WPF label:

```text
prompt124
```

Redeem through the real WPF UI and prove:

```text
pairing/bootstrap checkpoint persisted
WPF creates obm_pos_dev_v1_pg itself
all migrations apply
pending migrations = 0
TblPosRuntimeProfile exists
TblPosRuntimeStateHistory exists
Transaction A commits DatabaseReady-equivalent
Transaction B commits ApplicationReady-equivalent
exactly one current profile row exists
expected history transitions exist exactly once
MainWindow opens
```

### Step C — Restart offline

After ApplicationReady-equivalent and MainWindow proof:

```text
stop the API helper/full API
prove port 7161 has no listener
close WPF normally
launch WPF again
```

Required:

```text
MainWindow opens directly
InstallationV0 does not flash/open
no Pairing Code required
no refresh token required
WpfJwt 401/offline state is cloud degraded only
MainWindow remains alive and responsive for at least 60 seconds
```

Repeat one more close/relaunch and prove the same result.

## Destructive migration/reset regression guards

Find and guard every destructive helper reachable from WPF startup/installation:

```text
EnsureDeleted
DROP DATABASE
DROP SCHEMA
TRUNCATE
reset/recreate helper
migration cleanup wrapper
```

Normal startup and normal installation retry must never call destructive reset.

Add focused tests proving:

```text
fresh install creates absent DB
migration preserves sentinel rows on upgrade
retry does not drop an existing DB
ApplicationReady startup never recreates/reseeds DB
protected/non-disposable DB names are rejected by reset helpers
reset requires explicit disposable Development authorization
```

## Tests and build

Run focused tests for:

```text
PlatformApp DB-name handoff
safe DB-name validation
WPF checkpoint DB-name persistence
fresh DB self-creation
migration creation of runtime-profile tables
Transaction A atomicity and rollback
Transaction B atomicity and rollback
idempotent resume from DatabaseReady
startup routing from current profile only
history has zero startup dependency
ApplicationReady + API offline opens MainWindow
WpfJwt 401 does not reopen InstallationV0
no destructive reset on startup/retry
LOCAL_SEED_NO_OUTBOX
```

Run WPF, InstallationV0, API, and PlatformApp builds only where changed.

Physical proof overrides build/test results.

## PASS gate

PASS requires all of the following:

```text
obm_pos_dev_v1_pg did not exist before WPF installation
PlatformApp supplied the safe DB name through the existing pairing contract
WPF created the DB itself
WPF applied attached migrations from zero
pending migrations = 0
TblPosRuntimeProfile physically exists
TblPosRuntimeStateHistory physically exists
exactly one current profile row exists
DatabaseReady-equivalent transition committed atomically with baseline
ApplicationReady-equivalent transition committed after local finalization
no manual SQL/data fabrication
MainWindow opened after installation
API was then stopped
second and third WPF launches opened MainWindow directly offline
InstallationV0 did not flash/open after ApplicationReady-equivalent
60-second offline MainWindow stability passed
```

PASS verdict:

```text
OBM_WPF_SELF_PROVISIONED_V1_INSTALLATION_RUNTIME_PROFILE_MAINWINDOW_OFFLINE_PHYSICALLY_PROVEN_READY_FOR_OPERATOR_SCREENSHOT
```

Any remaining InstallationV0 screen after ApplicationReady-equivalent is BLOCKED, not PASS.

## Status locks

Until PASS:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=false
MANUAL_POS1_TEST_READY=false
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
```

PASS may set only:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=true
MANUAL_POS1_TEST_READY=false
```

## Private artifact

Preserve all previous artifacts. Create:

```text
E:\Project2026\RecoveryReports\WpfSelfProvisionedV1InstallationV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ARTIFACT_VERIFICATION.md
PAIRING_DB_NAME_HANDOFF.md
TARGET_DB_ABSENT_BEFORE.md
WPF_DB_CREATION_CALL_CHAIN.md
PROVISIONING_CREDENTIAL_BOUNDARY.md
MIGRATION_MODEL_BEFORE.md
MIGRATION_MODEL_AFTER.md
ATTACHED_MIGRATION.md
FRESH_MIGRATION_PROOF.md
TRANSACTION_A_DATABASE_READY.md
TRANSACTION_B_APPLICATION_READY.md
STARTUP_ROUTING_BEFORE.md
STARTUP_ROUTING_AFTER.md
DESTRUCTIVE_PATH_AUDIT.md
API_ON_INSTALLATION_PROOF.md
API_OFFLINE_RESTART_PROOF.md
SECOND_RESTART_PROOF.md
FOCUSED_TEST_OUTPUT.txt
BUILD_OUTPUT.txt
UNIFIED_DIFF.patch
FINAL_STATE.md
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

## Public report

Write and push:

```text
report/report124.md
```

Required public fields:

```text
Verdict
Report122 artifact SHA verified
Prompt123 superseded acknowledged
PlatformApp DB-name owner/property/method
Redeem DB-name property
WPF checkpoint DB-name property
Target DB absent before yes/no
WPF created target DB yes/no
Target DB exact safe name
Provisioning owner/service/method
Attached migration identifier
Runtime profile/history tables created yes/no
Pending migrations after
Transaction A atomic proof
Transaction B atomic proof
Current profile row count/state
History transition counts
Manual SQL mutation count
Outbox rows created by installation
Startup reader class/method
History startup dependency count
API stopped/offline proof
MainWindow opened after installation yes/no
InstallationV0 shown after ApplicationReady yes/no
60-second offline proof
Second and third launch proof
Destructive startup/reset path count after
Build/test totals
Category/Booking/TblTenantPosDevice unchanged
Operator screenshot ready true/false
Manual POS1 test ready false
Private artifact path and aggregate SHA-256
```