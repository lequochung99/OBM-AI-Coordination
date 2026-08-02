# Prompt123 — Rebuild clean WPF installation on obm_pos_dev_v1_pg, repair destructive migration behavior, and prove offline MainWindow startup

## Starting checkpoint

The operator has identified the real reason the WPF keeps returning to InstallationV0:

```text
obm_pos_dev_v0_pg still exists
but all business/runtime table data was cleared during recent migration/reset work
TblPosRuntimeProfile and TblPosRuntimeStateHistory are empty
therefore startup correctly sees no completed local installation state
```

This supersedes the assumption that a retained WpfJwt, refresh token, or API reachability is the primary startup blocker.

Prompt120/report120 and the prompt121/122 design work remain evidence only. Do not continue patching the empty v0 database as though it were a valid installed-local state.

## Authoritative decision

Create and use a new clean Development WPF database:

```text
obm_pos_dev_v1_pg
```

Preserve `obm_pos_dev_v0_pg` unchanged for forensic evidence. Do not drop, truncate, reseed, rename, or repurpose it.

The new v1 installation must be performed by the real WPF InstallationV0 flow from a clean state and must finish with:

```text
TblPosRuntimeProfile = one current ApplicationReady row
TblPosRuntimeStateHistory = valid DatabaseReady and ApplicationReady transitions
MainWindow opens directly on subsequent startup
API may be offline after installation completion
```

## Architecture contract

### Current-state table

`TblPosRuntimeProfile` is the single local startup source of truth.

Required startup routing:

```text
no current row
=> open InstallationV0 from the beginning

current state = DatabaseReady
=> resume only local application-finalization work

current state = ApplicationReady
=> open MainWindow directly
```

### History table

`TblPosRuntimeStateHistory` is audit/history only.

It must not be required to decide whether MainWindow opens.

### Remote state

After `ApplicationReady`, these conditions are cloud-degraded states only:

```text
API offline
WpfJwt expired or rejected with 401
no refresh-token contract
SignalR unavailable
sync unavailable
```

They must not reopen InstallationV0, clear runtime profile, clear local activation, or block MainWindow.

## Strict scope

Execute only:

```text
1. Preserve and inspect v0 read-only to identify the destructive data-loss boundary.
2. Add a regression guard preventing ordinary migration/startup from clearing an existing database.
3. Create obm_pos_dev_v1_pg through the canonical Development provisioning boundary.
4. Point the canonical WPF Development installation lane to v1 without creating parallel runtime code paths.
5. Run the real installation from a clean ProductRoot/state.
6. Implement/repair the two-stage runtime-profile writer lifecycle.
7. Complete installation and physically prove MainWindow startup with API offline.
```

Do not implement or modify:

```text
Category Weight
Booking Weight
Price Weight save behavior
TblTenantPosDevice
API destination routing
CompanionApp or terminal modeling
sync E2E
refresh-token architecture
production/customer/reference databases
```

## Phase 1 — Prove the destructive boundary

Inspect current source, scripts, migration tooling, accepted artifacts, and recent diffs for every path capable of removing data:

```text
EnsureDeleted
DropDatabase
DROP DATABASE
DROP SCHEMA
TRUNCATE
bulk DELETE without tenant/key filter
clean/reset/recreate helpers
migration-history replacement
schema recreation before Migrate
manual reset scripts
```

Return exactly one primary classification:

```text
D1_EXPLICIT_DEV_RESET_DROPPED_AND_RECREATED_DB
D2_MIGRATION_OR_HELPER_DROPPED_SCHEMA_OR_TABLES
D3_TOOLING_POINTED_TO_WRONG_EMPTY_DATABASE
D4_STARTUP_OR_PHASE2_CODE_CLEARED_DATA
D5_OTHER_EXACTLY_PROVEN_DESTRUCTIVE_BOUNDARY
```

Required evidence:

```text
exact file/class/method/script
exact command or call chain
when it ran
why it was allowed
whether it can affect a non-empty DB again
```

Do not spend the whole task on forensics. Once the direct destructive boundary and guard location are proven, proceed.

## Phase 2 — Add non-destructive migration guards

Normal WPF startup, `Database.Migrate`, and InstallationV0 must never silently clear a non-empty database.

Add focused guards/tests proving:

```text
ordinary migrate preserves existing sentinel rows
ordinary startup does not call EnsureDeleted/drop/recreate
reset helpers require explicit disposable-development authorization
reset helpers reject production/customer/reference/protected DB names
reset helpers are not reachable from normal startup or installation completion
```

Do not remove legitimate EF migrations. Correct the destructive wrapper/call site.

## Phase 3 — Create the clean v1 Development database

Create exactly:

```text
obm_pos_dev_v1_pg
```

Requirements:

```text
PostgreSQL/Npgsql
UTF8
canonical Development runtime/provisioning role contract
no copied business data from v0
no manual table creation
no EnsureCreated
all schema comes from the attached WPF EF migration chain
pending migrations after apply = 0
```

Record v0 and v1 DB identities safely. Do not expose credentials or full connection strings.

## Phase 4 — Clean installation state

Use one clean, explicit Development installation state bound to v1.

Preserve the old ProductRoot/state as a versioned forensic backup. Do not delete it.

The clean installation may use a new versioned ProductRoot only when required to avoid old checkpoint/database binding contamination. Record the exact path and configuration source.

Because this is an intentionally new clean installation, a new Pairing Code/redeem is permitted only through the existing authorized PlatformAppV0 flow. Do not fabricate tokens, copy DPAPI files, or hand-write completion checkpoints.

## Phase 5 — Rebuild the installation writer lifecycle

### Transaction A — database baseline completion

The existing Phase2/local database installation service must execute one transaction containing:

```text
required migrations/schema current check
minimal baseline seed
upsert exactly one TblPosRuntimeProfile current row to DatabaseReady
append one TblPosRuntimeStateHistory transition to DatabaseReady
commit
```

If any baseline operation fails:

```text
baseline changes rollback
DatabaseReady profile rollback
DatabaseReady history rollback
```

Required baseline remains minimal:

```text
settings
parameters
printer defaults
required roles
only required local setup/runtime rows
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

Phase2 installation seed must create zero `TblLocalOutbox` rows.

### Transaction B — application finalization

After actual local application activation/finalization succeeds, execute a second short transaction:

```text
upsert the same TblPosRuntimeProfile current row to ApplicationReady
append one TblPosRuntimeStateHistory transition to ApplicationReady
commit
```

Do not hold one database transaction across UI/network/application installation work.

If the process stops after Transaction A but before Transaction B:

```text
next startup reads DatabaseReady
resumes only application finalization
must not reseed the database
must not require a new redeem when the authorized local state is still valid
```

### Idempotency/cardinality

Prove:

```text
exactly one current TblPosRuntimeProfile row for the local POS identity
re-running DatabaseReady writer does not duplicate the current row
re-running ApplicationReady writer does not duplicate the current row
history transition duplication is prevented or explicitly idempotent
```

Use the existing entity columns/status vocabulary when possible. Do not invent a second profile table or readiness framework.

## Phase 6 — Startup routing rewrite

The normal WPF startup decision must query `TblPosRuntimeProfile` before remote calls.

Required behavior:

```text
Missing row -> InstallationV0
DatabaseReady -> resume local finalization
ApplicationReady -> MainWindow directly
```

`TblPosRuntimeStateHistory` must have startup dependency count `0`.

The following must have MainWindow eligibility dependency count `0` after `ApplicationReady`:

```text
ProtectedHello success
API reachability
WpfJwt validity
refresh-token availability
SignalR state
sync state
TblSchemaVersion custom readiness row
TblSystemBaselineVersion custom readiness row
Phase2TrialCompletionMarker
```

EF migration history may still be used to protect schema compatibility, but it must not be confused with installation completion.

## Phase 7 — Physical clean installation proof

Use visible WPF label:

```text
prompt123
```

Run the real clean installation against `obm_pos_dev_v1_pg`.

Required physical sequence:

```text
clean start shows InstallationV0
complete authorized Phase1 when required
run Phase2 baseline installation
observe DatabaseReady committed
complete local application finalization
observe ApplicationReady committed
open MainWindow
```

Record sanitized row counts/state names only; do not expose identities or token values.

## Phase 8 — Offline local-first acceptance

After `ApplicationReady` is committed:

```text
stop full API
prove no listener on 127.0.0.1:7161
close WPF normally
launch the normal WPF Development profile again
```

PASS requires:

```text
resolved DB = obm_pos_dev_v1_pg
provider = Npgsql/PostgreSQL
pending migrations = 0
TblPosRuntimeProfile current row count = 1
current state = ApplicationReady
required history transitions present
MainWindow opens directly
InstallationV0 does not show or flash
MainWindow remains alive and responsive for at least 60 seconds
local DB read succeeds
cloud/sync status may be Offline/Degraded
close and launch a second time
second launch again opens MainWindow directly with API still offline
```

Build/test success cannot replace physical MainWindow proof.

## Required tests

At minimum add/run focused tests for:

```text
ordinary migration preserves seeded sentinel data
normal startup cannot reach destructive reset helper
empty profile routes to InstallationV0
DatabaseReady routes to finalization resume
ApplicationReady routes directly to MainWindow
API 401/offline after ApplicationReady does not reopen installation
Transaction A rollback does not leave DatabaseReady
Transaction B rollback does not leave ApplicationReady
profile upsert cardinality remains one
history transition idempotency
Phase2 seed creates zero TblLocalOutbox rows
restart after ApplicationReady remains MainWindow
```

Expected:

```text
all focused tests pass
0 skipped
WPF build errors = 0
```

## Hard prohibitions

Do not:

```text
mutate obm_pos_dev_v0_pg
copy rows from v0 into v1
manually INSERT/UPDATE runtime profile just to force PASS
use EnsureCreated
run destructive reset from normal startup
write completion markers outside the owning lifecycle writer
add five-table readiness framework
use TblParameter as the installation flag
add refresh-token architecture
resume Category/Booking/Price Weight work
```

## PASS verdict

PASS only when every physical acceptance item succeeds:

```text
OBM_WPF_V1_CLEAN_INSTALLATION_RUNTIME_PROFILE_LIFECYCLE_MAINWINDOW_OFFLINE_PHYSICALLY_PROVEN_READY_FOR_OPERATOR_SCREENSHOT
```

Any visible InstallationV0 on an `ApplicationReady` normal startup is BLOCKED.

Allowed narrow blockers:

```text
BLOCKED_WPF_DESTRUCTIVE_MIGRATION_BOUNDARY_UNRESOLVED
BLOCKED_WPF_V1_DATABASE_PROVISIONING
BLOCKED_WPF_PHASE1_CLEAN_INSTALLATION
BLOCKED_WPF_DATABASE_READY_TRANSACTION
BLOCKED_WPF_APPLICATION_READY_TRANSACTION
BLOCKED_WPF_RUNTIME_PROFILE_STARTUP_ROUTING
BLOCKED_WPF_MAINWINDOW_PHYSICAL_PROOF
```

Status on PASS:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=true
MANUAL_POS1_TEST_READY=false
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
```

## Required private artifact

Create a new versioned artifact without overwriting prior artifacts:

```text
E:\Project2026\RecoveryReports\WpfV1CleanInstallationRuntimeProfileV001
```

Include at minimum:

```text
PRIVATE_HANDOFF.md
V0_FORENSIC_READONLY_PROOF.md
DESTRUCTIVE_BOUNDARY.md
NON_DESTRUCTIVE_GUARD.md
V1_DATABASE_PROVISIONING.md
MIGRATION_PRESERVES_DATA_PROOF.md
CLEAN_PRODUCTROOT_STATE.md
PHASE1_PROOF.md
DATABASE_READY_TRANSACTION.md
APPLICATION_READY_TRANSACTION.md
RUNTIME_PROFILE_CARDINALITY.md
RUNTIME_HISTORY_PROOF.md
STARTUP_ROUTING.md
API_OFFLINE_MAINWINDOW_60_SECOND_PROOF.md
SECOND_LAUNCH_PROOF.md
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

Write and push:

```text
report/report123.md
```

The public report must include:

```text
Verdict
Destructive-boundary classification D1-D5
Exact destructive owner/call site
Non-destructive guard implemented yes/no
v0 preserved unchanged yes/no
v1 DB exact safe name
v1 migrations applied and pending count
Clean ProductRoot/state path
New redeem performed yes/no and why
Transaction A physical proof
Transaction B physical proof
TblPosRuntimeProfile current row count/state
TblPosRuntimeStateHistory transition counts
Phase2 outbox rows created count
MainWindow opens directly yes/no
InstallationV0 shown on ApplicationReady startup yes/no
API offline proof yes/no
60-second MainWindow proof yes/no
Second-launch proof yes/no
WPF build/test totals
Category/Booking/Price Weight unchanged
TblTenantPosDevice/API sync unchanged
Operator screenshot ready true/false
Manual POS1 test ready false
Private artifact path and aggregate SHA-256
```