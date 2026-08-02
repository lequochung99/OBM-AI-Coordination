# Prompt 096 — Implement the clean transaction-group schema and migrations from zero

## Authoritative clean-slate decision

The operator has declared the current POS and API databases disposable development databases.

Do not preserve old development rows, partial outbox history, mixed old/new contracts, or backward compatibility that obstructs the canonical transaction-group architecture.

This task is Phase A only:

```text
schema
entity models
EF mappings
forward migrations
migration-from-zero proof
```

Do not implement the Price Rule sender, whole-group uploader, API ingest endpoint, pull/apply loop, or group ACK runtime behavior in this task except for minimal DTO/entity skeletons required to compile the schema changes.

Those runtime phases will follow only after the schema is physically proven from empty PostgreSQL databases.

## Source evidence to read before editing

Read completely:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report094.md
report/report095.md
prompt/prompt095.md
```

Read every file in the prompt095 private artifact:

```text
E:\Project2026\RecoveryReports\CleanTransactionGroupSyncV001
```

At minimum:

```text
PRIVATE_HANDOFF.md
WPF_SCHEMA.md
API_SCHEMA.md
MIGRATIONS.md
ARCHITECTURE.md
UNIFIED_DIFF.patch
TEST_OUTPUT.txt
```

Record locally before the first edit:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
ARCHITECTURE=POSTGRESQL_ONLY_CLEAN_TRANSACTION_GROUP
PHASE=SCHEMA_AND_MIGRATIONS_ONLY
LEGACY_DEV_DATA_PRESERVATION=NOT_REQUIRED
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Proven blocker

The first blocking boundary is the schema.

Current WPF `TblLocalOutbox` already has:

```text
TenantGuid
EventGuid
TransactionGuid
SourceClientId
SequenceNumber
EntityType
EntityGuid nullable
Operation
Payload
Sent
Processor
AttemptCount
LastAttemptAt
ErrorMessage
```

but lacks the persisted group completeness and lease contract:

```text
ExpectedEventCount
non-null EntityGuid
unique source group sequence
sequence check constraints
group claim owner and expiry
NextAttemptAt
SentAt
pending complete-group index
retry scheduling index
```

Current API event/delivery storage also lacks a complete canonical group contract. Pull is flat-event based and ACK is event/prefix based.

Do not stop because migrations or breaking development-only schema changes are required. This prompt explicitly authorizes them.

## Canonical data types

Before coding, choose and document one consistent type for each concept across:

```text
WPF model
WPF PostgreSQL schema
API request DTO
API event model
API delivery model
API PostgreSQL schema
WPF pull DTO
```

Required concepts:

```text
TransactionGuid = uuid / Guid
SequenceNumber = one consistent integer type
ExpectedEventCount = one consistent integer type
EntityGuid = uuid / Guid, non-null for canonical transaction-group events
SourceClientId = bounded string
DestinationClientId = bounded string
Operation = exactly I/U/D
```

Do not leave WPF `int` and API `long` disagreement without an explicit justified conversion contract. Prefer one type everywhere.

## PostgreSQL-only mapping cleanup

The prompt095 evidence shows active generated mappings containing SQL Server expressions or semantics, including examples such as:

```text
sysutcdatetime()
newid()
IsRowVersion() over byte[]
```

Audit the active mappings for all tables touched by this task:

```text
TblLocalOutbox
TblEventLog
TblEventDelivery
TblEventDeliveryGroupAck
```

Replace active SQL Server-only defaults or concurrency semantics with proven PostgreSQL equivalents.

Requirements:

```text
no active sysutcdatetime()
no active newid()
no SQL Server rowversion assumption
PostgreSQL-compatible UUID default
PostgreSQL-compatible UTC/local timestamp default consistent with the project's timestamp policy
PostgreSQL-compatible optimistic concurrency model or no false concurrency claim
```

Do not merely make migrations compile while leaving invalid runtime mappings.

Return the exact before/after mapping code and explain each PostgreSQL decision.

## WPF canonical `TblLocalOutbox` schema

Audit existing equivalent fields and avoid redundant duplicates.

The final model must persist at least:

```text
OutboxId or existing stable PK
TenantGuid
EventGuid
TransactionGuid
SourceClientId
SequenceNumber
ExpectedEventCount
EntityType
TableId when still required by canonical sync
EntityId when still required
EntityGuid non-null
Operation
Payload
Sent/status
Processor when still required
ClaimedBy
ClaimExpiresAt
AttemptCount
NextAttemptAt
LastAttemptAt
ErrorMessage safe state
ServerEventSequence when still required
CreatedAt
OccurredAt
SentAt
UpdatedAt
```

Required constraints:

```text
SequenceNumber >= 1
ExpectedEventCount >= 1
SequenceNumber <= ExpectedEventCount
unique TenantGuid + SourceClientId + TransactionGuid + SequenceNumber
```

Required indexes:

```text
pending complete-group selection
retry scheduling
group lease/claim recovery
existing processor/status behavior when still canonical
```

The index design must support selecting the oldest eligible complete group without scanning all historical sent rows.

Do not backfill old development rows.

## API canonical event schema

Extend or correct the real canonical API source event table, currently represented by `TblEventLog`.

Final model must persist:

```text
EventSequence PK
EventGuid
TenantGuid
SourceClientId
TransactionGuid
SequenceNumber
ExpectedEventCount
EntityType
EntityGuid
EntityId when canonical
TableId when canonical
Operation
Payload
OccurredAt
CreatedAt
Published/IsSent only when still part of the canonical durable flow
LastStatusChange
ProducerId
ServerPublishedAt
```

Required constraints/indexes:

```text
unique TenantGuid + SourceClientId + TransactionGuid + SequenceNumber
unique TenantGuid + EventGuid
SequenceNumber >= 1
ExpectedEventCount >= 1
SequenceNumber <= ExpectedEventCount
transaction-group lookup index
existing publication indexes only when still used
```

Do not retain redundant status columns solely because disposable dev rows used them. Keep only what current runtime or the planned group flow actually requires, and document the decision.

## API canonical delivery schema

Extend or correct `TblEventDelivery` so one destination's complete group can be queried and acknowledged atomically later.

Final model must persist:

```text
Id PK
EventSequence FK
TenantGuid
SourceClientId
DestinationClientId
SubscriberId when still required by routing
TransactionGuid
SequenceNumber
ExpectedEventCount
EntityType
EntityGuid
Operation
PayloadJson
Status
AttemptCount
RetryCount only when not redundant
CreatedAt
OccurredAt
NextAttemptAt
DeliveredAt
AcknowledgedAt
LastStatusChange
concurrency field only when PostgreSQL-compatible and actually used
```

Required constraints/indexes:

```text
unique EventSequence + DestinationClientId
complete pending group pull index by tenant + destination + status + transaction + sequence
retry scheduling index
SequenceNumber >= 1
ExpectedEventCount >= 1
SequenceNumber <= ExpectedEventCount
```

If both `SubscriberId` and `DestinationClientId` remain, prove their distinct meanings. Do not keep duplicate identity concepts without justification.

## Group acknowledgement schema

Add a canonical API group acknowledgement entity/table, for example `TblEventDeliveryGroupAck`, unless the existing schema has an equivalent complete-group ACK owner.

Required fields:

```text
Id PK
TenantGuid
DestinationClientId
TransactionGuid
ExpectedEventCount
AcknowledgedAt
AcknowledgedBy
```

Required uniqueness:

```text
unique TenantGuid + DestinationClientId + TransactionGuid
```

Required check:

```text
ExpectedEventCount >= 1
```

Do not implement the ACK endpoint in this task. Only make the schema/model ready.

## Migration mechanism audit

Determine the actual canonical migration mechanism separately for:

```text
WPF eNailSalonDbContext
ApiServer ExternalDbContext
```

Use one real mechanism per DbContext:

```text
EF Core migrations
or
existing canonical versioned SQL migration runner
```

Do not create fake EF migration files that are never applied.

Do not add manual one-off SQL outside the canonical migration runner.

Return:

```text
current migration mechanism
new migration file paths
migration IDs/versions
how empty DB applies them
how pending migrations are detected
```

## Clean migration-chain policy

Because current data is disposable, the migration chain may be repaired when old development-only migrations conflict with a valid empty-database build.

Allowed:

```text
new forward migrations
controlled replacement of invalid dev-only migration steps
removal of dead compatibility branches
schema defaults changed for PostgreSQL
constraints added without backfill
```

Forbidden:

```text
silently deleting migration history without evidence
leaving two competing migration lanes
manual schema mutation outside migration source
mutating the operator's current canonical dev DB
```

Produce a clear migration-chain decision:

```text
append-only forward migration
or
controlled clean-baseline repair
```

and explain why.

## DTO skeletons — compile readiness only

Add only the minimal compile-time DTO/model skeletons needed by the new schema phase, such as:

```text
SyncTransactionGroupRequest
SyncTransactionGroupEventRequest
SyncTransactionGroupResponse
SyncDeliveryGroupDto
SyncDeliveryGroupEventDto
SyncDeliveryGroupAckRequest
```

Do not wire endpoints or runtime services in this task.

The DTO fields must exactly match the canonical schema concepts and use the same sequence/count types.

## Fresh disposable database proof

Create new versioned disposable PostgreSQL databases. Do not use or mutate the operator's current canonical dev databases.

Minimum:

```text
one fresh WPF schema DB
one fresh API schema DB
```

Applying from zero must prove:

```text
all migrations applied
pending migrations = 0
all target tables exist
all required columns exist
correct PostgreSQL data types
correct nullability
correct PostgreSQL defaults
all check constraints exist
all unique indexes exist
all pending/retry/lease indexes exist
foreign keys exist
no SQL Server-only default expression remains
```

Then run direct schema-behavior tests.

### WPF constraint tests

Prove PostgreSQL rejects:

```text
SequenceNumber = 0
ExpectedEventCount = 0
SequenceNumber > ExpectedEventCount
duplicate TenantGuid + SourceClientId + TransactionGuid + SequenceNumber
null EntityGuid
```

Prove valid rows with contiguous group metadata insert successfully.

### API constraint tests

Prove PostgreSQL rejects:

```text
duplicate source group sequence
duplicate TenantGuid + EventGuid
invalid sequence/count relationship
duplicate destination delivery for the same source event and destination
duplicate destination group ACK
```

Prove valid event, delivery, and ACK rows insert successfully.

Drop only the disposable proof databases after capturing evidence, or preserve them under a clearly versioned disposable name when required for the next phase. Do not touch current dev DBs.

## Build and focused tests

Run:

```text
WPF build
ApiServer build
schema/migration-focused tests
PostgreSQL-only provider guard
active SQL Server token scan for touched runtime files
```

Do not run checkout/payment.

Do not run unrelated broad migration suites that fail because of known missing external artifacts unless those artifacts are directly part of this schema phase. Report unrelated failures separately; do not misclassify them as this task's failure.

## Required private evidence artifact

Create a new versioned local folder, never overwrite earlier evidence, for example:

```text
CleanTransactionGroupSchemaV001
```

Required files:

```text
PRIVATE_HANDOFF.md
WPF_ENTITY_MODEL.md
WPF_EF_MAPPING.md
WPF_MIGRATION.md
API_ENTITY_MODELS.md
API_EF_MAPPING.md
API_MIGRATION.md
POSTGRESQL_MAPPING_CLEANUP.md
SCHEMA_MATRIX.md
CONSTRAINT_INDEX_MATRIX.md
MIGRATION_CHAIN_DECISION.md
DISPOSABLE_DB_PROOF.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
TEST_OUTPUT.txt
SHA256SUMS.txt
```

Do not push these detailed files to GitHub.

## Mandatory private handoff

Return directly to the operator:

1. Exact migration mechanism for WPF and API.
2. Exact migration-chain decision.
3. Complete before/after C# entity classes.
4. Complete before/after EF mappings.
5. Complete migration code/SQL.
6. Exact PostgreSQL replacements for SQL Server defaults/concurrency mappings.
7. Complete schema matrix: column, type, nullability, default, constraint, index.
8. Disposable DB names and creation/apply/drop commands with credentials omitted.
9. Full sanitized migration and constraint test output.
10. Build/test counts.
11. Unified diff for every changed schema/model/migration file.
12. Explicit list of runtime sender/uploader/API/pull/ACK files intentionally left unchanged for later phases.
13. Local evidence artifact path and SHA-256 manifest.

Do not return only a summary or test counts.

## Mutation boundaries

Allowed:

```text
WPF entity/model/mapping edits
API entity/model/mapping edits
canonical migrations
schema DTO skeletons needed for compile
focused schema tests
disposable DB create/migrate/test/drop
```

Forbidden:

```text
mutating current operator dev DBs
implementing sender/uploader/API runtime behavior in this phase
implementing pull/apply/ACK runtime behavior in this phase
checkout/payment tests
TurnEngine calculation changes
Price Rule UI behavior changes
OBM source commit/push
credentials in artifacts or reports
```

## Public report

Create and push only:

```text
report/report096.md
```

It may contain only:

```text
verdict
WPF group schema implemented yes/no
API event/delivery/ACK schema implemented yes/no
PostgreSQL-only mappings corrected yes/no
migration from zero proven yes/no
constraints/indexes proven yes/no
DTO skeletons aligned yes/no
WPF build result
API build result
focused test counts
current operator DB automatically mutated no
checkout tested no
one aggregate evidence SHA-256
```

## Valid verdicts

```text
OBM_CLEAN_TRANSACTION_GROUP_SCHEMA_READY_FOR_SENDER_IMPLEMENTATION
```

```text
BLOCKED_WPF_TRANSACTION_GROUP_MIGRATION
```

```text
BLOCKED_API_TRANSACTION_GROUP_MIGRATION
```

```text
BLOCKED_POSTGRESQL_MAPPING_CLEANUP
```

```text
BLOCKED_MIGRATION_FROM_ZERO_PROOF
```

```text
BLOCKED_BUILD_OR_TEST
```
