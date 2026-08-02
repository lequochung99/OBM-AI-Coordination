# Prompt126 — Rebuild the canonical two-phase OBM-POS installation flow: local database first, cloud pairing second

## Authoritative correction

This prompt supersedes the implementation direction of prompt125.

The prior direction incorrectly made PlatformApp/pairing own the local PostgreSQL database name. That is not the product contract.

The canonical installation has exactly two independent phases:

```text
Phase 1 — Local OBM-POS installation
WPF collects local PostgreSQL connection/provisioning inputs directly from the operator:
- host, with a safe local default
- port, with the existing safe PostgreSQL default
- PostgreSQL username
- PostgreSQL password
- target local database name

WPF then creates the local database when absent, applies migrations, seeds the required baseline, records local runtime readiness, and completes local application activation.

No Pairing Code is required for Phase 1.

Phase 2 — Cloud/API authorization
After local installation is complete, the operator enters a Pairing Code.
WPF redeems it, receives the existing WpfJwt/bootstrap credential, and persists it securely on the PC.

Phase 2 enables API/sync/cloud functionality but does not determine whether the local OBM-POS application can run.
```

PlatformApp owns Tenant/POS selection and Pairing Code issuance. PlatformApp does not own or transmit local PostgreSQL host, username, password, or database name.

## Starting evidence

Read completely:

```text
report/report120.md
report/report122.md
report/report124.md
prompt/prompt124.md
prompt/prompt125.md
```

Verify the private artifacts referenced by those reports when locally available.

Accepted evidence:

```text
- Current canonical Development DB obm_pos_dev_v0_pg was emptied during prior reset/migration work.
- The normal WPF startup returned to InstallationV0 because no valid local installed state remained.
- TblPosRuntimeProfile and TblPosRuntimeStateHistory repository/model classes exist in source, but the active attached EF migration/model chain did not physically create those tables in the inspected DB.
- CleanLocalDatabaseService.CreateCleanDatabaseAsync already accepts a local TargetDatabaseName.
- Pairing/redeem currently returns WpfJwt only; no refresh-token contract exists.
- Retained WpfJwt loss or API availability must not control local MainWindow eligibility.
```

## Product contract locks

### Phase 1 is fully local

Phase 1 must not require:

```text
PlatformApp online
Pairing Code
WpfJwt
API protected hello
SignalR
sync availability
Tenant/POS cloud authorization
```

The local operator provides the PostgreSQL inputs directly in WPF.

### Phase 2 is cloud authorization only

Pairing Code redemption may occur only after local installation reaches ApplicationReady.

An optional early Pairing Code/API preflight may be retained only if the current UI already supports it, but before ApplicationReady it may do no more than:

```text
check API reachability
check that the code format/request can be accepted safely
show that cloud authorization will be possible later
```

It must not:

```text
become a prerequisite for database creation
persist an active cloud credential before local installation completion
write local installation completion state
control MainWindow eligibility
```

### Local-first behavior

Once the local runtime profile is ApplicationReady:

```text
API offline
WpfJwt missing
WpfJwt expired
WpfJwt rejected with 401
no refresh-token path
SignalR unavailable
sync failure
```

must result only in:

```text
Cloud/API status = Offline or Degraded
```

They must not:

```text
open InstallationV0
close MainWindow
block checkout/local settings/local database use
clear local runtime readiness
clear local activation
clear the database configuration
```

## Frozen work

Do not implement or modify:

```text
Category Weight
Booking Weight
Price Weight save semantics
TblTenantPosDevice
API destination routing
POS2 pull/apply/ACK
CompanionApp/payment-terminal modeling
Firebase cleanup
.env cleanup
refresh-token architecture
new sync uploader/bot/endpoint
```

Do not reset/drop/recreate `obm_pos_dev_v0_pg`. Preserve it as damaged-state evidence.

Do not pre-create the new target DB outside WPF.

## Strict goal

Rebuild and physically prove this exact flow:

```text
WPF Phase 1 local installer
-> operator enters local PostgreSQL host/port/user/password/database name
-> WPF validates the local PostgreSQL connection
-> WPF creates the target DB when absent
-> WPF applies the canonical attached Npgsql migrations
-> WPF seeds the minimal baseline
-> WPF records DatabaseReady atomically with the baseline
-> WPF completes local application finalization
-> WPF records ApplicationReady
-> WPF opens MainWindow with API offline
-> operator optionally performs Phase 2 Pairing Code redeem
-> WPF stores WpfJwt securely
-> token/API failure never downgrades the local ApplicationReady state
```

Use this Development target for physical proof:

```text
obm_pos_dev_v1_pg
```

The name must be entered in WPF Phase 1 UI. It must not come from PlatformApp and must not be hardcoded in production source.

## Phase 0 — Audit the active installation and startup call chains

Before editing, map the exact current source call chains for:

```text
WPF startup window selection
InstallationV0 Phase 1 UI
local PostgreSQL input collection
local DB settings persistence
CleanLocalDatabaseService.CreateCleanDatabaseAsync
maintenance DB connection used for CREATE DATABASE
runtime DB connection resolution
canonical WPF DbContext
migration execution
baseline seed transaction
runtime-profile repository
application finalization/activation
Pairing Code redeem
WpfJwt protected persistence
MainWindow eligibility
```

Identify and report:

```text
LOCAL_DB_INPUT_UI_OWNER
LOCAL_DB_SETTINGS_MODEL
LOCAL_DB_SETTINGS_PROTECTED_STORAGE_OWNER
DATABASE_CREATE_SERVICE
MIGRATION_OWNER
BASELINE_TRANSACTION_OWNER
RUNTIME_PROFILE_WRITER_OWNER
APPLICATION_FINALIZATION_OWNER
PAIRING_REDEEM_OWNER
WPF_TOKEN_STORAGE_OWNER
STARTUP_ROUTING_OWNER
```

Do not create parallel services when an existing owner can be corrected.

## Phase 1 — Rebuild the WPF local-installation UI and local configuration contract

The WPF local installation phase must collect:

```text
Host
Port
PostgreSQL username
PostgreSQL password
Target local database name
```

Requirements:

```text
- Host defaults to the existing safe local PostgreSQL host, normally 127.0.0.1 or localhost according to current conventions.
- Port defaults to the existing PostgreSQL port convention.
- Database name is validated by the one existing/canonical PostgreSQL DB-name validator.
- Username/password are never logged or written to reports.
- Connection strings are never printed in full.
- Password is never stored in plaintext.
- Reuse the existing protected local DB-settings mechanism; do not invent a second configuration store.
- Normal production source must have zero active fallback to Phase2TrialConstants.ApprovedDatabaseName or another fixed Development DB name.
```

The local DB settings must survive restart through the existing protected local configuration boundary because the runtime app must reconnect later.

Record only safe proof fields:

```text
host classification
port
safe DB name
username-present marker
password-protected marker
provider=Npgsql/PostgreSQL
```

## Phase 2 — Attach the runtime-profile physical schema to the canonical WPF migration chain

The fresh target DB must physically contain:

```text
dbo."TblPosRuntimeProfile"
dbo."TblPosRuntimeStateHistory"
```

Use the existing runtime-profile domain/repository classes when viable:

```text
Services/RuntimeProfile/PosRuntimeState.cs
Services/RuntimeProfile/PosRuntimeProfileModels.cs
Services/RuntimeProfile/PostgresPosRuntimeProfileRepository.cs
```

Requirements:

```text
- Attach the entities/mappings to the one canonical WPF DbContext/model.
- Create exactly one attached Npgsql migration when the active migration chain lacks these tables.
- Update the canonical model snapshot.
- Do not create TblSchemaVersion, TblSystemBaselineVersion, or Phase2TrialCompletionMarker merely to satisfy old readiness logic.
- Do not use EnsureCreated.
- Do not create tables with manual psql/pgAdmin SQL.
```

Use existing state names if they already express the required semantics. If current source only has `Activated`, first prove its meaning. Avoid duplicate state vocabularies. The required semantic stages are:

```text
DatabaseReady
ApplicationReady
```

## Phase 3 — WPF self-provisions the target DB

Physical precondition:

```text
obm_pos_dev_v1_pg does not exist
```

Prove absence without creating it manually.

From the actual WPF InstallationV0 UI, enter the local DB inputs and execute Phase 1.

Required flow:

```text
validate local PostgreSQL access
-> connect to the maintenance database using the operator-supplied local credential
-> create obm_pos_dev_v1_pg when absent
-> connect to obm_pos_dev_v1_pg through the canonical runtime DB settings
-> apply the attached WPF migrations from zero
-> pending migrations = 0
```

Safety requirements:

```text
- Existing target DB must never be silently dropped or cleared.
- If the target DB already exists and is non-empty/inconsistent, stop with a recoverable explicit decision; do not auto-reset it.
- No normal startup path may call EnsureDeleted, DROP DATABASE, DROP SCHEMA, TRUNCATE, or a destructive reset helper.
- Destructive test helpers must be unreachable without an explicit disposable-development authorization.
```

## Phase 4 — Transaction A: baseline plus DatabaseReady

Use the existing Phase 2/baseline seed service, corrected as necessary.

In one local PostgreSQL transaction:

```text
seed the minimal required baseline
upsert exactly one current TblPosRuntimeProfile row with DatabaseReady-equivalent state
append one TblPosRuntimeStateHistory DatabaseReady transition
commit
```

The minimal baseline may include only the currently accepted required categories, such as:

```text
required settings
required parameters
printer defaults
required built-in roles
required local setup/activation records proven by source
```

Do not seed:

```text
employees
services
customers
Invoice
TblOutputInfo
Booking
operational history
event/delivery history
```

Installation seed must create:

```text
TblLocalOutbox rows = 0
```

Atomicity proof is mandatory:

```text
- On injected failure before commit: no partial baseline, no DatabaseReady profile, no DatabaseReady history.
- On success: baseline, current profile, and history commit together.
```

## Phase 5 — Transaction B: local application finalization plus ApplicationReady

After Transaction A commits, complete the existing local application finalization/activation boundary.

This stage must not require API/Pairing/WpfJwt.

Then, in a second short local PostgreSQL transaction:

```text
upsert the same current TblPosRuntimeProfile row to ApplicationReady-equivalent state
append one TblPosRuntimeStateHistory ApplicationReady transition
commit
```

Requirements:

```text
- Exactly one current profile row for the local installation.
- History is append-only audit.
- Restart while state=DatabaseReady resumes finalization; it must not recreate/reseed the DB.
- Replaying finalization when already ApplicationReady is idempotent and creates no duplicate transition.
```

Do not hold one database transaction open across UI waits, network calls, or process restart.

## Phase 6 — Rewrite startup routing around the local runtime profile

Startup must evaluate local state before remote/cloud state.

Canonical routing:

```text
No usable protected local DB settings
-> InstallationV0 local configuration phase

Configured DB absent
-> InstallationV0 local DB creation phase

DB exists but no current runtime-profile row
-> recover/resume local installation; do not infer ApplicationReady

Current state = DatabaseReady
-> resume local application finalization

Current state = ApplicationReady
-> open production MainWindow directly
```

`TblPosRuntimeStateHistory` has zero startup-gating responsibility.

Remove the following from MainWindow eligibility after ApplicationReady:

```text
Pairing Code existence
WpfJwt validity
ProtectedHello success
API reachability
SignalR availability
sync availability
legacy trial markers
```

Remote/cloud state is evaluated after MainWindow selection and exposed as connected/degraded/offline status only.

## Phase 7 — Phase 2 cloud pairing after local installation

After MainWindow or the post-installation completion UI is available, allow the operator to enter a Pairing Code.

Use the existing canonical chain:

```text
PlatformApp creates Pairing Code for Tenant/POS
-> WPF redeem
-> API returns existing WpfJwt/bootstrap credential
-> WPF persists it using the existing DPAPI/protected token store
```

Requirements:

```text
- Do not add local DB host/user/password/name to PlatformApp or pairing payloads.
- Do not create a refresh-token contract in this task.
- Pairing failure leaves ApplicationReady unchanged.
- API offline leaves ApplicationReady unchanged.
- WpfJwt 401 leaves ApplicationReady unchanged.
- Re-pairing updates only cloud credential state, not local installation state.
```

Optional pre-install API/pairing preflight, if retained, must remain nonbinding and must not persist an active token before ApplicationReady.

## Phase 8 — Physical acceptance

Use visible WPF label:

```text
prompt126
```

### Case A — clean local installation with no Pairing Code

Prove physically:

```text
obm_pos_dev_v1_pg absent before
WPF local UI receives host/port/user/password/DB name
WPF creates obm_pos_dev_v1_pg
WPF applies migrations from zero
pending migrations = 0
TblPosRuntimeProfile exists
TblPosRuntimeStateHistory exists
baseline commit succeeds
current profile row count = 1
current state = ApplicationReady-equivalent
DatabaseReady history count = 1
ApplicationReady history count = 1
TblLocalOutbox installation rows = 0
API port 7161 has no listener
no Pairing Code was redeemed
MainWindow opens directly
InstallationV0 closes and does not reopen
MainWindow remains responsive for at least 60 seconds
```

### Case B — restart while API is offline and no token exists

Close normally and launch again.

Prove:

```text
MainWindow opens directly
InstallationV0 does not flash
local DB remains intact
profile remains ApplicationReady
no new baseline rows
no duplicate history transition
no DB reset/recreate
```

### Case C — cloud pairing after local installation

Start the full API/PlatformApp only after Cases A/B pass.

Create/redeem one valid Pairing Code through the existing PlatformApp Tenant/POS flow.

Prove:

```text
WpfJwt is returned
protected token record is persisted
local DB name/config does not come from PlatformApp
local runtime profile remains ApplicationReady
```

Stop the API and restart WPF again.

Prove MainWindow still opens directly with cloud status Offline/Degraded.

## Required tests and guards

Run focused tests for:

```text
local installation does not require Pairing Code/API
local DB input validation
protected DB-settings persistence and read-back
fresh DB create only when absent
existing DB is not silently dropped/cleared
attached migration creates runtime profile/history tables
Transaction A rollback/commit atomicity
Transaction B idempotency
DatabaseReady restart resume
ApplicationReady startup opens MainWindow offline
missing/expired/rejected WpfJwt does not reopen InstallationV0
Pairing failure does not change local runtime profile
history does not gate startup
no legacy fixed DB-name fallback
no installation outbox rows
```

Run WPF build and relevant InstallationV0 tests.

Physical proof overrides unit-test/build success.

## PASS gate

PASS requires all of these:

```text
WPF directly collected local PostgreSQL inputs
PlatformApp did not own local DB configuration
WPF self-created obm_pos_dev_v1_pg from an absent state
runtime profile/history tables were created by attached migration
baseline + DatabaseReady committed atomically
ApplicationReady was recorded through the production lifecycle writer
MainWindow opened with API offline and no Pairing Code/token
second launch again opened MainWindow directly
later Pairing Code redeem persisted WpfJwt without changing local readiness
final API-offline restart still opened MainWindow directly
```

PASS verdict:

```text
OBM_WPF_TWO_PHASE_LOCAL_FIRST_INSTALLATION_AND_POST_INSTALL_PAIRING_PHYSICALLY_PROVEN_READY_FOR_OPERATOR_SCREENSHOT
```

Narrow blockers only:

```text
BLOCKED_WPF_LOCAL_DB_INPUT_CONTRACT
BLOCKED_WPF_SELF_DATABASE_PROVISIONING
BLOCKED_WPF_RUNTIME_PROFILE_ATTACHED_MIGRATION
BLOCKED_WPF_BASELINE_DATABASE_READY_TRANSACTION
BLOCKED_WPF_APPLICATION_READY_FINALIZATION
BLOCKED_WPF_LOCAL_FIRST_MAINWINDOW_STARTUP
BLOCKED_WPF_POST_INSTALL_PAIRING
BLOCKED_WPF_PHYSICAL_MAINWINDOW_PROOF
```

Do not report PASS while InstallationV0 remains visible instead of the production MainWindow.

## Required artifact and report

Preserve all previous artifacts. Create a new versioned private artifact under:

```text
E:\Project2026\RecoveryReports\WpfTwoPhaseLocalFirstInstallationV001
```

Include at minimum:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
BEFORE_ARCHITECTURE.md
AFTER_ARCHITECTURE.md
LOCAL_DB_INPUT_CONTRACT.md
PROTECTED_DB_SETTINGS_PROOF.md
ATTACHED_MIGRATION_PROOF.md
SELF_DATABASE_CREATION_PROOF.md
TRANSACTION_A_DATABASE_READY.md
TRANSACTION_B_APPLICATION_READY.md
STARTUP_ROUTING_PROOF.md
NO_PAIRING_LOCAL_MAINWINDOW_PROOF.md
POST_INSTALL_PAIRING_PROOF.md
API_OFFLINE_RESTART_PROOF.md
DESTRUCTIVE_PATH_GUARDS.md
FOCUSED_TEST_OUTPUT.txt
WPF_BUILD_OUTPUT.txt
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Create and push:

```text
report/report126.md
```

The report must state exact physical outcomes and must not expose passwords, tokens, full connection strings, or private identity values.
