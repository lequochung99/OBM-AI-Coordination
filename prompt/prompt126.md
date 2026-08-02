# Prompt126 — Implement canonical V004 simple two-phase installation and perform a fresh self-provisioned OBM-POS install

## Starting checkpoint

Latest accepted report:

```text
report/report125.md
coordination commit: 2ceb9fbfae8152119dcfd658700417f3178a381d
verdict: BLOCKED_ACTIVE_V0_DB_NAME_FALLBACK_REMOVAL
private artifact SHA-256: 372338b64a6ca458b6cfb3ca36d9786a70ddcff0a4f8e024b65b558add6dedcf
```

Prompt125 added `LocalPosDatabaseName` through PlatformApp -> pairing authorization -> redeem -> WPF checkpoint, but that ownership is now non-canonical.

The operator has established a new binding architecture document:

```text
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL_V004.md
expected SHA-256:
54d38eb1dd0b1fe53564d4550138f74d8b3099e1d85d521a2821f20d808450cb
```

V004 supersedes V003 and all prior installation/startup prompt interpretations.

If the exact local V004 file is absent or its SHA does not match, stop with:

```text
BLOCKED_CANONICAL_V004_DOCUMENT_MISSING_OR_MISMATCHED
```

Do not infer or substitute an older document.

## Authoritative architecture

### Exactly two independent phases

```text
Phase 1 — local WPF installation
Phase 2 — cloud pairing after MainWindow
```

### Phase 1

WPF alone owns:

```text
PostgreSQL host, default 127.0.0.1
PostgreSQL port, default 5432
PostgreSQL username
protected PostgreSQL password
operator-selected local database name
server connection test
database creation
EF/Npgsql migrations
minimal baseline seed
runtime-profile state
MainWindow startup routing
```

Phase 1 must not require:

```text
PlatformApp
Pairing Code
WpfJwt
API availability
SignalR
CompanionApp
BookingConsole
```

### Phase 2

Only after local `ApplicationReady` and MainWindow are available:

```text
operator enters Pairing Code
WPF redeems it
WPF stores the API credential in protected local storage
cloud/API/sync/SignalR become Online when authorized
```

Missing token, expired token, HTTP 401, absent refresh-token support, or API outage must never reopen local installation.

## Strict scope

Execute this task as one controlled refactor and physical fresh-install proof:

```text
1. Verify and adopt canonical V004.
2. Remove prompt125 DB-name ownership from PlatformApp/pairing/redeem.
3. Restore local DB configuration ownership to WPF local setup.
4. Ensure the WPF migration chain creates TblPosRuntimeProfile and TblPosRuntimeStateHistory.
5. Implement the two runtime-state transactions.
6. Make startup route only from protected local DB configuration + TblPosRuntimeProfile.
7. Remove active production fallback references to obm_pos_dev_v0_pg.
8. Add destructive-operation guards.
9. Perform a real fresh WPF self-provisioned installation to a previously absent DB.
10. Prove MainWindow startup while API is offline.
11. Only after local proof, exercise Phase 2 pairing when an authorized Pairing Code can be issued safely.
```

Do not resume Category Weight, Booking Weight, TblTenantPosDevice routing, POS2 sync, terminal/Companion modeling, refresh-token architecture, or broad Firebase/env cleanup.

## Phase 0 — Evidence and source intake

Read completely:

```text
INSTALLATION_RUNTIME_CANONICAL_V004.md
report/report122.md
report/report124.md
report/report125.md
prompt/prompt125.md
```

Inspect the active source call chains only for:

```text
WPF local PostgreSQL setup UI and configuration persistence
CleanLocalDatabaseService.CreateCleanDatabaseAsync
canonical Npgsql DbContext and migration assembly
runtime-profile entities/repository
Phase2 baseline seed transaction
local application finalization
App.xaml.cs / LocalPosStartupService routing
PlatformApp prompt125 LocalPosDatabaseName diff
pairing authorization/redeem/checkpoint LocalPosDatabaseName diff
all active obm_pos_dev_v0_pg references
all destructive reset/drop/truncate paths reachable from startup/install/migrate
```

Record before editing:

```text
CANONICAL_DOCUMENT=V004
CANONICAL_SHA_VERIFIED=true
PHASE1_OWNER=WPF_LOCAL_ONLY
PHASE2_OWNER=CLOUD_PAIRING_AFTER_MAINWINDOW
PLATFORMAPP_LOCAL_DB_OWNERSHIP=FORBIDDEN
TARGET_FRESH_DB=obm_pos_dev_v1_pg
TARGET_DB_MUST_BE_ABSENT_BEFORE=true
MANUAL_DB_PRECREATION=FORBIDDEN
MANUAL_SQL_RUNTIME_PROFILE_INSERT=FORBIDDEN
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
MANUAL_POS1_TEST_READY=false
```

## Phase 1 — Remove the non-canonical prompt125 handoff

Prompt125 introduced `LocalPosDatabaseName` through PlatformApp and pairing. Under V004, ordinary local DB configuration belongs only to WPF.

Remove `LocalPosDatabaseName` from active production ownership in:

```text
PlatformApp UI
PlatformApp client request
pairing-code request
pairing authorization state
installation attempt state
redeem response
bootstrap identity response
WPF pairing/redeem checkpoint
normal Phase2 target-DB mapping
```

Requirements:

```text
PlatformApp still supports administrator authorization, Tenant/POS selection, and Pairing Code issuance
Pairing Code behavior remains cloud authorization only
no local host/port/user/password/database name crosses the pairing contract
no token/header/WpfJwt weakening
no new endpoint
```

Compatibility code may remain only when direct source evidence proves a required backward-read boundary. It must not be active for new V004 installation and must be clearly marked compatibility-only.

Focused tests must prove:

```text
PlatformApp pairing request has no local DB configuration
redeem response has no local DB configuration
WPF pairing checkpoint has no local DB configuration
```

## Phase 2 — Canonical WPF local setup

Reuse the existing InstallationV0/local database setup UI and service boundaries. Do not create a second installer.

The UI must collect or resolve:

```text
Host: default 127.0.0.1
Port: default 5432
Username
Password
Database name
```

For the physical Development proof use:

```text
Database name: obm_pos_dev_v1_pg
```

The DB name is operator-entered in WPF. It must not be hardcoded as the active production value.

Persist the resulting local DB configuration using the existing protected local configuration boundary.

Security requirements:

```text
password protected locally
no password/full connection string in logs, reports, screenshots, source, or artifact text
no user-secrets dependency for WPF runtime
no PlatformApp dependency
```

Validation must reject malformed names, connection-string fragments, SQL-like input, path separators, quotes, semicolons, and protected DB names.

## Phase 3 — Canonical runtime-profile schema

The fresh attached WPF EF/Npgsql migration chain must create:

```text
TblPosRuntimeProfile
TblPosRuntimeStateHistory
```

Use the existing entity/repository design where valid. Attach it to the one canonical WPF DbContext/model and create the minimum required migration/snapshot change.

Do not create:

```text
TblSchemaVersion
TblSystemBaselineVersion
Phase2TrialCompletionMarker
another readiness framework
another DbContext
manual SQL table creation
```

Before physical install, prove from migration metadata that a zero-state DB will receive both tables.

## Phase 4 — Two short transactions

### Transaction A — DatabaseReady

After database creation and migrations are current:

```text
BEGIN
  seed the approved minimal baseline
  upsert exactly one current runtime-profile row = DatabaseReady
  append one DatabaseReady history transition
COMMIT
```

Requirements:

```text
baseline and DatabaseReady commit or roll back together
zero TblLocalOutbox rows from installation seed
no employee/service/customer/Invoice/Booking/runtime-history business seed
idempotent retry
```

### Transaction B — ApplicationReady

After local application finalization succeeds:

```text
BEGIN
  upsert the same current profile row = ApplicationReady
  append one ApplicationReady history transition
COMMIT
```

If existing source uses a proven equivalent such as `Activated`, map it to V004 `ApplicationReady` semantics rather than creating a duplicate meaning.

Resume requirements:

```text
no profile row -> resume local DB installation
DatabaseReady -> resume only application finalization; do not reseed
ApplicationReady -> MainWindow directly
```

History must never gate startup.

## Phase 5 — Startup routing

The normal startup order must be:

```text
resolve protected local DB configuration
connect to configured local DB
verify essential schema
read the single current TblPosRuntimeProfile row
route locally
open MainWindow when ApplicationReady
initialize API session afterward
```

Forbidden pre-MainWindow gates:

```text
Pairing Code
WpfJwt
ProtectedHello
API readiness
SignalR
refresh token
PlatformApp state
CompanionApp state
BookingConsole state
history-row counts
installation evidence counts
```

API/token failure after `ApplicationReady` must only set cloud state to Offline/Degraded/Reauthorization Required.

## Phase 6 — Remove active legacy DB-name fallback

Audit every active `obm_pos_dev_v0_pg` reference.

Classify each as:

```text
production fallback
Development test fixture
historical artifact/documentation
protected-name guard
```

Required result:

```text
ACTIVE_PRODUCTION_DB_NAME_CONSTANT_FALLBACK_COUNT=0
```

Do not delete valid tests merely because they use a disposable fixture name. Rename/isolate them when needed.

The current damaged/empty `obm_pos_dev_v0_pg` must not be used as the target of this installation and must not be copied into v1.

## Phase 7 — Destructive-operation guard

Normal startup, install resume, migration, API failure, token failure, and local recovery assessment must have zero reachable calls to:

```text
EnsureDeleted
DROP DATABASE
DROP SCHEMA
TRUNCATE
automatic reset/recreate
automatic reseed of an existing business DB
```

`CleanLocalDatabaseService` may create a missing validated DB. It must not silently drop an existing DB.

Add focused tests proving:

```text
Migrate preserves sentinel rows
normal startup cannot call a destructive reset path
existing DB with user/business data and missing profile routes to Recovery UI
fresh absent DB routes to Local Database Setup
```

## Phase 8 — Physical fresh local installation

Use the actual WPF Development executable/profile and visible label:

```text
prompt126
```

Before launch prove safely:

```text
obm_pos_dev_v1_pg exists = false
API listener 127.0.0.1:7161 = absent/offline
PlatformApp not required
Pairing Code not required
```

Do not create the DB manually with pgAdmin, psql, script, test helper, or external provisioning command.

Through the real WPF UI/service flow:

```text
1. Enter local host/port/username/password/database name.
2. Test PostgreSQL server access.
3. WPF creates obm_pos_dev_v1_pg.
4. WPF applies all attached migrations.
5. Pending migrations becomes 0.
6. TblPosRuntimeProfile physically exists.
7. TblPosRuntimeStateHistory physically exists.
8. Transaction A commits baseline + DatabaseReady + history.
9. Transaction B commits ApplicationReady + history.
10. MainWindow opens.
```

Required safe post-state:

```text
current runtime-profile row count = 1
current state = ApplicationReady or proven canonical equivalent
DatabaseReady history transition count = 1
ApplicationReady history transition count = 1
installation-created TblLocalOutbox row count = 0
```

## Phase 9 — Offline restart proof

Keep API port 7161 offline.

Prove:

```text
first MainWindow observation >= 60 seconds
close normally
second launch opens MainWindow directly
InstallationV0 does not flash
close normally
third launch opens MainWindow directly
InstallationV0 does not flash
local DB read/local CRUD smoke succeeds
no Pairing Code/token exists requirement
```

## Phase 10 — Cloud pairing only after local PASS

Only after Phase 8 and Phase 9 pass may this task exercise Phase 2.

Start the existing full API and PlatformApp lanes, issue an authorized Pairing Code for the selected Tenant/POS, then redeem it from the existing WPF cloud-connection boundary.

Prove:

```text
local runtime remains ApplicationReady before/during/after pairing
credential stored protected locally
API session becomes Online when accepted
HTTP 401/offline simulation changes only cloud status
MainWindow remains open
```

Do not add refresh-token architecture.

If no operator-authorized Pairing Code can be issued in this run, report cloud pairing as a narrow blocker after preserving local installation PASS. Do not downgrade or undo `ApplicationReady`.

## Build and tests

Run focused tests for:

```text
V004 phase ownership
prompt125 DB-name handoff removal
local DB config validation and protected persistence
fresh DB creation request uses operator-entered name
runtime-profile migration mapping
Transaction A atomicity and retry
Transaction B atomicity and retry
startup routing for no row / DatabaseReady / ApplicationReady
history is not a startup dependency
API 401/offline does not reopen installation
destructive path guards
active v0 fallback count = 0
```

Run WPF, PlatformApp, and API builds for changed projects.

Physical proof overrides build/test success.

## PASS and blocker verdicts

Full PASS verdict:

```text
OBM_WPF_V004_FRESH_LOCAL_INSTALL_MAINWINDOW_OFFLINE_AND_POST_INSTALL_PAIRING_PHYSICALLY_PROVEN
```

Local installation PASS but cloud pairing not completed:

```text
OBM_WPF_V004_LOCAL_INSTALL_MAINWINDOW_OFFLINE_PROVEN_BLOCKED_POST_INSTALL_PAIRING
```

Narrow blockers only:

```text
BLOCKED_CANONICAL_V004_DOCUMENT_MISSING_OR_MISMATCHED
BLOCKED_WPF_LOCAL_DB_CONFIGURATION
BLOCKED_WPF_DATABASE_SELF_PROVISIONING
BLOCKED_WPF_RUNTIME_PROFILE_MIGRATION
BLOCKED_WPF_DATABASE_READY_TRANSACTION
BLOCKED_WPF_APPLICATION_READY_TRANSACTION
BLOCKED_WPF_MAINWINDOW_ROUTING
BLOCKED_WPF_MAINWINDOW_PHYSICAL_PROOF
BLOCKED_POST_INSTALL_PAIRING
```

Do not return another generic installation blocker.

## Status locks

Until the physical local MainWindow proof passes:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=false
MANUAL_POS1_TEST_READY=false
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
```

After local proof:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=true
MANUAL_POS1_TEST_READY=false
```

## Required artifact and report

Preserve prior artifacts unchanged. Create:

```text
E:\Project2026\RecoveryReports\WpfCanonicalV004FreshInstallationV001
```

At minimum include:

```text
PRIVATE_HANDOFF.md
CANONICAL_V004_VERIFICATION.md
PROMPT125_HANDOFF_REMOVAL.md
LOCAL_DB_CONFIGURATION.md
RUNTIME_PROFILE_SCHEMA.md
MIGRATION_PROOF.md
TRANSACTION_A_PROOF.md
TRANSACTION_B_PROOF.md
STARTUP_ROUTING.md
DESTRUCTIVE_PATH_GUARD.md
FRESH_DB_ABSENCE_AND_CREATION.md
MAINWINDOW_60_SECOND_PROOF.md
SECOND_THIRD_LAUNCH_PROOF.md
POST_INSTALL_PAIRING_PROOF.md
FOCUSED_TEST_OUTPUT.txt
BUILD_OUTPUT.txt
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Create and push:

```text
report/report126.md
```

The public report must state exact physical facts and must not expose passwords, tokens, complete connection strings, or private identities.
