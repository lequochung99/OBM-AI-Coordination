# Prompt130 — Fix only the InstallationV0 local-install button enablement predicate and complete the existing physical V1 installation

## Starting checkpoint

Read completely:

```text
report/report128.md
report/report129.md
prompt/prompt129.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL_V004.md
```

Accepted current evidence:

```text
- Existing InstallationV0Window now exposes Host, Port, Username, PasswordBox, and Local Database Name.
- Existing protected local DB settings store is wired.
- Existing Phase2TargetSafetyGuard accepts exactly obm_pos_dev_v1_pg in Development.
- Existing CleanLocalDatabaseService.CreateCleanDatabaseAsync is wired from the Install Local Database Baseline action.
- Focused InstallationV0 tests are 58/58 PASS.
- obm_pos_dev_v1_pg is absent.
- API 127.0.0.1:7161 is offline.
- No Pairing Code has been redeemed for this local-install proof.
```

The operator has physically launched the latest prompt129 WPF build and entered valid local PostgreSQL inputs. The UI reports:

```text
LocalDatabaseConfigResolved=True
LocalDatabaseAuthenticationSucceeded=True
SchemaReady=False
RuntimeProfileCount=0
LocalPosReady=False
API status=OfflineDeferred / WPF_BOOTSTRAP_API_OFFLINE_DEFERRED
```

However, the `Install Local Database Baseline` button remains disabled.

This is now the only source defect authorized in this task before continuing the existing physical installation.

## Canonical rule

The Install button is a local Phase 1 action. It must not depend on:

```text
API reachability
WpfJwt acceptance
ProtectedHello success
Pairing Code
SignalR
sync availability
```

The button must be enabled when all local conditions are true:

```text
local DB configuration fields are valid
PostgreSQL host/port/user/password authentication succeeds
exact target DB name passes Phase2TargetSafetyGuard
no local install operation is already running
local runtime is not already ApplicationReady
```

`SchemaReady=False`, target DB absent, `RuntimeProfileCount=0`, and `LocalPosReady=False` are reasons to install, not reasons to disable the Install button.

The `Open OBM-POS` button must remain disabled until the existing local lifecycle reaches ApplicationReady.

## Strict scope

Execute only:

```text
1. Find the exact IsEnabled/CanExecute predicate for Install Local Database Baseline.
2. Record every current predicate input and owning class/method.
3. Remove only remote/API/token/pairing dependencies and any inverted local readiness condition that incorrectly disables the button before installation.
4. Reuse the existing click handler, protected settings store, target guard, CleanLocalDatabaseService, migration, baseline, runtime-profile, startup, and MainWindow paths.
5. Physically continue the fresh local install using the operator-entered PostgreSQL password through the UI.
6. Prove DB creation, migration, local lifecycle, MainWindow, and restart behavior.
```

Do not create:

```text
new installer window
new DB service
new settings store
new startup coordinator
new pairing flow
new API/token contract
```

Do not modify:

```text
Category Weight
Booking Weight
Price Weight
TblTenantPosDevice
sync routing
CompanionApp/payment-terminal architecture
Firebase/.env cleanup
refresh-token architecture
```

Do not drop, reset, truncate, or copy any database.

## Phase 1 — Prove the exact enablement defect

Record:

```text
INSTALL_BUTTON_OWNER=<control/command/property>
ENABLEMENT_METHOD=<exact class/method>
CURRENT_INPUTS=<all booleans/state values>
REMOTE_DEPENDENCY_COUNT=<count>
INVERTED_LOCAL_READINESS_CHECKS=<exact checks>
```

Under the operator-observed state, prove why the current predicate returns false.

Correct the smallest possible production code path.

Required post-fix truth table:

```text
valid local config + PostgreSQL auth success + safe target + not busy + not ApplicationReady
=> Install enabled

API offline / WpfJwt rejected / no Pairing Code
=> Install still enabled

operation running
=> Install disabled

ApplicationReady
=> Install disabled; Open OBM-POS/MainWindow path available
```

## Phase 2 — Focused tests and build

Add or update only focused tests for:

```text
operator-observed prompt129 state enables Install
API offline does not disable Install
WpfJwt/ProtectedHello failure does not disable Install
SchemaReady=False enables local install when other local checks pass
unsafe target disables Install
invalid/missing password disables Install
operation-in-progress disables Install
ApplicationReady disables Install
Open OBM-POS remains disabled before ApplicationReady
```

Run the existing InstallationV0 focused tests and WPF build.

Expected:

```text
all focused tests pass
0 skipped
build errors=0
```

## Phase 3 — Physical fresh local installation

Use visible label:

```text
prompt130
```

Preconditions:

```text
obm_pos_dev_v1_pg absent
API port 7161 offline
no Pairing Code redeemed
```

Launch the actual WPF InstallationV0 UI. The operator enters the real PostgreSQL password locally; never print or capture it.

Prove:

```text
Install Local Database Baseline is enabled
operator clicks it once
WPF creates obm_pos_dev_v1_pg through CleanLocalDatabaseService
no external/manual DB creation occurs
attached migrations apply from zero
pending migrations=0
minimal baseline commits
TblPosRuntimeProfile/TblPosRuntimeStateHistory exist when required by canonical V004
DatabaseReady is recorded through the existing production lifecycle writer
ApplicationReady is recorded through the existing production lifecycle writer
installation creates 0 TblLocalOutbox rows
MainWindow opens directly after completion
InstallationV0 closes and does not reopen
MainWindow stays responsive for at least 60 seconds with API offline
```

Then close normally and launch twice more.

Both restarts must prove:

```text
MainWindow opens directly
InstallationV0 does not flash
local DB remains intact
no reseed
no duplicate lifecycle transitions
no DB recreate/reset
API may remain offline
```

Do not perform cloud pairing in this task unless all local physical acceptance above already passes. The task may stop after local MainWindow/restart proof.

## PASS gate

PASS requires every field below:

```text
Install button enabled under operator-observed local-ready-to-install state=yes
API/token/pairing dependency count in Install enablement=0
WPF-created obm_pos_dev_v1_pg=yes
pending migrations=0
ApplicationReady=yes
MainWindow 60-second proof=yes
restart 1 direct MainWindow=yes
restart 2 direct MainWindow=yes
InstallationV0 shown/flashed after ApplicationReady=no
manual DB creation/reset/drop count=0
```

PASS verdict:

```text
OBM_WPF_V004_INSTALL_ENABLEMENT_FIXED_V1_LOCAL_INSTALL_MAINWINDOW_OFFLINE_PHYSICALLY_PROVEN
```

Narrow blockers only:

```text
BLOCKED_WPF_INSTALL_BUTTON_ENABLEMENT
BLOCKED_WPF_V1_DATABASE_CREATION
BLOCKED_WPF_V1_MIGRATION_FROM_ZERO
BLOCKED_WPF_DATABASE_READY_TRANSACTION
BLOCKED_WPF_APPLICATION_READY_FINALIZATION
BLOCKED_WPF_MAINWINDOW_PHYSICAL_PROOF
```

Do not report PASS while the Install button is disabled or while the production MainWindow has not physically opened.

## Required artifact and report

Preserve previous artifacts. Create:

```text
E:\Project2026\RecoveryReports\WpfV004InstallEnablementAndPhysicalV1V001
```

Include at minimum:

```text
PRIVATE_HANDOFF.md
ENABLEMENT_BEFORE.md
ENABLEMENT_AFTER.md
ENABLEMENT_TRUTH_TABLE.md
FOCUSED_TEST_OUTPUT.txt
BUILD_OUTPUT.txt
V1_DATABASE_CREATION_PROOF.md
MIGRATION_PROOF.md
DATABASE_READY_PROOF.md
APPLICATION_READY_PROOF.md
MAINWINDOW_60_SECOND_PROOF.md
RESTART_1_PROOF.md
RESTART_2_PROOF.md
NO_DESTRUCTIVE_ACTION_PROOF.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Create and push:

```text
report/report130.md
```
