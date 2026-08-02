# Prompt129 — Complete the existing InstallationV0 local database UI wiring and physically finish the V004 local installation

## Starting checkpoint

Read completely:

```text
docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL_V004.md
prompt/prompt126.md
prompt/prompt126_EXISTING_IMPLEMENTATION_MINIMAL_COMPLETION_ADDENDUM.md
report/report126.md
prompt/prompt128.md
report/report128.md
```

Accepted report128 checkpoint:

```text
Coordination commit: 75a05c151b3baaa3d0822519f912e4cdab143685
Verdict: BLOCKED_WPF_V1_DATABASE_CREATION
Private artifact: WpfV004ApprovedV1PhysicalInstallationV001
Manifest SHA-256: D371416976C883939E86B6272205BF1C246EE1783E05B6D653BF7CCCA96C1F5C
```

Accepted proven facts:

```text
- V004 source/test alignment is complete at the minimal existing-implementation layer.
- Exact Development safety target obm_pos_dev_v1_pg is approved.
- Phase2TargetSafetyGuard accepts only obm_pos_dev_v1_pg for this Development proof.
- Wildcard approval count is zero.
- Global safety bypass count is zero.
- obm_pos_dev_v1_pg was absent at the start of prompt128.
- API 127.0.0.1:7161 was offline.
- No Pairing Code was redeemed.
- CleanLocalDatabaseService.CreateCleanDatabaseAsync already exists and can create an absent PostgreSQL database.
- The current InstallationV0Window does not expose local DB inputs and does not invoke the existing service.
```

## Authoritative operator direction

The installation implementation already exists. Do not rebuild InstallationV0 from scratch.

This task must only complete the missing UI-to-service wiring and then continue through the existing V004 installation pipeline until the production MainWindow opens.

## Strict scope

Execute only:

```text
1. Audit the current InstallationV0Window UI/code-behind/view-model and identify the existing Phase2/local-database action.
2. Reuse the existing local DB settings model, protected storage, validators, CleanLocalDatabaseService, migration owner, baseline seed service, runtime-profile writer, startup router, and MainWindow launch path.
3. Add or expose the minimum local PostgreSQL input controls in the existing InstallationV0Window.
4. Wire the existing Install Local Database action to the existing CleanLocalDatabaseService.CreateCleanDatabaseAsync call chain.
5. Physically create obm_pos_dev_v1_pg from an absent state through the actual WPF UI.
6. Continue through migrations, baseline seed, DatabaseReady, ApplicationReady, and MainWindow.
7. Prove restart behavior while API remains offline.
```

Do not perform post-install Pairing Code redeem in this task. First close local installation and MainWindow physical proof. Cloud pairing remains a separate later step.

## Frozen work

Do not implement or modify:

```text
Category Weight
Booking Weight
Price Weight save semantics
TblTenantPosDevice
API destination routing
POS2 pull/apply/ACK
CompanionApp/payment-terminal design
Firebase cleanup
.env cleanup
refresh-token architecture
new uploader/bot/endpoint
PlatformApp Tenant/POS/Pairing behavior
WpfJwt/header/signing behavior
```

Do not reset, drop, truncate, rename, copy, or reseed:

```text
obm_pos_dev_v0_pg
any production/reference/customer database
```

Do not pre-create `obm_pos_dev_v1_pg` using pgAdmin, psql, PowerShell, EF CLI, or an external helper. The actual WPF UI must initiate database creation.

## Existing implementation reuse lock

Before editing, identify and record:

```text
INSTALLATION_WINDOW=<exact XAML/window>
INSTALLATION_UI_OWNER=<exact class>
LOCAL_DB_REQUEST_MODEL=<exact existing type>
LOCAL_DB_SETTINGS_STORE=<exact existing protected store>
DB_NAME_VALIDATOR=<exact existing validator>
TARGET_SAFETY_GUARD=Phase2TargetSafetyGuard.Validate
DATABASE_CREATE_SERVICE=CleanLocalDatabaseService.CreateCleanDatabaseAsync
MIGRATION_OWNER=<exact existing class/method>
BASELINE_OWNER=<exact existing class/method>
RUNTIME_PROFILE_WRITER=<exact existing class/method>
STARTUP_ROUTER=<exact existing class/method>
MAINWINDOW_LAUNCH_OWNER=<exact existing class/method>
```

New production service count must remain zero unless a direct missing-boundary proof makes one unavoidable. Prefer method-level corrections and UI wiring.

Do not create:

```text
second installer window
second DB-settings store
second database creation service
second migration runner
second baseline seeder
second runtime-profile repository
second startup coordinator
```

## Phase 1 — Minimal local database input UI

When protected local database configuration is absent or the configured Development target DB does not exist, the existing InstallationV0 window must expose the minimum fields:

```text
PostgreSQL Host
PostgreSQL Port
PostgreSQL Username
PostgreSQL Password
Local Database Name
```

Requirements:

```text
- Host defaults to the existing safe loopback convention.
- Port defaults to the existing PostgreSQL local port convention.
- Password uses a masked PasswordBox or equivalent secure WPF control.
- Password must not be copied into normal TextBox binding, logs, exceptions, reports, screenshots, or diagnostics.
- Database name uses the existing canonical validator.
- For this approved Development proof, the entered target is exactly obm_pos_dev_v1_pg.
- Do not restore an active production fallback constant to obm_pos_dev_v0_pg.
- Do not hardcode obm_pos_dev_v1_pg as a normal production fallback. A Development UI default/profile value is allowed only when explicitly classified and guarded as Development-only.
```

Reuse the current InstallationV0 layout and style. Add only the minimum controls needed; do not redesign the page.

## Phase 2 — Minimal UI action wiring

Reuse the current Phase2/local database button when possible. The UI must provide a clear action such as the existing `Install Local Database Baseline` action.

The owning UI handler must:

```text
1. Validate host/port/user/password-present/database name.
2. Run the exact-target Development safety guard.
3. Persist the validated local DB configuration through the existing protected local settings store.
4. Build the existing CleanLocalDatabase request without printing the password or full connection string.
5. Await CleanLocalDatabaseService.CreateCleanDatabaseAsync.
6. Surface progress and recoverable errors in the existing InstallationV0 UI.
7. Disable duplicate concurrent submissions.
8. Keep the process alive on validation/network/PostgreSQL errors.
```

Do not use `async void` except the unavoidable WPF event handler boundary; all internal operations must return/await `Task` and own their exceptions.

Do not invoke API, Pairing Code, WpfJwt, SignalR, or sync in this local database action.

## Phase 3 — Physical target precondition

Before launching the physical install, prove safely:

```text
TARGET_DB=obm_pos_dev_v1_pg
TARGET_EXISTS_BEFORE=false
ENVIRONMENT=Development
API_7161_LISTENER=false
PAIRING_REDEEMED=false
```

If `obm_pos_dev_v1_pg` already exists, stop with:

```text
BLOCKED_WPF_V1_TARGET_ALREADY_EXISTS
```

Do not delete it to force the test.

## Phase 4 — WPF creates the database through the actual UI

Use visible WPF label:

```text
prompt129
```

Launch the operator-equivalent WPF Development target and use the real InstallationV0 UI.

Enter the approved local settings and invoke the UI action.

Prove the call chain:

```text
InstallationV0 UI handler
-> existing request/model
-> existing protected DB settings store
-> Phase2TargetSafetyGuard.Validate
-> CleanLocalDatabaseService.CreateCleanDatabaseAsync
-> PostgreSQL maintenance connection
-> CREATE DATABASE obm_pos_dev_v1_pg
```

Required proof:

```text
WPF_CREATED_TARGET_DB=true
EXTERNAL_DB_CREATE_COUNT=0
DROP_DATABASE_COUNT=0
DROP_SCHEMA_COUNT=0
TRUNCATE_COUNT=0
ENSURE_DELETED_COUNT=0
```

## Phase 5 — Continue through the existing installation pipeline

After WPF creates the DB, continue through the existing implementation:

```text
connect to obm_pos_dev_v1_pg
-> apply attached Npgsql/EF migrations from zero
-> pending migrations = 0
-> run minimal baseline transaction
-> write DatabaseReady through the production runtime-profile writer
-> complete local application finalization
-> write ApplicationReady through the production writer
-> open production MainWindow
```

Do not manually insert/update runtime-profile or history rows.

If the runtime-profile/history tables are still absent from the attached migration chain, correct only the existing canonical EF model/migration boundary and create the minimum attached migration required. Do not create a separate readiness framework.

Required state proof:

```text
TblPosRuntimeProfile exists
TblPosRuntimeStateHistory exists
current profile row count = 1
current state = ApplicationReady-equivalent
DatabaseReady transition count = 1
ApplicationReady transition count = 1
installation TblLocalOutbox rows = 0
```

Transaction A must remain:

```text
minimal baseline
+ DatabaseReady current profile
+ DatabaseReady history
= one commit
```

Transaction B must remain:

```text
local finalization completion
+ ApplicationReady current profile
+ ApplicationReady history
= one short commit
```

## Phase 6 — MainWindow physical acceptance

Keep API port 7161 offline and do not redeem a Pairing Code.

PASS requires:

```text
MainWindow opens directly after local installation completes
InstallationV0 closes and does not reopen
MainWindow remains responsive for at least 60 seconds
local database operations can initialize without API
cloud/API status may be Offline/Degraded
process does not crash or exit
```

Then close normally and launch twice more.

Each restart must prove:

```text
MainWindow opens directly
InstallationV0 does not flash
profile remains ApplicationReady
no database recreate/reset
no baseline reseed
no duplicate runtime-history transition
no Pairing Code/token requirement
```

## Focused tests

Add/run focused tests for:

```text
InstallationV0 exposes local DB inputs when protected config is missing
password is not stored/logged in plaintext
validation prevents empty/unsafe target submission
UI action invokes the existing CleanLocalDatabaseService exactly once
concurrent double-click is prevented
service exception remains recoverable and does not crash WPF
exact Development target guard accepts v1 and rejects v0/protected targets
normal production flow has no fixed v0 fallback
existing DB is not dropped or cleared
ApplicationReady opens MainWindow while API is offline
restart does not reseed/recreate
```

Run relevant builds and tests. Physical proof overrides build/test success.

## PASS gate

PASS verdict:

```text
OBM_WPF_V004_EXISTING_INSTALLATION_UI_WIRED_V1_DB_CREATED_MAINWINDOW_OFFLINE_PHYSICALLY_PROVEN
```

PASS is forbidden unless all are true:

```text
obm_pos_dev_v1_pg absent before
WPF UI initiated database creation
obm_pos_dev_v1_pg exists after
pending migrations = 0
runtime profile = ApplicationReady-equivalent
MainWindow opened with API offline and no Pairing Code
60-second MainWindow proof passed
two restart proofs passed
no destructive database action occurred
```

Narrow blockers only:

```text
BLOCKED_WPF_LOCAL_DB_INPUT_UI
BLOCKED_WPF_LOCAL_DB_SETTINGS_PERSISTENCE
BLOCKED_WPF_V1_DATABASE_CREATION
BLOCKED_WPF_ATTACHED_MIGRATION
BLOCKED_WPF_DATABASE_READY_TRANSACTION
BLOCKED_WPF_APPLICATION_READY_TRANSACTION
BLOCKED_WPF_MAINWINDOW_PHYSICAL_PROOF
BLOCKED_WPF_RESTART_PROOF
```

Do not report PASS while InstallationV0 remains visible instead of the production MainWindow.

## Artifact and report

Preserve prior artifacts. Create:

```text
E:\Project2026\RecoveryReports\WpfV004InstallationUiWiringV001
```

Include at minimum:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
REPORT128_VERIFICATION.md
EXISTING_UI_CALL_CHAIN.md
MINIMAL_UI_DIFF.md
PROTECTED_DB_SETTINGS_PROOF.md
TARGET_ABSENCE_PROOF.md
WPF_DATABASE_CREATE_PROOF.md
MIGRATION_FROM_ZERO_PROOF.md
DATABASE_READY_TRANSACTION.md
APPLICATION_READY_TRANSACTION.md
MAINWINDOW_60_SECOND_PROOF.md
RESTART_ONE_PROOF.md
RESTART_TWO_PROOF.md
DESTRUCTIVE_PATH_NEGATIVE_PROOF.md
FOCUSED_TEST_OUTPUT.txt
BUILD_OUTPUT.txt
UNIFIED_DIFF.patch
FINAL_STATE.md
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Create and push:

```text
report/report129.md
```

The public report must include:

```text
Verdict
Canonical V004 SHA verified
Existing UI/service owners reused
New production service count
Local DB input controls added/exposed
Password plaintext/log count
UI-to-CleanLocalDatabaseService call proof
Target absent before
WPF created target DB
External DB creation count
Migration count and pending migrations
Runtime profile/history state
Installation outbox count
API offline proof
Pairing redeemed count
MainWindow direct-open proof
InstallationV0 flash/reopen proof
60-second proof
Restart one/two proof
Destructive action counts
Build/test totals
Operator screenshot ready true/false
Manual POS1 test ready false
Private artifact aggregate SHA-256
```
