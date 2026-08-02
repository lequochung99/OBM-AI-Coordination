# Prompt 108 — Reset and migrate only the canonical API Development database through the approved maintenance boundary

## Starting checkpoint

Report107 passed with:

```text
OBM_MAIN_WPF_DEV_RESET_MIGRATION_READY_FOR_API_RESET
```

Coordination references:

```text
report/report107.md
report107 commit: 122cfde6738c744787e3387e95a928f922d99fe7
prompt107 private artifact aggregate SHA-256:
47f68c634a5984611f3cb8b39ba3999f6005a558ad1e0d64bf998f7f4c2a0c58
```

Report107 proves the canonical WPF Development database is now:

```text
recreated in UTF8
migrated from zero
pending migrations = 0
physical grouped schema proven
focused tests = 8 passed, 0 failed, 0 skipped
```

The WPF Development database is now an accepted clean anchor. Do not reset, reseed, patch, or otherwise mutate it in this task.

Prompt103 already resolved both canonical Visual Studio Development database lanes and selected R1 because Development data is disposable. Prompt098 already accepted the API PostgreSQL migration chain and transaction-group event/delivery schema on an empty PostgreSQL database.

This task closes only the API Development database reset/migration gate.

## Architectural lock

The only canonical sync flow remains:

```text
WPF domain Save + TblLocalOutbox in one local transaction
-> existing periodic WPF outbox bot
-> existing standard API sync controller/service path
-> existing API TblEventLog/TblEventDelivery transaction path
-> successful API commit
-> existing post-commit SignalR notification
```

Do not create or modify:

```text
uploader
periodic outbox bot
API sync endpoint
API ingest pipeline
parallel transaction-group transport
parallel ACK/delivery system
SignalR publisher/path
POS2 pull/apply
```

Do not audit or merge prompt100 runtime sync code yet.

Do not start the transaction-group happy path or the 15-case E2E matrix yet.

Do not seed E2E identities, tenants, subscriptions, routing, Price Rules, outbox rows, events, deliveries, or ACK rows.

## Strict scope

Execute only:

```text
1. Read and verify the relevant accepted private artifacts.
2. Resolve the exact canonical API Visual Studio Development database from the active Development configuration.
3. Prove the target is PostgreSQL/Npgsql, Development-only, local/approved, distinct from the accepted WPF Development database, and not protected production/customer/reference data.
4. Stop only the ApiServer Development process or other proven holders of that exact API target; absence of such a process is stop-ready.
5. Use the existing approved PostgreSQL maintenance/reset boundary against only that API Development target.
6. Close sessions only for the exact target.
7. Drop and recreate the same API Development database name in UTF8.
8. Apply the accepted ExternalDbContext PostgreSQL migration chain from zero.
9. Prove pending migrations = 0 and the transaction-group event/delivery/group-ACK schema physically exists.
10. Rerun focused API migration/design-time/reset safety tests and build the API.
```

## Required evidence intake before execution

Read completely:

```text
prompt/prompt098.md
report/report098.md
prompt/prompt103.md
report/report103.md
prompt/prompt107.md
report/report107.md
```

Read the prompt098 private artifact identified by:

```text
aggregate SHA-256:
213a1ea747625159c071da05bf5497612480d55b4ef35e5be01cd7445e9d0bdd
```

Read the prompt103 private artifact:

```text
E:\Project2026\RecoveryReports\MainVisualStudioDevSyncGroupE2EV001
aggregate SHA-256:
dd5985776be1f431c34f3971be1e38a6b890f7649d60434c335905652f441495
```

Read the prompt107 private artifact:

```text
E:\Project2026\RecoveryReports\MainWpfDevResetExecutionV002
aggregate SHA-256:
47f68c634a5984611f3cb8b39ba3999f6005a558ad1e0d64bf998f7f4c2a0c58
```

At minimum recover from those artifacts:

```text
canonical API Development connection resolution source
safe API Development database name
safe WPF Development database name for distinctness proof
approved host/port boundary
protected administrative credential source name
approved maintenance database
accepted ExternalDbContext migration assembly/factory/executor
accepted API migration identifiers and expected history
physical API schema/constraint/index contract
all prior safety predicates and protected-name refusals
```

Verify each aggregate SHA before destructive execution.

Record locally before reset:

```text
PROMPT098_ARTIFACT_VERIFIED=true
PROMPT103_ARTIFACT_VERIFIED=true
PROMPT107_ARTIFACT_VERIFIED=true
API_ENVIRONMENT=Development
API_PROVIDER=Npgsql
API_HOST_CLASSIFICATION=<loopback or approved local Development>
API_TARGET_DATABASE=<safe exact name>
WPF_ACCEPTED_DATABASE=<safe exact name>
API_TARGET_DIFFERS_FROM_WPF=true
MAINTENANCE_DATABASE=<safe name>
MAINTENANCE_DATABASE_DIFFERS_FROM_API_TARGET=true
PROTECTED_CREDENTIAL_SOURCE=<name only>
RESET_STRATEGY=R1
```

Do not expose credentials, full connection strings, passfile contents, JWTs, private keys, raw tenant/device identifiers, or business payloads.

## Canonical project lane

Use only:

```text
E:\Project2026\1ApiServer\ApiServer01
```

Use the actual Development connection resolution used by the main Visual Studio ApiServer lane.

Do not create:

```text
alternate ExternalDbContext
parallel migration assembly
second design-time factory
second reset utility
new credential framework
test-only database transport
```

Reuse the accepted prompt098 design-time Npgsql factory, migration assembly, and migration executor/reset boundary. If a narrow defect prevents safe execution against the canonical Development target, correct only that defect and add focused regression coverage.

## Phase 1 — Resolve and prove the exact API Development target

Resolve the database through the active ApiServer Development configuration and actual runtime connection-resolution code.

Required safe proof:

```text
Environment = Development
provider = Npgsql/PostgreSQL
host = loopback or explicitly approved local Development host
database name equals the API target resolved in prompt103
database name differs from the WPF Development target accepted in report107
database name differs from the maintenance database
production/customer/reference protected names remain rejected
```

Open a short-lived read-only connection to the exact API target and prove physical identity.

If the protected normal API credential cannot be resolved or authenticated, stop with:

```text
BLOCKED_API_DEV_TARGET_CONNECTION
```

and report the credential source name, exact sanitized exception chain, and SQLSTATE when available.

## Phase 2 — Process and session readiness

Inspect only processes associated with the canonical ApiServer Development lane.

Classify:

```text
no matching ApiServer process = API_PROCESS_STOP_READY=PASS
```

If an ApiServer Development process is running against the exact target, stop only that process through the smallest normal boundary.

Do not stop PostgreSQL globally.

Do not kill unrelated WPF, PlatformAppV0, BookingConsole, browser, IDE, or customer processes.

After establishing the maintenance connection, query sessions only for the exact API Development target.

Required markers:

```text
MAINTENANCE_CONNECTION_SUCCEEDED=true
TARGET_SESSION_COUNT_BEFORE=<integer>
TARGET_SESSION_COUNT_AFTER_CLOSE=0
```

## Phase 3 — Execute the approved API R1 reset

Use the existing approved PostgreSQL maintenance/reset mechanism.

Requirements:

```text
maintenance connection uses PostgreSQL/Npgsql
maintenance connection uses the same approved local Development host/port
maintenance connection uses the approved protected administrative credential source
maintenance connection connects to a non-target maintenance database
target is exact allowlisted API Development database from prompt103
Environment remains Development
protected names remain rejected
only sessions for the exact API target are terminated
DROP only the exact API target
CREATE the same database name
encoding = UTF8
```

Do not manually create application tables.

Do not use `EnsureCreated`.

Do not use `DROP ... CASCADE` against unrelated schemas/databases.

Required reset markers:

```text
API_DEV_RESET_EXECUTED=true
API_DEV_DB_RECREATED=true
API_DEV_DB_ENCODING=UTF8
```

On failure, report:

```text
exact reset utility/class/method/command
exact failed stage
sanitized exception and inner exception chain
PostgreSQL SQLSTATE or NOT_AVAILABLE
maintenance connection established yes/no
session counts before/after
DROP executed yes/no
CREATE executed yes/no
```

## Phase 4 — Apply ExternalDbContext migrations from zero

Use the accepted prompt098 PostgreSQL migration boundary for `ExternalDbContext`.

Required proof:

```text
provider = Npgsql.EntityFrameworkCore.PostgreSQL
normal ASP.NET host not required for design-time migration execution
accepted migration assembly used
complete selected migration chain applied from an empty database
__EFMigrationsHistory contains the expected chain exactly once
pending migrations = 0
no EnsureCreated
no manual application-table creation
no SQL Server provider fallback
```

If migration application exposes a source migration defect:

```text
fix only the smallest production-capable source defect
reset the API Development database again through the same approved maintenance boundary
reapply the full chain from zero
```

Do not patch only the physical database.

Do not alter WPF migrations or the accepted WPF Development database.

## Phase 5 — Physical API schema proof

Prove at minimum:

### TblEventLog

```text
table exists
EventSequence primary/identity contract exists
EventGuid UUID NOT NULL
TenantGuid UUID NOT NULL
SourceClientId required
TransactionGuid UUID NOT NULL
SequenceNumber integer NOT NULL
ExpectedEventCount integer NOT NULL
EntityGuid UUID NOT NULL
SequenceNumber >= 1 check
ExpectedEventCount >= 1 check
SequenceNumber <= ExpectedEventCount check
unique source-group sequence index exists
unique TenantGuid + EventGuid index exists
```

### TblEventDelivery

```text
table exists
EventSequence FK to TblEventLog exists
TenantGuid required
SourceClientId required
DestinationClientId required
SubscriberId required
TransactionGuid UUID NOT NULL
SequenceNumber integer NOT NULL
ExpectedEventCount integer NOT NULL
EntityGuid UUID NOT NULL
sequence/count checks exist
EventSequence + DestinationClientId unique index exists
complete-group pull index exists
retry index exists
no false SQL Server bytea rowversion mapping
```

### TblEventDeliveryGroupAck

```text
table exists
TenantGuid required
DestinationClientId required
TransactionGuid UUID NOT NULL
ExpectedEventCount integer NOT NULL
ExpectedEventCount >= 1 check exists
unique TenantGuid + DestinationClientId + TransactionGuid index exists
```

Prove the three tables are writable with a transaction-rolled-back probe or another non-persistent physical test.

Do not insert persistent sync/E2E data.

## Phase 6 — Preserve the accepted WPF anchor

Do not mutate the WPF Development database.

Use safe read-only evidence from report107/private artifact and, only if necessary, a short-lived read-only verification to record:

```text
WPF_DEV_MUTATED=false
WPF pending migrations remains 0
```

Do not rerun WPF reset or WPF migration execution.

## Phase 7 — Focused tests and build

Run actual focused tests for:

```text
API Development target resolution
protected-name refusal
maintenance database != target
API target != WPF target
session filtering by exact target
API reset safety boundary
ExternalDbContext design-time creation
migration list/history
migration from zero
pending migrations = 0
physical transaction-group schema
```

Expected:

```text
all focused tests pass
0 skipped
```

Build the main ApiServer project after any correction.

Report actual warning and error totals. Existing unrelated warnings do not invalidate PASS when errors are zero and all required physical proof passes.

Build/test PASS does not override a failed physical reset/migration/schema proof.

## End state

Leave the canonical API Development database:

```text
present
UTF8
clean
migration-current
pending migrations = 0
transaction-group event/delivery/group-ACK schema physically proven
no E2E identity/routing/business rows added
```

Leave the accepted WPF Development database unchanged.

Do not start ApiServer runtime sync testing in this task.

## Required private artifact

Create a new versioned artifact:

```text
E:\Project2026\RecoveryReports\MainApiDevResetExecutionV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ARTIFACT_VERIFICATION.md
API_DEV_DB_RESOLUTION.md
WPF_API_DISTINCTNESS_PROOF.md
PROCESS_SESSION_PROOF.md
MAINTENANCE_CONNECTION_PROOF.md
MAINTENANCE_RESET_PROOF.md
MIGRATION_FROM_ZERO_PROOF.md
MIGRATION_HISTORY.md
SCHEMA_PROOF.md
NON_PERSISTENT_WRITE_PROBE.md
FOCUSED_TEST_OUTPUT.txt
FINAL_API_DEV_DB_STATE.md
WPF_PRESERVATION_PROOF.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Do not overwrite or delete earlier artifacts.

## Public report

Create and push only:

```text
report/report108.md
```

Include:

```text
Verdict
Prompt098 artifact SHA verified yes/no
Prompt103 artifact SHA verified yes/no
Prompt107 artifact SHA verified yes/no
Canonical API Development DB resolved yes/no
API Environment Development proof yes/no
API provider Npgsql proof yes/no
API host local/approved proof yes/no
API target differs from WPF target yes/no
Physical API target connection proof yes/no
ApiServer process stop-ready yes/no
Maintenance database differs from target yes/no
Maintenance connection proof yes/no
Target session count before reset
Target session count after close
API Development reset executed yes/no
API Development DB recreated yes/no
UTF8 proof yes/no
ExternalDbContext migration chain applied from zero yes/no
API migration history exact-once proof yes/no
API pending migrations count
TblEventLog physical schema proof yes/no
TblEventDelivery physical schema proof yes/no
TblEventDeliveryGroupAck physical schema proof yes/no
Non-persistent write probe yes/no
Focused test totals
API build totals
WPF Development DB mutated yes/no
WPF pending migrations after task
Sync flow code changed yes/no
E2E/sync data seeded yes/no
Production/customer/reference DB mutated yes/no
Private artifact yes/no
Aggregate SHA-256
```

Do not expose secrets, complete connection strings, tokens, passfile contents, raw tenant/device identifiers, or private business payloads.

## Verdicts

PASS only when the canonical API Development database has been physically reset, recreated in UTF8, migrated from zero, and proven current:

```text
OBM_MAIN_API_DEV_RESET_MIGRATION_READY_FOR_SYNC_FLOW_AUDIT
```

Narrow blockers only:

```text
BLOCKED_API_DEV_ARTIFACT_VERIFICATION
BLOCKED_API_DEV_DB_RESOLUTION
BLOCKED_API_DEV_TARGET_CONNECTION
BLOCKED_API_DEV_ADMIN_CONNECTION
BLOCKED_API_DEV_MAINTENANCE_CONNECTION
BLOCKED_API_DEV_DROP
BLOCKED_API_DEV_CREATE
BLOCKED_API_DEV_MIGRATION_APPLY
BLOCKED_API_DEV_SCHEMA_PROOF
BLOCKED_API_DEV_FOCUSED_TESTS
```

A blocked result must name the exact failed boundary, exact class/method/command, sanitized exception chain, SQLSTATE when available, session/reset state, and whether either Development database was mutated.

Do not return a generic database-reset blocker.
