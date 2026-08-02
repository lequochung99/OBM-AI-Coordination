# Prompt 114 — Reuse the proven protected PostgreSQL provisioning boundary, repair the canonical API role contract, and complete the physical POS1 → API happy path

## Starting checkpoint

Prompt113 returned:

```text
BLOCKED_MAIN_DEV_API_ROLE_CONTRACT
```

Coordination references:

```text
report/report113.md
report113 commit:
14d9164a80261d4dfafe80a760c40ca617780386

prompt113 private artifact aggregate SHA-256:
b7460bc0e6a6fda241537af758524ea17a9daf4d37942828bac92aca04fb11cd
```

Report113 proves:

```text
canonical API DB runtime proof = yes
ApiServer starts non-interactively on loopback = yes
actual startup does not require runtime SELECT on __EFMigrationsHistory
failed proof command = dotnet ef migrations list through DesignTimeExternalDbContextFactory
public.__EFMigrationsHistory owner = postgres
normal runtime role is not superuser and is not object owner
normal runtime role incorrectly has CREATEDB=true
no migration/provisioning role was resolved in prompt113
no grant/provisioning source-of-truth correction was applied
no identity/routing/domain/outbox/event/delivery/SignalR write occurred
```

This task must recover and reuse the already proven protected PostgreSQL administration/provisioning boundary from the accepted reset artifacts, repair the role contract at its source, apply it idempotently to `obm_api_dev_v0_pg`, and then resume the same physical happy path from prompts111–113.

## Authoritative user lock

Do not begin Service Category Weight or Booking Weight in this task or after a blocked result.

The user must not be told to start manual application testing until all of the following are physically proven:

```text
canonical API role contract repaired
pending migrations = 0 through the correct migration/provisioning boundary
runtime role least-privilege positive and negative proofs pass
one production Price Rule Save reaches the API through the existing periodic WPF worker
one atomic EventLog/Delivery commit succeeds
SignalR occurs only after commit
all local outbox rows are marked Sent together
focused tests and architecture guards pass with 0 skipped
```

Only a PASS verdict from this task may set:

```text
MANUAL_POS1_TEST_READY=true
```

Any blocker must report:

```text
MANUAL_POS1_TEST_READY=false
```

## Canonical architecture locks

### PostgreSQL roles

Preserve a clear separation of responsibility:

```text
maintenance/admin role
- destructive database maintenance and role/grant provisioning only
- protected, non-interactive, not used by normal ApiServer requests

migration/provisioning role or existing migration-owner boundary
- applies ExternalDbContext migrations and verifies migration history
- owns or is authorized to manage the application schema objects
- applies the canonical runtime grants

ApiServer runtime role
- normal application access only
- LOGIN when required
- NOSUPERUSER
- NOCREATEDB
- NOCREATEROLE
- NOREPLICATION
- not database owner
- not schema owner
- not migration/table owner
```

Do not solve this by granting the runtime role ownership, membership in the admin role, or blanket privileges.

Forbidden:

```text
SUPERUSER
GRANT admin-role TO runtime-role
GRANT ALL ON DATABASE
GRANT ALL ON SCHEMA
GRANT ALL ON ALL TABLES
ALTER DATABASE OWNER TO runtime-role
ALTER SCHEMA OWNER TO runtime-role
ALTER TABLE ... OWNER TO runtime-role
restoring CREATEDB after removal
password literals or complete connection strings in source/scripts/artifacts
```

### Authentication

The retired Firebase email/password path remains absent.

The WPF installation/bootstrap authorization flow remains:

```text
Pairing Code -> redeem -> WpfJwt -> bootstrap/me
```

Do not confuse PostgreSQL provisioning credentials with WpfJwt, Platform administrator authentication, or production sync authentication.

### Sync

There remains exactly one canonical sync flow:

```text
WPF domain Save + TblLocalOutbox in one PostgreSQL transaction
-> existing periodic WPF outbox worker
-> existing canonical FlushOutbox/API service path
-> existing standard API grouped ingest controller/service
-> one TblEventLog/TblEventDelivery transaction
-> one durable API commit
-> existing post-commit SignalR notification
```

Do not create another uploader, worker, endpoint, ingest service, event writer, delivery transport, ACK path, or SignalR publisher.

## Strict scope

Execute only:

```text
1. Read and verify the prompt113 private artifact.
2. Recover the exact protected non-interactive PostgreSQL admin/provisioning boundary already used successfully by prompts107 and 108.
3. Prove why prompt113 failed to discover or reuse that boundary.
4. Define and persist the canonical API role/grant contract in the existing provisioning/reset mechanism.
5. Apply the contract idempotently to only obm_api_dev_v0_pg.
6. Remove CREATEDB from the normal ApiServer runtime role.
7. Prove migration history and pending migrations = 0 through the correct migration/provisioning role or owner boundary, not by broadening runtime privileges unnecessarily.
8. Prove the runtime role has exactly the positive runtime permissions needed and lacks destructive/DDL/admin capabilities.
9. Resume and complete the physical POS1 -> API grouped-sync happy path.
10. Rerun focused tests, architecture guards, and WPF/API builds.
```

Do not execute:

```text
DB reset/drop/recreate
new migration generation unless a narrowly proven source defect requires it
15-case failure/recovery matrix
Service Category Weight
Booking Weight
POS2 pull/apply/ACK
checkout/payment changes
Queue changes
BookingConsole changes
cloud or production deployment
```

Do not mutate production/customer/reference databases.

## Required evidence intake

Read completely:

```text
prompt/prompt107.md
report/report107.md
prompt/prompt108.md
report/report108.md
prompt/prompt109.md
report/report109.md
prompt/prompt110.md
report/report110.md
prompt/prompt111.md
report/report111.md
prompt/prompt112.md
report/report112.md
prompt/prompt113.md
report/report113.md
prompt/prompt112_SQL_TEMPLATE_POLICY_ADDENDUM.md
```

Read and verify these versioned artifacts:

```text
E:\Project2026\RecoveryReports\MainWpfDevResetExecutionV002
aggregate SHA-256:
47f68c634a5984611f3cb8b39ba3999f6005a558ad1e0d64bf998f7f4c2a0c58

E:\Project2026\RecoveryReports\MainApiDevResetExecutionV001
aggregate SHA-256:
e9d8298486f31f40581cb4445fa0abac25030bd586303098c05e1a9225f0d0ea

E:\Project2026\RecoveryReports\MainDevApiStartupAndSyncHappyPathV001
aggregate SHA-256:
6cf0b248fe2888e5605b54c22bcb09d8a3d0bc3acf4e2efa913cf7cd83e3a7e6

prompt113 private artifact
aggregate SHA-256:
b7460bc0e6a6fda241537af758524ea17a9daf4d37942828bac92aca04fb11cd
```

Report107 publicly identified the protected credential source name:

```text
OBM_PLATFORM_V2_P6_POS_PG_ADMIN
```

Treat that only as a verified candidate name. Confirm from the private artifacts and current source whether it is the same approved local PostgreSQL admin/provisioning boundary applicable to the canonical API Development host. Do not assume or copy it blindly.

At minimum inspect:

```text
all files and commands used by prompt107 to open the protected maintenance connection
all files and commands used by prompt108 to reset/recreate/migrate obm_api_dev_v0_pg
protected environment import scripts and key names only
start-api-local.ps1 and prompt112 changes
ExternalDbContext design-time factory and migration executor
canonical API reset/provisioning/grant utility
role creation/alter/grant SQL or C# code
__EFMigrationsHistory ownership and grants
public schema ownership/default privileges
all API tables, sequences, functions, and required runtime operations
runtime connection role attributes and inherited memberships
```

Never print passwords, tokens, complete connection strings, passfile contents, private keys, or protected values.

Record before editing:

```text
PROMPT107_ARTIFACT_VERIFIED=true
PROMPT108_ARTIFACT_VERIFIED=true
PROMPT113_ARTIFACT_VERIFIED=true
CANONICAL_API_DB=obm_api_dev_v0_pg
RUNTIME_CREATEDB_BEFORE=true
MANUAL_POS1_TEST_READY=false
DB_RESET=FORBIDDEN
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
SYNC_ARCHITECTURE_CHANGE=FORBIDDEN
```

## Phase 1 — Recover the already proven provisioning boundary

Produce direct sanitized evidence for:

```text
PROMPT107_ADMIN_BOUNDARY=<exact script/class/method>
PROMPT108_API_RESET_BOUNDARY=<exact script/class/method>
PROTECTED_SOURCE_NAME=<name only>
PROTECTED_SOURCE_AVAILABLE_NOW=<yes/no>
TARGET_HOST_MATCH=<yes/no>
TARGET_PORT_MATCH=<yes/no>
NONINTERACTIVE_CONNECTION_PROOF=<yes/no>
WHY_PROMPT113_DID_NOT_REUSE_IT=<exact narrow reason>
```

The preferred result is reuse of the same protected local PostgreSQL administration mechanism that already reset and migrated the canonical Development databases.

Do not create a second secret store, new password prompt, new user-secret DB credential, or alternate admin framework.

If the previously proven protected source is no longer available, stop with:

```text
BLOCKED_MAIN_DEV_API_PROVISIONING_CREDENTIAL_UNAVAILABLE
```

Report the exact missing source name and setup boundary only. Do not ask the user to paste a password into chat, source, or report.

## Phase 2 — Define the exact role matrix from real runtime operations

Audit actual API call sites and SQL generated by EF for the happy path and normal runtime. Produce an explicit matrix:

```text
Object/category
Required operation
Runtime role permission
Migration/provisioning permission
Maintenance/admin permission
Evidence call site
```

At minimum classify:

```text
DATABASE obm_api_dev_v0_pg
SCHEMA public or actual application schema
__EFMigrationsHistory
TblEventLog
TblEventDelivery
TblEventDeliveryGroupAck
identity sequences used by those tables
identity/routing/subscription tables required by production auth and destination resolution
any functions explicitly called by normal runtime
```

Canonical principle:

```text
actual ApiServer startup/runtime does not need schema migration ownership
migration-current proof uses the migration/provisioning boundary
runtime receives only CONNECT, schema USAGE, required table DML/SELECT, required sequence privileges, and explicitly proven EXECUTE rights
```

Do not grant SELECT on `__EFMigrationsHistory` to runtime unless a complete actual runtime call chain proves it is required. Prompt113 already reported it was observed only in the proof command.

## Phase 3 — Correct the existing source-of-truth idempotently

Extend the existing canonical provisioning/reset mechanism used by prompt108. Do not create a parallel provisioning pipeline.

It must idempotently enforce at least:

```text
ALTER ROLE <runtime> NOSUPERUSER NOCREATEDB NOCREATEROLE NOREPLICATION;
REVOKE or absence of unintended role memberships;
CONNECT only to the intended Development database as required;
USAGE on the application schema;
explicit table privileges required by actual runtime;
explicit sequence privileges required by identity inserts;
explicit function EXECUTE only when proven;
no ownership transfer to runtime;
no blanket GRANT ALL;
```

Use generated/quoted identifiers safely through the existing implementation. Do not concatenate untrusted identifiers into raw SQL.

The migration/provisioning role or current schema owner must be the boundary that:

```text
reads __EFMigrationsHistory
lists applied/pending migrations
applies future migrations
applies/reapplies canonical grants after schema change
```

When future migrations create new runtime tables/sequences, the same existing provisioning boundary must be able to reapply the explicit grants. Document whether this is implemented through an idempotent grant manifest, a post-migration provisioning step, or an already established equivalent.

Do not finalize the two SQL template scripts yet. Record the proven role/grant contract for later export to:

```text
E:\Project2026\2SQL PostgreSQL
```

## Phase 4 — Apply and physically prove the API role contract

Using the recovered protected provisioning boundary, apply the contract only to `obm_api_dev_v0_pg`.

Required physical proofs:

```text
runtime role CREATEDB=false
runtime role CREATEROLE=false
runtime role SUPERUSER=false
runtime role is not database/schema/table owner
runtime role cannot CREATE DATABASE
runtime role cannot CREATE or ALTER application tables
runtime role cannot DROP application tables
runtime role cannot read or alter protected objects outside its contract
migration/provisioning boundary can read __EFMigrationsHistory
migration history contains the accepted chain exactly once
pending migrations = 0
```

Positive runtime proof must use the normal runtime role and a rolled-back or isolated Development-only transaction:

```text
connect to canonical API DB
read required identity/routing data
insert/update/read/delete or transaction-rollback probe for the exact runtime tables required by the grouped sync path
use required sequences successfully
no persistent probe residue
```

Do not manually insert happy-path EventLog/Delivery rows. Permission probes must be rolled back or separately tagged and cleaned deterministically before E2E.

If the runtime role lacks an operation genuinely required by the production call chain, add only that explicit privilege to the source-of-truth and rerun all positive/negative proofs.

## Phase 5 — Re-prove canonical runtime startup

Start the actual ApiServer through the corrected non-interactive `start-api-local.ps1` boundary.

Prove:

```text
LocalDevelopment profile
loopback-only binding
canonical DB = obm_api_dev_v0_pg
normal runtime role is used
readiness succeeds
no hidden prompt
no admin/provisioning credential is passed to the normal request process
no secret value logged
```

The runtime process must not inherit or retain the admin/provisioning credential after startup.

## Phase 6 — Resume the physical POS1 → API happy path

Resume all remaining prompt111–113 phases through the existing production boundaries.

### Development-only prerequisites

Create only the existing-contract prerequisites for:

```text
one generated Development tenant
one generated POS1 source identity
one generated POS2 destination identity/subscriber mapping
subscriptions/routing for exactly TblTurnPolicy and TblTurnAmountRule
minimal Price Rule Save prerequisites
```

Use the existing canonical setup/test-fixture boundary. Do not use production/customer identities.

Do not manually insert:

```text
TblLocalOutbox
TblEventLog
TblEventDelivery
Price Rule outbox payloads
```

### Production Price Rule Save

Use the production Price Rule Save boundary to create one true change.

Prove:

```text
one DbContext
one explicit local PostgreSQL transaction
domain changes + complete TblLocalOutbox group commit atomically
ExpectedEventCount equals actual count
contiguous SequenceNumber
TblTurnPolicy parent first when created
no partial local state
```

### Existing periodic worker

Use the registered production WPF periodic outbox worker and canonical FlushOutbox chain.

Prove:

```text
one worker cycle
one atomic group claim
one grouped HTTP request
zero parallel production path invocations
```

### API durable transaction

For the generated TransactionGuid, prove:

```text
production sync authentication and identity validation pass
one API transaction begins
TblEventLog count = ExpectedEventCount
TblEventDelivery destination count = ExpectedEventCount per valid destination
TblEventDelivery source count = 0
one durable commit
no partial durable state
```

### SignalR and local completion

Prove:

```text
API commit completes before SignalR publish attempt
one existing SignalR publisher path
notification-only metadata, no business payload
publish succeeds
all local group rows become Sent together
SentAt populated
claim/lease cleared
no mixed local status
```

Required marker:

```text
SYNC_GROUP_MAIN_DEV_HAPPY_PATH_COMMITTED
```

## Phase 7 — Focused tests, architecture guards, and builds

Run focused tests for:

```text
protected provisioning boundary recovery
idempotent role/grant application
runtime NOCREATEDB/NOCREATEROLE/NOSUPERUSER
runtime positive application permissions
runtime negative DDL/admin permissions
migration history and pending migrations through the correct role
non-interactive ApiServer startup with runtime-only credential
production worker -> canonical FlushOutbox chain
complete-group claim/validation
one grouped HTTP request
production sync auth/identity match
API atomic EventLog/Delivery commit
source exclusion
SignalR after commit
all-or-none local completion
```

Rerun prompt110 architecture guards.

Expected:

```text
all pass
0 skipped
parallel production paths remaining = 0
```

Build:

```text
WPF
ApiServer
```

Build/test success does not override failed physical role, startup, or happy-path proof.

## End state

PASS requires all of the following:

```text
canonical API runtime role is least-privilege and CREATEDB=false
protected provisioning boundary is documented and reusable non-interactively
migration history and pending migrations = 0 are proven through the correct boundary
ApiServer runs with runtime-only credential on canonical DB
one physical canonical POS1 -> API grouped-sync happy path is committed
SignalR is after commit
local outbox group is Sent atomically
no second sync path exists
Service Category Weight and Booking Weight remain untouched
MANUAL_POS1_TEST_READY=true
```

Do not tell the user to test before this PASS state.

## Required private artifact

Preserve all earlier artifacts unchanged. Create:

```text
E:\Project2026\RecoveryReports\MainDevApiRoleContractAndSyncHappyPathV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ARTIFACT_VERIFICATION.md
PROMPT113_BLOCKER_INTAKE.md
PROTECTED_PROVISIONING_BOUNDARY_RECOVERY.md
PROMPT113_DISCOVERY_GAP.md
ROLE_MEMBERSHIP_AND_OWNERSHIP_BEFORE.md
RUNTIME_OPERATION_MATRIX.md
ROLE_GRANT_CONTRACT.md
PROVISIONING_SOURCE_BEFORE.md
PROVISIONING_SOURCE_AFTER.md
ROLE_CONTRACT_APPLY_PROOF.md
MIGRATION_ROLE_HISTORY_PROOF.md
RUNTIME_POSITIVE_PERMISSION_PROOF.md
RUNTIME_NEGATIVE_PERMISSION_PROOF.md
API_RUNTIME_CREDENTIAL_SEPARATION.md
API_RUNTIME_DB_PROOF.md
SYNC_AUTH_CALL_CHAIN.md
DEVELOPMENT_PREREQUISITES.md
PRICE_RULE_SAVE_PROOF.md
WPF_WORKER_PROOF.md
GROUPED_HTTP_PROOF.md
API_TRANSACTION_PROOF.md
SIGNALR_AFTER_COMMIT_PROOF.md
LOCAL_COMPLETION_PROOF.md
HAPPY_PATH_COUNTS.md
MANUAL_TEST_READINESS.md
FOCUSED_TEST_OUTPUT.txt
ARCHITECTURE_GUARD_OUTPUT.txt
FINAL_STATE.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

## Public report

Create and push only:

```text
report/report114.md
```

Include:

```text
Verdict
Prompt107 artifact SHA verified yes/no
Prompt108 artifact SHA verified yes/no
Prompt113 artifact SHA verified yes/no
Protected provisioning boundary recovered yes/no
Protected credential source name only
Prompt113 discovery gap classification
Canonical migration/provisioning role identified yes/no
Grant/provisioning source-of-truth corrected yes/no
Role contract applied idempotently yes/no
Runtime role CREATEDB before/after
Runtime role CREATEROLE after
Runtime role SUPERUSER after
Runtime role object/database/schema owner after yes/no
Broad GRANT ALL used yes/no
Runtime SELECT on __EFMigrationsHistory granted yes/no and reason
Migration history proof yes/no
API pending migrations count
Runtime positive application permission proof yes/no
Runtime negative DDL/admin permission proof yes/no
Canonical API runtime proof yes/no
ApiServer loopback readiness yes/no
Admin/provisioning credential absent from normal runtime process yes/no
Production sync auth physically passed yes/no
Development prerequisites created yes/no
Production Price Rule Save used yes/no
Local domain + outbox atomic commit yes/no
Local group expected/actual count
Local group sequence/order proof yes/no
Existing periodic WPF worker used yes/no
Production worker cycle count
Grouped HTTP request count
Canonical API ingest action used yes/no
API transaction begin count
API durable commit count
TblEventLog group row count
TblEventDelivery destination row count
TblEventDelivery source row count
Source exclusion proof yes/no
SignalR attempted after commit yes/no
SignalR publish succeeded yes/no
Destination notification observed yes/no/not-applicable
WPF group rows marked Sent count
All-or-none local completion proof yes/no
Parallel production path invoked count
Happy-path marker present yes/no
Manual POS1 test ready true/false
Focused test totals
Architecture guard totals
WPF build totals
API build totals
WPF/API DB reset performed yes/no
Manual outbox/event/delivery insertion used yes/no
Category Weight changed yes/no
Booking Weight changed yes/no
Production/customer/reference DB mutated yes/no
Private artifact yes/no
Aggregate SHA-256
```

Do not expose passwords, tokens, complete connection strings, passfile contents, raw Development identities, or business payload values.

## Verdicts

PASS:

```text
OBM_MAIN_DEV_API_ROLE_CONTRACT_AND_CANONICAL_SYNC_HAPPY_PATH_READY_FOR_MANUAL_POS1_TEST
```

Narrow blockers only:

```text
BLOCKED_MAIN_DEV_API_PROVISIONING_CREDENTIAL_UNAVAILABLE
BLOCKED_MAIN_DEV_API_PROVISIONING_BOUNDARY_RECOVERY
BLOCKED_MAIN_DEV_API_ROLE_CONTRACT
BLOCKED_MAIN_DEV_API_GRANT_APPLY
BLOCKED_MAIN_DEV_API_MIGRATION_HISTORY
BLOCKED_MAIN_DEV_API_RUNTIME_PERMISSION
BLOCKED_MAIN_DEV_API_RUNTIME_STARTUP
BLOCKED_MAIN_DEV_SYNC_AUTH
BLOCKED_MAIN_DEV_DESTINATION_ROUTING
BLOCKED_MAIN_DEV_PRICE_RULE_SAVE
BLOCKED_MAIN_DEV_GROUP_UPLOAD
BLOCKED_MAIN_DEV_API_COMMIT
BLOCKED_MAIN_DEV_SIGNALR
BLOCKED_MAIN_DEV_LOCAL_COMPLETION
BLOCKED_MAIN_DEV_HAPPY_PATH_TESTS
```

Every blocked result must include:

```text
exact failed class/script/method/command
exact protected source name or missing boundary
sanitized exception chain and SQLSTATE when available
runtime role attributes before/after
which grants/role changes were or were not applied
all database/write/reset state
MANUAL_POS1_TEST_READY=false
```

Do not return a generic role, credential, startup, or sync blocker.
