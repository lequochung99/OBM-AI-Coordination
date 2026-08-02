# Prompt125 — Add the canonical Local POS database-name handoff from PlatformApp pairing to the WPF protected installation checkpoint

## Starting checkpoint

Read completely:

```text
prompt/prompt124.md
report/report124.md
```

Accepted coordination references:

```text
report124 commit:
ba40543515b0ecebdf7bb485c21c7456b5e0d9ae

report124 private artifact:
E:\Project2026\RecoveryReports\WpfSelfProvisionedV1InstallationV001

aggregate SHA-256:
57ef55a0ac26b70dab8c22b7ecc8c464609648b01c0f3459c587d2fe32b0de28
```

Report124 returned:

```text
BLOCKED_PLATFORMAPP_PAIRING_DB_NAME_HANDOFF_MISSING
```

The active chain currently has no field that allows PlatformApp to specify the Local POS PostgreSQL database name and carry it through pairing/redeem into the WPF installation plan.

Confirmed active boundaries:

```text
PlatformApp UI:
<PLATFORM_APP>/Pages/Home.razor

PlatformApp client:
PlatformAppV0ApiClient.CreatePairingCodeAsync

API controller:
PlatformAppV0Phase1Controller.CreatePairingCode

API pairing authorization state:
PlatformAppV0PairingAuthorization

WPF DB provisioning service:
CleanLocalDatabaseService.CreateCleanDatabaseAsync
request property already available:
TargetDatabaseName
```

The WPF currently still displays/uses the old development constant:

```text
Phase2TrialConstants.ApprovedDatabaseName = obm_pos_dev_v0_pg
```

This constant must not remain the active production source for a new installation.

## Authoritative product contract

PlatformApp specifies the target Local POS database name as part of the existing POS-station pairing authorization.

The canonical handoff is:

```text
PlatformApp operator selects Tenant/POS
-> enters Local POS database name
-> creates Pairing Code
-> API stores the database name with that pairing authorization/installation attempt
-> WPF redeems the Pairing Code
-> redeem response carries the same database name
-> WPF persists it in the protected Phase1 installation checkpoint/plan
-> restart reads the same persisted name
-> Phase2 later passes it to CleanLocalDatabaseService.TargetDatabaseName
```

The database name is not a password and may be displayed safely, but it is an installation identity field and must be validated and immutable for the lifetime of one pairing authorization/installation attempt.

Use one canonical contract property name across PlatformApp/API/redeem/WPF:

```text
LocalPosDatabaseName
```

At the existing Phase2 provisioning boundary, map it once to:

```text
TargetDatabaseName
```

Do not create a second alias, legacy fallback property, parallel pairing endpoint, or alternate checkpoint.

## Strict scope

Implement only the database-name handoff:

```text
1. PlatformApp UI input and validation.
2. PlatformApp API client request/response contract.
3. API pairing-code request and stored pairing authorization/installation attempt.
4. WPF redeem response contract.
5. WPF protected checkpoint/installation-plan persistence.
6. Restart/resume proof that the same value is retained.
7. Mapping readiness to CleanLocalDatabaseService.TargetDatabaseName without invoking DB creation in this task.
8. Focused tests and builds.
```

This task ends after physical handoff and restart persistence are proven.

Do not create, drop, migrate, seed, or otherwise mutate any PostgreSQL database in prompt125.

## Frozen work

Do not modify or implement:

```text
CleanLocalDatabaseService database-creation behavior
runtime profile/history entity or migration
TblPosRuntimeProfile lifecycle writer
TblPosRuntimeStateHistory lifecycle writer
DatabaseReady/ApplicationReady startup routing
WPF MainWindow gate
WpfJwt claims, signing, validation, lifetime, or authorization policy
refresh-token architecture
API database schema/migrations
WPF database schema/migrations
TblTenantPosDevice
sync/outbox/Provider/SignalR
Category Weight
Booking Weight
Price Weight
Firebase cleanup
.env cleanup
CompanionApp/payment-terminal modeling
```

Do not redeem a production/customer Pairing Code. Use only the canonical disposable Development pairing path.

## Phase 1 — Audit the exact active contract before editing

Inspect completely:

```text
PlatformApp Home.razor POS1-POS10 selection and Pairing Code UI
PlatformAppV0ApiClient pairing request/response DTOs
PlatformAppV0Phase1Controller.CreatePairingCode
PlatformAppV0Store and PlatformAppV0State persistence
PlatformAppV0PairingAuthorization
installation attempt state/model
pairing-code response model
WPF redeem controller response model
WPF Phase1InstallationService redeem handling
WPF protected checkpoint/state models and store
WPF restart/resume checkpoint reader
Phase2TrialConstants.ApprovedDatabaseName call sites
CleanLocalDatabaseService request and DB-name validator/guard
all existing PostgreSQL safe-name/protected-name validators
```

Record before editing:

```text
ACTIVE_DB_NAME_HANDOFF_FIELD_COUNT=0
ACTIVE_WPF_CONSTANT_SOURCE=Phase2TrialConstants.ApprovedDatabaseName
TARGET_DEVELOPMENT_VALUE=obm_pos_dev_v1_pg
DATABASE_MUTATION_ALLOWED=false
TOKEN_OR_WPFJWT_CHANGES_ALLOWED=false
MANUAL_POS1_TEST_READY=false
```

## Phase 2 — Establish one validation boundary

Reuse the strongest existing PostgreSQL database-name validator and protected-name guard already used by the WPF provisioning service. Do not create conflicting regex/validation rules in multiple projects.

The accepted Development proof value is:

```text
obm_pos_dev_v1_pg
```

Validation must reject at minimum:

```text
empty/whitespace
connection-string fragments
server/host/port syntax
quotes or semicolons
path separators
SQL keywords/commands embedded as input
names outside PostgreSQL identifier length rules
protected production/customer database names according to existing guardrails
```

PlatformApp must show a clear validation error before creating a Pairing Code.

The API must validate again and fail closed; client-side validation is not sufficient.

Do not accept or transmit:

```text
host
port
username
password
full connection string
maintenance DB credential
runtime DB credential
```

Only the safe database name crosses the pairing contract.

## Phase 3 — Extend the existing PlatformApp pairing boundary

Add a visible field to the existing Tenant/POS + Pairing Code flow:

```text
Label: Local POS Database Name
Value for physical Development proof: obm_pos_dev_v1_pg
```

Requirements:

```text
field is associated with the selected logical POS station
operator must confirm a valid value before Pairing Code creation
existing POS1-POS10 UI remains one canonical UI
no duplicate Pairing Code card/page/endpoint
```

Add `LocalPosDatabaseName` to the existing create-pairing-code request and carry it through:

```text
PlatformApp UI
-> PlatformAppV0ApiClient
-> PlatformAppV0Phase1Controller.CreatePairingCode
-> PlatformAppV0PairingAuthorization
-> installation-attempt/pairing durable state
```

Once the Pairing Code is issued, the database name for that authorization/attempt is immutable. A later UI edit requires a newly issued Pairing Code/authorization; it must not silently alter an already-issued code.

Do not expose secret material in responses or logs.

## Phase 4 — Extend redeem and WPF protected persistence

Add `LocalPosDatabaseName` to the successful WPF redeem response using the existing redeem endpoint.

Do not add a new endpoint.

WPF must:

```text
receive LocalPosDatabaseName
validate it again with the canonical validator
persist it atomically with the existing protected Phase1 checkpoint/installation plan
read it back immediately
retain it across process restart
make it available to the existing Phase2 request builder
map it once to CleanLocalDatabaseService.TargetDatabaseName
```

Do not call `CreateCleanDatabaseAsync` in prompt125.

Do not place the database name into WpfJwt claims unless the current contract absolutely requires it and direct evidence proves there is no safer existing redeem-response/authorization-state route. The preferred design is stored pairing authorization -> redeem response -> protected WPF checkpoint.

Do not store any DB password or connection string in the checkpoint.

## Phase 5 — Remove the active hardcoded fallback

Audit every call site of:

```text
Phase2TrialConstants.ApprovedDatabaseName
```

For the active production installation path:

```text
checkpoint LocalPosDatabaseName must be the only source
missing field must produce an explicit recoverable installation-contract error
no silent fallback to obm_pos_dev_v0_pg
```

A test-only constant may remain only if it is provably isolated to test code and cannot be selected by the normal Development/Production installation path.

Required count:

```text
ACTIVE_PRODUCTION_DB_NAME_CONSTANT_FALLBACK_COUNT=0
```

## Phase 6 — Physical handoff acceptance

Use the canonical Development PlatformApp/API/WPF pairing path.

Before creating the code, prove safely:

```text
PlatformApp selected logical POS station=<safe identifier>
LocalPosDatabaseName=obm_pos_dev_v1_pg
```

Then execute:

```text
1. Enter obm_pos_dev_v1_pg in PlatformApp.
2. Create one fresh Development Pairing Code.
3. Prove stored pairing authorization contains exactly the same safe database name.
4. Redeem that code once from the actual WPF InstallationV0 build.
5. Prove the redeem response contains the same database name.
6. Prove WPF protected checkpoint persists the same database name.
7. Close WPF.
8. Restart WPF using the same ProductRoot.
9. Prove checkpoint resume reads obm_pos_dev_v1_pg without another redeem.
10. Prove the Phase2 request builder resolves TargetDatabaseName=obm_pos_dev_v1_pg.
11. Stop before any DB creation/migration/seed call.
```

Use visible WPF label:

```text
prompt125
```

Do not print the Pairing Code, WpfJwt, token, password, connection string, or raw protected checkpoint contents in the report.

## Phase 7 — Tests and builds

Add/run focused tests for:

```text
valid LocalPosDatabaseName accepted
invalid/unsafe names rejected in PlatformApp and API
protected database names rejected
create-pairing-code stores the database name
issued pairing authorization is immutable
redeem response returns the stored name
WPF checkpoint persists and reloads the name
restart resumes without new redeem
missing field fails explicitly with no v0 fallback
Phase2 request maps LocalPosDatabaseName -> TargetDatabaseName
no database provisioning method invoked in prompt125 acceptance
WpfJwt contract unchanged
```

Run:

```text
PlatformApp build/tests
ApiServer focused build/tests
WPF InstallationV0 build/tests
```

Expected:

```text
0 build errors
0 skipped focused tests
```

## PASS gate

PASS requires all of the following:

```text
PlatformApp has one Local POS Database Name input in the existing pairing flow
LocalPosDatabaseName reaches the stored pairing authorization
redeem returns the exact stored value
WPF validates and persists it in the protected checkpoint
restart reads the same value without redeeming again
Phase2 request resolves TargetDatabaseName from the checkpoint
active hardcoded obm_pos_dev_v0_pg fallback count = 0
no database created/dropped/migrated/seeded
no WpfJwt/token/header behavior changed
```

PASS verdict:

```text
PLATFORMAPP_WPF_LOCAL_DB_NAME_PAIRING_HANDOFF_PHYSICALLY_PROVEN_READY_FOR_SELF_PROVISIONING
```

Narrow blockers only:

```text
BLOCKED_PLATFORMAPP_DB_NAME_INPUT_CONTRACT
BLOCKED_PAIRING_AUTHORIZATION_DB_NAME_PERSISTENCE
BLOCKED_REDEEM_DB_NAME_RESPONSE_CONTRACT
BLOCKED_WPF_DB_NAME_CHECKPOINT_PERSISTENCE
BLOCKED_WPF_DB_NAME_RESTART_RESUME
BLOCKED_ACTIVE_V0_DB_NAME_FALLBACK_REMOVAL
BLOCKED_PHYSICAL_DB_NAME_HANDOFF_PROOF
```

Do not claim MainWindow readiness or manual POS1 readiness in prompt125.

Required status:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=false
MANUAL_POS1_TEST_READY=false
```

## Required private artifact

Preserve all previous artifacts unchanged. Create a new versioned artifact:

```text
E:\Project2026\RecoveryReports\PlatformPairingLocalDbNameHandoffV001
```

Include at minimum:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
REPORT124_VERIFICATION.md
ACTIVE_CONTRACT_BEFORE.md
VALIDATION_CONTRACT.md
PLATFORMAPP_UI_PROOF.md
PAIRING_AUTHORIZATION_PERSISTENCE.md
REDEEM_RESPONSE_PROOF.md
WPF_CHECKPOINT_PERSISTENCE.md
WPF_RESTART_RESUME_PROOF.md
PHASE2_TARGET_DB_MAPPING.md
HARDCODED_FALLBACK_AUDIT.md
NO_DATABASE_MUTATION_PROOF.md
FOCUSED_TEST_OUTPUT.txt
BUILD_OUTPUT.txt
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
report/report125.md
```

The report must include:

```text
Verdict
Report124 artifact SHA verified yes/no
Canonical property name
PlatformApp input added yes/no
Validation owner/method
Create-pairing-code request carries field yes/no
Stored pairing authorization carries field yes/no
Issued authorization immutable proof yes/no
Redeem response carries field yes/no
WPF checkpoint persists field yes/no
Restart reads field without redeem yes/no
Resolved Phase2 TargetDatabaseName
Active v0 constant fallback count
Database mutation count
WpfJwt/token/header changes count
PlatformApp/API/WPF test totals
PlatformApp/API/WPF build totals
Operator MainWindow screenshot ready false
Manual POS1 test ready false
Private artifact path and aggregate SHA-256
Coordination commit SHA
```
