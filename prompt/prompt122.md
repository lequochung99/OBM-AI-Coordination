# Prompt122 — Rebuild the WPF local installation/runtime lifecycle around TblPosRuntimeProfile and TblPosRuntimeStateHistory

## Status and authority

This prompt supersedes the implementation direction in prompt121 and all prompt121 addenda. Use prompt121/report120 only as evidence intake. Do not continue the five-readiness-table design.

Current accepted facts:

```text
The canonical WPF Development database is obm_pos_dev_v0_pg.
TblPosRuntimeProfile physically exists but is empty.
TblPosRuntimeStateHistory physically exists but is empty.
InstallationV0/Phase2 does not currently write the local runtime lifecycle state.
Normal WPF startup therefore returns to InstallationV0 and does not open MainWindow.
The retained WpfJwt/checkpoint still exists and decrypts.
API offline or WpfJwt 401 must not block a locally installed OBM-POS.
MainWindow has not yet been physically restored.
```

Report references:

```text
report/report118.md
report/report119.md
report/report120.md
report120 private artifact aggregate SHA-256:
894d8a903fc0fbb68051975fbee289166aca13d523d02115b7282c43c9809907
```

## Objective

Rebuild the complete local installation-state lifecycle so that:

```text
TblPosRuntimeProfile
= the single current-state source of truth used by WPF startup

TblPosRuntimeStateHistory
= append-only lifecycle audit history only
```

Implement and physically prove all three operator-required outcomes:

```text
1. Recover the current Development database by using production code to record that database installation and local application activation are complete.
2. Rewrite the startup routing so it reads TblPosRuntimeProfile first and deterministically chooses InstallationV0, local finalization resume, or MainWindow.
3. Rewrite the installation/Phase2 completion path so future installations write the runtime profile and history at the correct transactional boundaries.
```

Do not solve this by manual SQL, hand-edited markers, token bypass, or additional readiness frameworks.

## Authoritative lifecycle model

Use two local completion stages:

```text
DatabaseReady
ApplicationReady
```

Reuse existing enum/status names if the source already has semantically equivalent values. Do not introduce duplicate status names. If the existing names differ, document the exact mapping.

### Meaning

```text
No current profile row:
  Local installation has not completed its database phase.
  Route to InstallationV0 at the appropriate initial/local-database step.

DatabaseReady:
  Migrations and required baseline seed committed successfully.
  Local application finalization/activation is incomplete or must be resumed.
  Route directly to the local finalization step; do not restart pairing or re-run completed seed.

ApplicationReady:
  Local database and local application activation are complete.
  Route directly to MainWindow.
```

Remote/cloud state is independent:

```text
API offline
WpfJwt expired or rejected with 401
refresh token unavailable
SignalR offline
sync unavailable
```

These may set cloud/sync status to Degraded or Offline, but must not change an ApplicationReady profile, clear local activation, reopen InstallationV0, or block MainWindow.

## Required architecture

### TblPosRuntimeProfile

This is the only startup current-state table.

Requirements:

```text
exactly one current row for the local POS installation identity
idempotent upsert semantics
current lifecycle status
safe local installation/station identity fields already defined by the existing entity/schema
timestamps/version fields already defined by the existing entity/schema
no token or secret values
```

Before changing schema, inspect and document the physical table, EF entity, mapping, keys, indexes, constraints, and existing status values.

Do not create a new profile table.

### TblPosRuntimeStateHistory

This is append-only audit/history.

Requirements:

```text
append one transition record when the lifecycle state actually changes
no duplicate history row for an idempotent replay
startup dependency count = 0
never query history to decide whether MainWindow may open
no token or secret values
```

Do not create a second history table.

## Transaction boundaries

Do not keep a single database transaction open across file-system, DPAPI, network, API, or UI operations.

### Transaction A — database installation completion

After migrations are current and within the existing Phase2 baseline transaction:

```text
BEGIN LOCAL DB TRANSACTION
  write all required baseline seed rows
  upsert TblPosRuntimeProfile to DatabaseReady
  append TblPosRuntimeStateHistory transition to DatabaseReady
COMMIT
```

The profile/history write must commit atomically with the baseline seed.

If any required baseline write fails:

```text
baseline seed rolls back
DatabaseReady profile write rolls back
DatabaseReady history write rolls back
```

Rules:

```text
LOCAL_SEED_NO_OUTBOX=true
TblLocalOutbox rows created by installation seed = 0
no employee/service/customer/invoice/booking/turn/history business seed
no speculative seed outside the accepted minimal baseline
```

### Local application finalization

After Transaction A commits, perform the existing local application finalization idempotently. This may include only already-defined local activation/checkpoint/station-finalization work. Do not require the remote API to remain online after the initial authorized installation handoff.

Every finalization step must be restart-safe.

### Transaction B — application activation completion

Only after local finalization has been verified:

```text
BEGIN LOCAL DB TRANSACTION
  upsert TblPosRuntimeProfile from DatabaseReady to ApplicationReady
  append TblPosRuntimeStateHistory transition to ApplicationReady
COMMIT
```

If the process exits after Transaction A but before Transaction B, the next startup must detect DatabaseReady and resume finalization instead of restarting the complete installation.

## Startup routing rewrite

The normal WPF startup path must evaluate local state before remote/bootstrap/cloud validation.

Required decision order:

```text
1. Resolve the canonical ProductRoot and canonical local PostgreSQL database.
2. Verify the local database can be opened safely.
3. Read the single current TblPosRuntimeProfile row.
4. Route from the local lifecycle status.
5. Only after MainWindow/local finalization routing is chosen, start remote credential/API/SignalR/sync checks as nonfatal background work.
```

Required routing:

```text
profile missing
  -> InstallationV0 database-installation path

profile = DatabaseReady
  -> resume local application finalization
  -> never redo completed baseline seed
  -> never require new pairing merely because WpfJwt is expired/401

profile = ApplicationReady
  -> open production MainWindow directly
  -> InstallationV0 must not flash or replace MainWindow
```

If the profile row is corrupt, duplicated, has an unknown status, or conflicts with the bound local identity, show a precise recoverable repair state. Do not open MainWindow blindly and do not silently create a replacement row.

Remove the following from MainWindow eligibility:

```text
TblSchemaVersion
TblSystemBaselineVersion
Phase2TrialCompletionMarker
TblPosRuntimeStateHistory
remote ProtectedHello success
remote API reachability
WpfJwt validity
SignalR state
sync state
```

EF migration history may still validate schema currency, but it is not the installation-completion marker.

## Current Development database recovery

The current database already contains the required installed application data but the two lifecycle tables are empty.

Recover it through production code, not direct SQL.

Implement a narrow, idempotent recovery method owned by the same lifecycle service, for example conceptually:

```text
RecoverExistingInstalledLocalStateAsync
```

Use names consistent with the existing codebase.

The recovery method must:

```text
verify canonical DB = obm_pos_dev_v0_pg in Development
verify migrations/current schema and required baseline facts from direct local evidence
verify the existing local station/installation identity source
refuse to mark ready if required local facts are missing
use the same production upsert/history methods used by normal installation
record DatabaseReady transition
perform/verify local application finalization
record ApplicationReady transition
be idempotent on replay
never expose or replace token values
never call manual SQL
never fabricate identity values
```

The method may be invoked through a focused Development repair entry point or the existing InstallationV0 resume path. Do not add a permanent hidden bypass button to production UI.

Before recovery prove:

```text
TblPosRuntimeProfile current row count = 0
TblPosRuntimeStateHistory relevant transition count = 0
```

After recovery prove:

```text
TblPosRuntimeProfile current row count = 1
current state = ApplicationReady or exact existing semantic equivalent
DatabaseReady history transition count = 1
ApplicationReady history transition count = 1
replaying recovery changes neither current-row count nor transition counts
```

## Required code organization

Use one lifecycle owner/service for all reads and writes. Do not scatter direct table access across App.xaml.cs, InstallationV0 UI code, and seed code.

The lifecycle owner must expose cohesive operations equivalent to:

```text
GetCurrentAsync
MarkDatabaseReadyAsync
MarkApplicationReadyAsync
RecoverExistingInstalledLocalStateAsync
```

Exact names may follow repository conventions.

Requirements:

```text
all writes use the caller's existing DbContext/transaction when part of Phase2
no nested independent SaveChanges that breaks atomicity
idempotent transition handling
concurrency-safe current-row upsert
history append only on real state change
clear typed result for Missing, DatabaseReady, ApplicationReady, Corrupt/RepairRequired
cancellation-token support
structured logging without secrets
```

The startup coordinator calls the lifecycle owner; it must not reproduce lifecycle SQL/predicates itself.

## Schema and migration rules

The physical tables exist. Therefore the default expectation is no migration.

A migration is allowed only if direct inspection proves an exact schema defect required for this lifecycle, such as:

```text
missing required status column
missing current-row uniqueness constraint
incorrect column type preventing existing entity mapping
missing required foreign/key constraint
```

If a migration is necessary:

```text
create exactly one attached Npgsql migration
update the canonical snapshot
apply exact-once to obm_pos_dev_v0_pg
pending migrations after = 0
no unrelated table/index changes
```

Do not create TblSchemaVersion, TblSystemBaselineVersion, or Phase2TrialCompletionMarker as part of this task.

## Security and local-first locks

Preserve:

```text
Pairing Code -> redeem -> WpfJwt for initial authorized installation handoff
protected DPAPI credential/checkpoint storage
canonical Provider owns normal API token/header behavior
Firebase email/password remains retired
```

Do not:

```text
add refresh-token infrastructure
redeem a new pairing code for this recovery
clear retained credentials on API failure
make MainWindow depend on remote authentication
put secrets in runtime profile/history
restore Firebase email/password
```

## Physical test matrix

Set visible WPF build label to:

```text
prompt122
```

### Case 1 — Missing profile

Using a safe isolated test state, prove:

```text
profile missing
-> routes to InstallationV0 database phase
-> does not open MainWindow
```

### Case 2 — DatabaseReady resume

Prove:

```text
profile = DatabaseReady
-> does not redo baseline seed
-> resumes only local application finalization
-> transitions exactly once to ApplicationReady
```

Include a controlled restart between Transaction A and Transaction B.

### Case 3 — ApplicationReady with API offline

Stop the API and verify port 7161 has no listener.

Prove:

```text
profile = ApplicationReady
-> MainWindow opens directly
-> InstallationV0 does not show or flash
-> local PostgreSQL operations remain usable
-> WPF stays alive and responsive for at least 60 seconds
```

### Case 4 — ApplicationReady with retained WpfJwt rejected

When safely reproducible without token disclosure or replacement, prove:

```text
ProtectedHello 401
-> cloud/auth status becomes degraded
-> profile remains ApplicationReady
-> MainWindow remains open
-> credential/checkpoint is not deleted
```

### Case 5 — second normal launch

Close normally and launch again with API still offline.

Prove:

```text
MainWindow opens directly again
InstallationV0 does not appear
current profile remains one ApplicationReady row
history transition counts remain unchanged
```

## Focused tests

Add/run focused tests for:

```text
missing profile -> InstallationV0
DatabaseReady -> finalization resume only
ApplicationReady -> MainWindow before remote checks
API offline does not downgrade ApplicationReady
WpfJwt 401 does not downgrade ApplicationReady
baseline + DatabaseReady + history atomic commit
baseline failure rolls back profile/history
ApplicationReady transition idempotency
history not used by startup
recovery writer idempotency
unknown/duplicate profile produces repair state
no installation outbox rows
no Firebase email/password path
```

Expected:

```text
all focused tests pass
0 skipped
WPF build errors = 0
```

Physical behavior overrides test/build success.

## Frozen work

Do not inspect, design, implement, or modify:

```text
Category Weight
Booking Weight
Price Weight save semantics
TblTenantPosDevice
TblPosLocal destination routing
CompanionApp/terminal modeling
API grouped sync happy path
API schema/role/grant work
PlatformAppV0 POS1-POS10 UI
Pairing Code UI/API contract
legacy/env cleanup beyond what is directly required to compile
```

Do not reset/drop/recreate WPF or API databases.

## PASS gate

PASS requires every item below:

```text
TblPosRuntimeProfile is the only current startup source of truth
TblPosRuntimeStateHistory is audit-only
current Development DB recovered through production lifecycle writer
exactly one ApplicationReady current profile exists
DatabaseReady and ApplicationReady history transitions exist exactly once
normal future Phase2 writes DatabaseReady atomically with baseline seed
local finalization writes ApplicationReady through the lifecycle writer
API port 7161 offline
normal WPF launch label prompt122
MainWindow opens directly
InstallationV0 does not show or flash
MainWindow remains responsive for at least 60 seconds
second launch again opens MainWindow directly
no new redeem
no token/checkpoint deletion
no manual SQL
no unrelated feature/schema work
```

PASS verdict:

```text
OBM_WPF_RUNTIME_PROFILE_LIFECYCLE_REBUILT_MAINWINDOW_OFFLINE_PHYSICALLY_RESTORED_READY_FOR_OPERATOR_SCREENSHOT
```

Any visible InstallationV0 on the ApplicationReady normal launch requires BLOCKED.

Allowed narrow blockers:

```text
BLOCKED_WPF_RUNTIME_PROFILE_PHYSICAL_SCHEMA
BLOCKED_WPF_RUNTIME_PROFILE_IDENTITY_CONTRACT
BLOCKED_WPF_PHASE2_TRANSACTION_OWNER
BLOCKED_WPF_APPLICATION_FINALIZATION_BOUNDARY
BLOCKED_WPF_RUNTIME_PROFILE_RECOVERY_VALIDATION
BLOCKED_WPF_MAINWINDOW_ROUTING
BLOCKED_WPF_MAINWINDOW_PHYSICAL_PROOF
```

Do not return a generic readiness, installation, migration, or token blocker.

## Required artifact

Preserve prior artifacts. Create:

```text
E:\Project2026\RecoveryReports\WpfRuntimeProfileLifecycleRebuildV001
```

Include at minimum:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
REPORT120_ARTIFACT_VERIFICATION.md
PHYSICAL_TABLE_SCHEMA.md
ENTITY_MAPPING.md
STATUS_MAPPING.md
LIFECYCLE_BEFORE.md
LIFECYCLE_AFTER.md
STARTUP_DECISION_BEFORE.md
STARTUP_DECISION_AFTER.md
TRANSACTION_A_PROOF.md
APPLICATION_FINALIZATION_PROOF.md
TRANSACTION_B_PROOF.md
CURRENT_DB_RECOVERY_PROOF.md
IDEMPOTENCY_PROOF.md
API_OFFLINE_MAINWINDOW_PROOF.md
WPFJWT_401_DEGRADED_PROOF.md
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

Create and push:

```text
report/report122.md
```

It must include:

```text
Verdict
Prompt120 artifact SHA verified yes/no
Visible label
Physical TblPosRuntimeProfile schema/key/status mapping
Physical TblPosRuntimeStateHistory schema/key mapping
Lifecycle root cause classification
Lifecycle owner service/class/methods
Current profile row count before/after
Current profile final state
History transition counts before/after/replay
Transaction A atomic proof
Transaction B proof
Current Development recovery invoked through production writer yes/no
Manual SQL used count
Migration created yes/no and identifier
Pending migrations after
Startup source-of-truth table count
History startup dependency count
API port 7161 offline proof
MainWindow opens directly yes/no
InstallationV0 shown/flashed yes/no
60-second MainWindow proof
Second-launch proof
WpfJwt 401 downgrades only cloud state yes/no
Credential/checkpoint preserved yes/no
Installation outbox rows created count
Focused test totals
WPF build totals
Category/Booking/Price Weight changed no
TblTenantPosDevice/API sync changed no
Operator MainWindow screenshot ready true/false
Manual POS1 test ready false
Private artifact path and aggregate SHA-256
Coordination commit SHA
```