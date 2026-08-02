# Prompt 113 — Close the canonical API PostgreSQL migration-history permission boundary and complete the single POS1 → API happy path

## Starting checkpoint

Prompt112 returned:

```text
BLOCKED_MAIN_DEV_API_STARTUP
```

Coordination references:

```text
report/report112.md
report112 commit:
8d85de56ff3b242afea04748de8b49cc94455e62

prompt112 private artifact aggregate SHA-256:
6cf0b248fe2888e5605b54c22bcb09d8a3d0bc3acf4e2efa913cf7cd83e3a7e6
```

Report112 proves the original startup credential blocker is closed:

```text
start-api-local.ps1 / LocalDevelopment identified
protected PostgreSQL runtime credential source resolved
hidden password prompt unreachable in canonical automation
canonical API DB runtime proof = yes
API startup non-interactive = yes
ApiServer loopback readiness = yes
runtime database = obm_api_dev_v0_pg
```

The new exact blocker is:

```text
SQLSTATE 42501 while reading __EFMigrationsHistory
API pending migrations could not be proven
happy path stopped before identity/routing/domain/outbox/event/delivery/SignalR writes
```

This task must classify and repair the PostgreSQL ownership/privilege boundary narrowly, then resume the same physical happy path from prompt111/prompt112.

## Authoritative architecture locks

### Database roles

Do not solve this by making the normal ApiServer runtime role:

```text
SUPERUSER
CREATEDB
CREATEROLE
database owner
schema owner
table owner
migration owner
```

Do not use:

```text
GRANT ALL ON DATABASE
GRANT ALL ON SCHEMA
GRANT ALL ON ALL TABLES
blanket default privileges broader than the proven runtime contract
```

The canonical design must distinguish, when the current source/provisioning already supports it:

```text
maintenance/admin role — destructive DB maintenance only
migration/provisioning role — schema migration and grants
runtime application role — only the permissions needed for normal ApiServer operation
```

Do not create a second credential framework or second database-provisioning pipeline.

### Authentication

The retired Firebase email/password path remains deleted.

WPF installation/bootstrap remains:

```text
Pairing Code -> redeem -> WpfJwt -> bootstrap/me
```

Do not broaden WpfJwt into an unrelated PostgreSQL or runtime-sync credential.

### Sync

There remains exactly one canonical sync flow:

```text
WPF domain Save + TblLocalOutbox in one local transaction
-> existing periodic WPF outbox worker
-> existing canonical FlushOutbox/API service
-> existing standard API grouped ingest controller/service
-> one TblEventLog/TblEventDelivery transaction
-> one durable API commit
-> existing post-commit SignalR notification
```

Do not create another uploader, worker, endpoint, ingest service, event writer, delivery path, ACK path, or SignalR publisher.

## Strict scope

Execute only:

```text
1. Read and verify the complete prompt112 private artifact.
2. Capture the exact SQLSTATE 42501 exception, object, schema, role, and query/call site.
3. Audit database/schema/table/sequence ownership and effective grants for the canonical API runtime role.
4. Determine whether __EFMigrationsHistory is read by actual ApiServer startup/runtime code or only by the prompt proof command.
5. Define the smallest least-privilege role matrix required by the real production runtime.
6. Repair the canonical source/provisioning/grant boundary and apply it safely to obm_api_dev_v0_pg.
7. Prove API migration history and pending migrations = 0 through the correct role/boundary.
8. Prove the normal runtime role can perform required application operations but cannot perform schema/destructive administration.
9. Resume and complete the full canonical POS1 -> API grouped-sync happy path.
10. Rerun focused tests, architecture guards, and WPF/API builds.
```

Do not execute:

```text
DB reset/drop/recreate
new migration generation unless a narrow source-owned grant defect genuinely requires an attached migration/provisioning update
15-case failure/recovery matrix
Category Weight implementation
Booking Weight implementation
POS2 pull/apply/ACK
checkout/payment changes
Queue changes
BookingConsole changes
cloud/production deployment
```

Do not mutate production/customer/reference databases.

## Canonical SQL template policy

The future derived SQL template location remains:

```text
E:\Project2026\2SQL PostgreSQL
```

There will eventually be exactly two canonical generated/template lanes:

```text
WPF/local OBM-POS database creation
ApiServer database creation
```

Do not finalize or overwrite those scripts in this task because Category Weight and Booking Weight may still change schema.

However, record the final proven API role/grant contract so the later schema-freeze SQL-export task can include it. EF/Npgsql models, migrations, and canonical provisioning code remain source of truth.

## Required evidence intake

Read completely:

```text
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
prompt/prompt112_SQL_TEMPLATE_POLICY_ADDENDUM.md
```

Read and verify:

```text
E:\Project2026\RecoveryReports\MainApiDevResetExecutionV001
aggregate SHA-256:
e9d8298486f31f40581cb4445fa0abac25030bd586303098c05e1a9225f0d0ea

E:\Project2026\RecoveryReports\LegacyFirebaseUserSecretRemovalV001
aggregate SHA-256:
b97b2eaed1738c497502c92b057c5133bf6b20345d302b3daea44541d0012dfa

E:\Project2026\RecoveryReports\CanonicalSyncFlowConsolidationV001
aggregate SHA-256:
a7d113ef381c07095b3ccd4145de734d4011e5eb51a78d2f6c7f6095ae868ccd

E:\Project2026\RecoveryReports\MainDevApiStartupAndSyncHappyPathV001
aggregate SHA-256:
6cf0b248fe2888e5605b54c22bcb09d8a3d0bc3acf4e2efa913cf7cd83e3a7e6
```

At minimum inspect:

```text
start-api-local.ps1 and prompt112 changes
Program.cs startup/readiness/migration checks
ExternalDbContext runtime construction
ExternalDbContext design-time factory and migration executor
all calls to GetPendingMigrations/GetAppliedMigrations/Migrate/EnsureCreated
canonical API reset/provisioning/grant utilities
migration files and model snapshot
PostgreSQL role/ownership/grant queries used by existing tooling
__EFMigrationsHistory schema/table mapping
application schema search_path assumptions
all API tables/sequences/functions required by runtime
```

Never print passwords, connection strings, passfile contents, token values, or secret values.

Record before editing:

```text
PROMPT112_ARTIFACT_VERIFIED=true
CANONICAL_API_DB=obm_api_dev_v0_pg
POSTGRES_SQLSTATE=42501
DB_RESET=FORBIDDEN
SUPERUSER_OR_OWNER_ESCALATION=FORBIDDEN
BLANKET_GRANT_ALL=FORBIDDEN
SYNC_ARCHITECTURE_CHANGE=FORBIDDEN
```

## Phase 1 — Capture the exact permission failure

Capture sanitized direct evidence:

```text
FAILED_COMMAND_OR_TEST=<exact command/test>
FAILED_CLASS_METHOD=<exact class/method when applicable>
FAILED_SQL_OPERATION=<SELECT/other>
FAILED_OBJECT_SCHEMA=<safe schema name>
FAILED_OBJECT_NAME=__EFMigrationsHistory or exact object
CONNECTED_DATABASE=obm_api_dev_v0_pg
EFFECTIVE_RUNTIME_ROLE=<safe role name or stable sanitized alias>
OBJECT_OWNER=<safe role name or stable sanitized alias>
DATABASE_OWNER=<safe role name or stable sanitized alias>
SCHEMA_OWNER=<safe role name or stable sanitized alias>
EXACT_SANITIZED_EXCEPTION=<message without secrets>
POSTGRES_SQLSTATE=42501
```

Determine exactly which condition applies:

```text
A. runtime role lacks USAGE on application schema
B. runtime role lacks SELECT on __EFMigrationsHistory
C. table owner/default privileges differ from the intended provisioning role
D. search_path resolves a different/inaccessible history table
E. prompt proof used the runtime role when the canonical migration role should be used
F. actual ApiServer startup incorrectly requires migration-history access
G. another narrowly proven PostgreSQL permission defect
```

Do not infer from SQLSTATE alone.

## Phase 2 — Map the canonical role contract

Produce an explicit permission matrix for:

```text
maintenance/admin role
migration/provisioning role
normal ApiServer runtime role
```

For each role prove whether it needs:

```text
CONNECT database
USAGE schema
CREATE schema objects
SELECT/INSERT/UPDATE/DELETE application tables
USAGE/SELECT/UPDATE sequences
SELECT __EFMigrationsHistory
ALTER/DROP/CREATE tables
execute functions/procedures
```

Rules:

```text
runtime role receives only permissions required by reachable production code
migration/provisioning role owns or can alter migration-created objects
maintenance role remains outside normal request/runtime operation
```

If actual startup code calls migration APIs, determine whether that is an intentional canonical deployment contract or an accidental coupling. Do not remove a legitimate safety check merely to avoid a grant; do not grant schema-owner rights merely to preserve accidental startup migration behavior.

## Phase 3 — Repair the source-of-truth grant/provisioning boundary

Use the smallest existing canonical boundary. Possible valid repairs, only when proven by Phase 1/2, include:

```text
add the missing narrow GRANT SELECT on __EFMigrationsHistory to the runtime role
add schema USAGE
add required CRUD/sequence grants for application runtime
correct ALTER DEFAULT PRIVILEGES for future migration-created objects under the proven migration owner
make the migration-current proof use the canonical migration/provisioning role while keeping runtime startup least-privileged
remove accidental runtime migration-history access if startup does not require it and provide a separate canonical readiness/provisioning proof
correct search_path/history schema configuration
```

Requirements:

```text
source/provisioning contract updated, not physical DB only
canonical Development DB corrected through the existing protected admin/migration boundary
repeatable/idempotent grant application
no secret in source or artifacts
no broad role escalation
no second permission framework
```

If the current source has no canonical role/grant owner and the contract cannot be proved safely, return:

```text
BLOCKED_MAIN_DEV_API_ROLE_CONTRACT
```

with exact ownership/grant evidence.

## Phase 4 — Prove migration-current and runtime least privilege

Required positive proof:

```text
canonical API DB identity = obm_api_dev_v0_pg
migration history expected chain readable through the correct canonical boundary
migration history exact-once
pending migrations = 0
ApiServer runtime starts non-interactively on loopback
readiness succeeds
```

Required runtime proof:

```text
runtime role can read/write the application tables required by the one happy path
runtime role can use required identity sequences
runtime role can execute required functions when any are proven
```

Required negative proof:

```text
runtime role cannot CREATE/DROP/ALTER application tables
runtime role cannot create databases or roles
runtime role is not owner/superuser
maintenance credential is not used for normal API requests
```

Use transaction-rolled-back or other non-persistent probes where practical.

## Phase 5 — Resume the canonical happy path

After the permission/migration-current gate passes, resume prompt112 from the first write boundary.

### Development-only prerequisites

Create only the existing-contract prerequisites for:

```text
one generated Development tenant
one generated POS1 source identity
one generated POS2 destination identity/subscriber mapping
subscriptions/routing for exactly TblTurnPolicy and TblTurnAmountRule
minimal Price Rule Save prerequisites
```

Do not reuse production/customer identifiers.

Do not manually insert:

```text
TblLocalOutbox
TblEventLog
TblEventDelivery
fabricated Price Rule outbox payloads
```

### Real Price Rule Save

Use the production Price Rule Save boundary and prove:

```text
one DbContext
one explicit local PostgreSQL transaction
domain rows + complete TblLocalOutbox group committed together
true change, not no-op
ExpectedEventCount equals actual count
contiguous SequenceNumber
parent-first ordering
```

### Existing WPF worker and grouped upload

Use the one existing registered production periodic WPF outbox worker and canonical FlushOutbox chain.

Prove:

```text
one worker cycle
one atomic group claim
one grouped HTTP request
no parallel path invoked
```

### API commit and SignalR

For the generated TransactionGuid prove:

```text
production sync auth/identity validation passed
one API transaction began
TblEventLog count = ExpectedEventCount
TblEventDelivery destination count = ExpectedEventCount per valid destination
TblEventDelivery source count = 0
one durable API commit
commit completed before SignalR publish
one existing SignalR publisher path
notification-only metadata
publish succeeded
```

### Local completion

Prove all rows in the local group are marked Sent together, SentAt populated, claim fields cleared, and no mixed status remains.

Required marker:

```text
SYNC_GROUP_MAIN_DEV_HAPPY_PATH_COMMITTED
```

## Phase 6 — Focused tests and builds

Run focused tests for:

```text
exact SQLSTATE 42501 regression
role ownership/grant matrix
runtime schema USAGE/table/sequence permissions
migration-history proof through correct role
pending migrations = 0
runtime negative DDL privilege proof
non-interactive canonical API startup
production worker -> canonical FlushOutbox chain
complete-group claim/validation
one grouped request
sync auth/identity match
atomic EventLog/Delivery commit
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

Build/test success does not override failed physical permission, migration-current, or happy-path proof.

## End state

Leave:

```text
canonical API DB migration-current
canonical WPF DB migration-current
ApiServer runtime role least-privileged and operational
migration/provisioning ownership/grant contract documented and repeatable
no hidden credential prompt
one successful canonical POS1 -> API grouped-sync happy path
Development-only prerequisites documented for later failure matrix
future E:\Project2026\2SQL PostgreSQL export requirements recorded but scripts not finalized
no production/customer/reference mutation
```

## Required private artifact

Preserve prior artifacts unchanged. Create:

```text
E:\Project2026\RecoveryReports\MainDevApiPermissionAndSyncHappyPathV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ARTIFACT_VERIFICATION.md
PROMPT112_BLOCKER_INTAKE.md
EXACT_42501_FAILURE.md
ROLE_OWNERSHIP_MATRIX.md
EFFECTIVE_GRANTS_BEFORE.md
RUNTIME_CALL_CHAIN_PERMISSION_REQUIREMENTS.md
GRANT_PROVISIONING_DECISION.md
GRANT_PROVISIONING_BEFORE.md
GRANT_PROVISIONING_AFTER.md
EFFECTIVE_GRANTS_AFTER.md
MIGRATION_HISTORY_PROOF.md
PENDING_MIGRATIONS_PROOF.md
RUNTIME_LEAST_PRIVILEGE_POSITIVE_PROOF.md
RUNTIME_LEAST_PRIVILEGE_NEGATIVE_PROOF.md
API_STARTUP_PROOF.md
SYNC_AUTH_CALL_CHAIN.md
DEVELOPMENT_PREREQUISITES.md
PRICE_RULE_SAVE_PROOF.md
WPF_WORKER_PROOF.md
GROUPED_HTTP_PROOF.md
API_TRANSACTION_PROOF.md
SIGNALR_AFTER_COMMIT_PROOF.md
LOCAL_COMPLETION_PROOF.md
HAPPY_PATH_COUNTS.md
SQL_TEMPLATE_FUTURE_GRANT_CONTRACT.md
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
report/report113.md
```

Include:

```text
Verdict
Prompt112 artifact SHA verified yes/no
Exact 42501 failed command/method
Exact inaccessible object/schema
Failure classification A-G
Runtime role is superuser/owner yes/no
Canonical migration/provisioning role identified yes/no
Grant/provisioning source-of-truth corrected yes/no
Broad GRANT ALL used yes/no
Runtime SELECT migration history required by actual startup yes/no
Migration history proof yes/no
API pending migrations count
Runtime positive application permission proof yes/no
Runtime negative DDL/admin permission proof yes/no
Canonical API DB runtime proof yes/no
API startup non-interactive yes/no
ApiServer loopback readiness yes/no
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
Focused test totals
Architecture guard totals
WPF build totals
API build totals
WPF/API DB reset performed yes/no
Manual outbox/event/delivery insertion used yes/no
Production/customer/reference DB mutated yes/no
Private artifact yes/no
Aggregate SHA-256
```

Do not expose passwords, tokens, full connection strings, passfile contents, raw Development identities, private business payloads, or sensitive role credential values.

## Verdicts

PASS:

```text
OBM_MAIN_DEV_API_LEAST_PRIVILEGE_AND_CANONICAL_SYNC_HAPPY_PATH_READY_FOR_FAILURE_MATRIX
```

Narrow blockers only:

```text
BLOCKED_MAIN_DEV_API_ROLE_CONTRACT
BLOCKED_MAIN_DEV_API_SCHEMA_USAGE
BLOCKED_MAIN_DEV_API_MIGRATION_HISTORY_PERMISSION
BLOCKED_MAIN_DEV_API_RUNTIME_TABLE_PERMISSION
BLOCKED_MAIN_DEV_API_RUNTIME_SEQUENCE_PERMISSION
BLOCKED_MAIN_DEV_API_MIGRATION_CURRENT
BLOCKED_MAIN_DEV_API_STARTUP
BLOCKED_MAIN_DEV_SYNC_AUTH
BLOCKED_MAIN_DEV_DESTINATION_ROUTING
BLOCKED_MAIN_DEV_PRICE_RULE_SAVE
BLOCKED_MAIN_DEV_GROUP_UPLOAD
BLOCKED_MAIN_DEV_API_COMMIT
BLOCKED_MAIN_DEV_SIGNALR
BLOCKED_MAIN_DEV_LOCAL_COMPLETION
BLOCKED_MAIN_DEV_HAPPY_PATH_TESTS
```

A blocked result must name the exact role/object/grant/call site, sanitized PostgreSQL error and SQLSTATE, and all write/reset state. Do not return another generic API-startup blocker.
