# Prompt133 — Resume the existing empty V1 database safely, apply the attached WPF schema, complete ApplicationReady, and open MainWindow offline

## Starting checkpoint

Read completely:

```text
docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL_V004.md
report/report131.md
report/report132.md
prompt/prompt131.md
prompt/prompt132.md
```

Verify the accepted private artifacts referenced by reports 131 and 132.

Accepted source status:

```text
- Active dependency on legacy SpacePos.Provisioning.Schema is removed.
- Schema installation uses the existing in-process WPF EF/Npgsql migration path.
- Provisioning credential and runtime credential are separated.
- Provisioning role is postgres and must remain provisioning-only.
- Runtime role is hung and must own normal migration/seed/runtime access.
- database-settings.json legacy schemaVersion drift is fixed through atomic compatibility upgrade.
- Builds pass and focused InstallationV0 tests are 60/60.
```

Direct operator physical evidence now proves:

```text
visible label = prompt132
configured target = obm_pos_dev_v1_pg
target database exists
operator inspected the database and found it empty: no application tables are visible
InstallationV0 remains visible
Install Local Database Baseline is not available for completing the empty existing database
API remains offline
```

This task must treat the current target as an **existing empty/incomplete Development database created by the earlier interrupted installation**, not as a completed database and not as permission to drop/recreate it.

## Authoritative classification rule

Classify the target database physically before mutation:

```text
E1_EXISTING_EMPTY_DATABASE
- target database exists
- no accepted WPF application tables
- no business/user data
- no completed migration history

E2_EXISTING_PARTIAL_INSTALLATION_DATABASE
- some attached migration objects exist
- no ApplicationReady state
- no business/user data requiring recovery review

E3_EXISTING_DATABASE_WITH_USER_OR_BUSINESS_DATA
- any real employee/service/customer/invoice/booking/transaction data exists
- automatic installation must stop and route to recovery review
```

Only E1 or E2 may be resumed automatically and idempotently. E3 must stop without mutation.

Do not rely only on pgAdmin tree rendering. Query PostgreSQL catalogs/read-only counts and record sanitized evidence.

## Strict scope

Complete only this existing call chain:

```text
InstallationV0Window
-> existing local settings and credentials
-> Phase2TargetSafetyGuard
-> existing database existence/classification check
-> existing in-process WPF EF/Npgsql migration path
-> existing baseline transaction
-> existing runtime-profile writer
-> existing local application finalization
-> existing MainWindow startup path
```

Do not create a new installer, migration service, startup coordinator, database-settings store, runtime-profile service, or pairing flow.

Frozen work:

```text
Pairing Code redeem
WpfJwt/refresh token
API/sync/SignalR architecture
Category Weight
Booking Weight
Price Weight
TblTenantPosDevice
CompanionApp/payment terminal
Firebase/.env cleanup
```

## Hard safety locks

For `obm_pos_dev_v1_pg`:

```text
DROP DATABASE = forbidden
DROP SCHEMA = forbidden
TRUNCATE = forbidden
EnsureDeleted = forbidden
manual SQL table creation = forbidden
manual completion/profile insertion = forbidden
copy from obm_pos_dev_v0_pg = forbidden
```

Do not edit database-settings.json manually. Do not expose either PostgreSQL password.

## Phase 1 — Prove exact physical target state

Using safe read-only inspection, record:

```text
DATABASE_EXISTS
DATABASE_OWNER
DATABASE_ENCODING
DATABASE_CONNECTION_LIMIT
PUBLIC_SCHEMA_OWNER
RUNTIME_ROLE_CAN_CONNECT
RUNTIME_ROLE_DATABASE_PRIVILEGES
RUNTIME_ROLE_SCHEMA_PRIVILEGES
USER_TABLE_COUNT
EF_MIGRATION_HISTORY_EXISTS
EF_MIGRATION_HISTORY_ROW_COUNT
APPLICATION_TABLE_COUNT
TBL_POS_RUNTIME_PROFILE_EXISTS
TBL_POS_RUNTIME_STATE_HISTORY_EXISTS
BUSINESS_DATA_PRESENT
```

Choose exactly E1, E2, or E3.

If E3, stop with:

```text
BLOCKED_WPF_V1_EXISTING_DATA_REQUIRES_RECOVERY_REVIEW
```

## Phase 2 — Correct existing-empty-database resume eligibility

The operator-observed state is conceptually:

```text
Local DB config resolved = true
PostgreSQL authentication succeeded = true
database exists = true
schema ready = false
runtime profile count = 0
ApplicationReady = false
```

For E1/E2 this state must enable the existing local-install action.

Correct only the smallest current predicate/call boundary so:

```text
existing empty or partial approved Development DB
+ valid protected local settings
+ valid runtime/provisioning credential availability
+ exact safety target accepted
+ no install operation running
=> Install/Resume Local Database Baseline enabled
```

The enablement must not depend on:

```text
API reachability
WpfJwt
ProtectedHello
Pairing Code
SignalR
sync
```

`Open OBM-POS` must remain disabled until ApplicationReady.

Add focused tests for E1 and E2 button eligibility and E3 recovery blocking.

## Phase 3 — Reconcile owner and privileges without using postgres at runtime

The earlier CREATE DATABASE may have left ownership with `postgres`.

Using the existing provisioning boundary only, prove and, when required, perform the minimal safe owner/grant reconciliation:

```text
Database owner = hung, or an existing canonical runtime owner proven by source
runtime role hung can CONNECT
runtime role hung can use/create in the required application schema during migration
postgres credential is cleared after provisioning work
```

Do not persist the postgres password or postgres as the runtime username.

After provisioning reconciliation, all migration, seed, runtime-profile, and normal app access must use runtime role `hung`.

## Phase 4 — Resume the attached WPF migrations in process

For E1:

```text
run the attached WPF EF/Npgsql migrations from zero
```

For E2:

```text
resume only pending attached migrations idempotently
```

Use the existing in-process migration path introduced/activated by prompt131.

Required proof:

```text
legacy SpacePos.Provisioning.Schema invocation count = 0
external schema executable count = 0
migration owner = existing canonical WPF migration path
applied migration identifiers recorded
pending migrations after = 0
```

The attached migration result must contain the runtime lifecycle schema required by V004:

```text
TblPosRuntimeProfile
TblPosRuntimeStateHistory
```

If the active attached migration/model chain still omits either required table, make only the smallest attached EF/Npgsql model/migration correction needed. Do not introduce the abandoned multi-readiness framework.

Do not create merely to satisfy old checks:

```text
TblSchemaVersion
TblSystemBaselineVersion
Phase2TrialCompletionMarker
```

## Phase 5 — Complete Transaction A and Transaction B

### Transaction A

In the existing baseline transaction:

```text
minimal accepted baseline seed
+ upsert one current runtime profile to DatabaseReady-equivalent
+ append one DatabaseReady history transition
-> one commit
```

Prove:

```text
injected failure before commit leaves no partial baseline/profile/history
successful replay is idempotent
installation TblLocalOutbox rows = 0
```

### Transaction B

After existing local application finalization succeeds:

```text
upsert the same current profile to ApplicationReady-equivalent
+ append one ApplicationReady history transition
-> one short commit
```

Prove:

```text
current profile row count = 1
ApplicationReady transition count = 1
replay creates no duplicate transition
```

Do not require API, Pairing Code, WpfJwt, SignalR, or sync.

## Phase 6 — Physical acceptance

Update visible label to:

```text
prompt133
```

Keep API `127.0.0.1:7161` offline.

From the actual WPF UI:

```text
1. Enter provisioning and runtime passwords locally without logging them.
2. Confirm obm_pos_dev_v1_pg is classified E1 or E2.
3. Confirm Install/Resume Local Database Baseline is enabled.
4. Click it once.
5. Resume the existing DB; do not recreate it.
6. Apply migrations and baseline/finalization.
7. Reach ApplicationReady.
8. Open the production MainWindow.
```

Required physical proof:

```text
DB identity/OID remains the same before and after resume
no drop/recreate
pending migrations = 0
current runtime profile count = 1
current state = ApplicationReady-equivalent
InstallationV0 closes
MainWindow remains alive and responsive for at least 60 seconds
```

Then close normally and launch twice:

```text
restart 1 -> MainWindow directly; no InstallationV0 flash
restart 2 -> MainWindow directly; no InstallationV0 flash
API remains offline
local DB remains intact
no duplicate baseline/history rows
```

Do not redeem a Pairing Code in this task.

## PASS gate

PASS requires all of:

```text
existing V1 database classified safely as E1/E2
same DB resumed without drop/recreate
runtime owner/privileges valid for hung
in-process attached migrations completed
pending migrations = 0
runtime profile/history tables exist
Transaction A completed atomically
Transaction B reached ApplicationReady
MainWindow physical 60-second proof passed
restart 1 and restart 2 opened MainWindow directly
API remained offline
```

PASS verdict:

```text
OBM_WPF_V1_EXISTING_EMPTY_DB_RESUMED_APPLICATIONREADY_MAINWINDOW_OFFLINE_PHYSICALLY_PROVEN
```

Narrow blockers only:

```text
BLOCKED_WPF_V1_EXISTING_DATA_REQUIRES_RECOVERY_REVIEW
BLOCKED_WPF_V1_EMPTY_DB_RESUME_ENABLEMENT
BLOCKED_WPF_V1_RUNTIME_OWNER_PRIVILEGES
BLOCKED_WPF_V1_ATTACHED_MIGRATION
BLOCKED_WPF_V1_BASELINE_TRANSACTION
BLOCKED_WPF_V1_APPLICATIONREADY
BLOCKED_WPF_V1_MAINWINDOW_PHYSICAL_PROOF
```

Do not report PASS while InstallationV0 remains visible instead of MainWindow.

## Required artifact/report

Create a new versioned private artifact:

```text
E:\Project2026\RecoveryReports\WpfV1ExistingEmptyDbResumeV001
```

Include:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
TARGET_CLASSIFICATION.md
DB_IDENTITY_BEFORE_AFTER.md
OWNER_GRANT_PROOF.md
RESUME_ENABLEMENT_BEFORE_AFTER.md
ATTACHED_MIGRATION_PROOF.md
TRANSACTION_A_PROOF.md
TRANSACTION_B_PROOF.md
MAINWINDOW_60_SECOND_PROOF.md
RESTART_1_PROOF.md
RESTART_2_PROOF.md
FOCUSED_TEST_OUTPUT.txt
BUILD_OUTPUT.txt
FINAL_STATE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Push:

```text
report/report133.md
```

The public report must include all PASS-gate fields, database mutation counts, source files changed, build/tests, private artifact path, and aggregate SHA-256.