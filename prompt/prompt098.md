# Prompt 098 — Create the canonical API PostgreSQL migration baseline and transaction-group event/delivery schema

## Operator approval and scope

Prompt097 passed with:

```text
OBM_WPF_POSTGRESQL_BASELINE_AND_OUTBOX_SCHEMA_READY_FOR_API_SCHEMA
```

The WPF migration boundary is now accepted:

```text
Design-time Npgsql factory: proven
Dedicated WPF migration baseline: proven
Disposable empty PostgreSQL apply: proven
Pending migrations: 0
TblLocalOutbox transaction-group schema: proven
Constraints/indexes: proven
Current operator DB mutated: no
```

This task is the next narrow phase and applies only to the ApiServer PostgreSQL schema/migration boundary.

Required outcome:

```text
ExternalDbContext can be created at design time without starting the normal API host.
A real PostgreSQL migration lane exists for the API event/delivery schema.
The transaction-group event, destination delivery, and complete-group ACK schema is physically proven on an empty disposable PostgreSQL database.
```

Do not implement or modify in this task:

```text
Price Rule Save runtime
TblTurnPolicy/TblTurnAmountRule local outbox staging
WPF whole-group uploader
API sync-transaction-group runtime endpoint
API transaction-group ingest service
SignalR notification flow
API group pull runtime
WPF group apply runtime
API group ACK runtime endpoint
BookingConsole runtime behavior
checkout/payment runtime
operator's current WPF or API development databases
```

Those runtime phases begin only after this API schema gate passes.

## Authoritative clean-slate decision

The current API database is a disposable development database.

There is no requirement to:

```text
backfill existing TblEventLog rows
backfill existing TblEventDelivery rows
preserve old prefix/event-level ACK rows
support mixed old/new transaction-group records
preserve SQL Server rowversion behavior
apply this task to the operator's current API DB
```

The final migration must create the correct forward PostgreSQL schema from an empty database.

Do not preserve an invalid migration or model contract merely to protect disposable development data.

Do not automatically drop, recreate, migrate, or seed the operator's current API database.

## Required evidence to read before editing

Read completely:

```text
<API_ROOT>/AGENTS.md when present
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
prompt/prompt095.md
prompt/prompt096.md
prompt/prompt097.md
report/report095.md
report/report096.md
report/report097.md
```

Read every file in the prompt095 and prompt096 private artifacts relevant to API schema:

```text
E:\Project2026\RecoveryReports\CleanTransactionGroupSyncV001
E:\Project2026\RecoveryReports\CleanTransactionGroupSchemaV001
```

At minimum:

```text
API_SCHEMA.md
API_EF_MAPPING.md
API_ENTITY_MODELS.md
API_MIGRATION.md
MIGRATIONS.md
MIGRATION_CHAIN_DECISION.md
POSTGRESQL_MAPPING_CLEANUP.md
PRIVATE_HANDOFF.md
TEST_OUTPUT.txt
UNIFIED_DIFF.patch
```

Record before editing:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
TASK_SCOPE=API_MIGRATION_BASELINE_ONLY
PROVIDER=NPGSQL_ONLY
CURRENT_OPERATOR_DB_MUTATION=FORBIDDEN
LEGACY_DEV_DATA_PRESERVATION=NOT_REQUIRED
RUNTIME_SYNC_IMPLEMENTATION=DEFERRED
```

## Proven prompt096 API blockers

Prompt096 proved that API migration inspection stopped before PostgreSQL access.

The design-time/model-validation failures included:

```text
placeholder API credential/startup path
ExternalDbContext design-time creation not isolated from normal startup
TblBookingConsoleHandoffCode requires a primary key or an explicit keyless contract
no API migration file created
no API model snapshot created
no disposable PostgreSQL API DB created
no migration-from-zero proof
```

It also proved active SQL Server-derived mapping patterns in the touched event/delivery surface:

```text
newid()
sysutcdatetime()
byte[] + IsRowVersion() SQL Server-style rowversion assumption
```

This task must correct those boundaries using PostgreSQL-native contracts.

## Phase 1 — Audit the actual API context and migration ownership

Return exact proof for:

```text
ExternalDbContext namespace and assembly
API executable/startup project
EF Core version
Npgsql EF provider version
current MigrationsAssembly setting
all existing migration folders and context ownership
all __EFMigrationsHistory assumptions
actual PostgreSQL schema name used by ExternalDbContext
whether the API currently uses EF migrations, a versioned SQL runner, or mixed mechanisms
```

Search for all implementations of:

```csharp
IDesignTimeDbContextFactory<ExternalDbContext>
```

If one exists, prove why prompt096 did not use it before changing it.

Do not merge PlatformAppV0-specific migrations, unrelated contexts, test contexts, or WPF migrations into the ExternalDbContext migration chain.

If ExternalDbContext already has a valid PostgreSQL migration chain from zero, extend it rather than creating a conflicting second history.

If no valid chain exists and the disposable DB policy permits it, establish one clean dedicated PostgreSQL baseline for ExternalDbContext.

Document the decision before generating migrations.

## Phase 2 — Fix ExternalDbContext design-time construction

Create or repair a dedicated design-time factory, for example:

```text
Data/Context/DesignTimeExternalDbContextFactory.cs
```

The exact path may follow project conventions.

Requirements:

```text
implements IDesignTimeDbContextFactory<ExternalDbContext>
uses UseNpgsql only
does not start the normal ASP.NET host
does not resolve controllers, SignalR hubs, background services, or PlatformAppV0 runtime services
does not require placeholder startup credentials
does not contain a password or full connection string literal
does not fall back to SQL Server
does not target the operator's current API DB by default
```

Connection resolution must be explicit and fail closed.

Use a dedicated environment variable such as:

```text
OBM_API_MIGRATION_CONNECTION
```

or an already established canonical equivalent.

The factory must:

```text
reject a missing value
reject placeholder credentials
reject protected/current/production DB names
require PostgreSQL/Npgsql
allow PGPASSFILE or the existing protected credential mechanism
redact connection metadata in logs
```

Do not print passwords, passfile contents, host secrets, raw tenant/device GUIDs, or complete connection strings.

## Phase 3 — Resolve TblBookingConsoleHandoffCode model validation correctly

Prompt096 proved this immediate API model blocker:

```text
The entity type 'TblBookingConsoleHandoffCode' requires a primary key to be defined.
```

Audit the actual table/entity contract before editing.

Determine which is true:

```text
A. It is a persisted table with a real primary key.
B. It is a persisted table with a natural/composite key.
C. It is a read-only view/query projection and must be HasNoKey().
D. The entity is stale/dead and should be removed from ExternalDbContext.
```

Use schema, CRUD call sites, migrations/scripts, and runtime code to prove the selection.

Do not invent a synthetic key merely to satisfy EF validation.

Do not mark a writable table keyless.

If the contract cannot be proven safely, stop with:

```text
BLOCKED_API_BOOKING_CONSOLE_HANDOFF_KEY_CONTRACT
```

and return complete evidence.

This repair is permitted only because it is required to establish the real ExternalDbContext migration/model lane. Do not change BookingConsole runtime behavior.

## Phase 4 — Final transaction-group API entity models

Update the real API entity models for the canonical transaction-group schema.

### TblEventLog

Required concepts:

```csharp
public long EventSequence { get; set; }
public Guid EventGuid { get; set; }
public Guid TenantGuid { get; set; }
public string SourceClientId { get; set; } = null!;
public Guid TransactionGuid { get; set; }
public int SequenceNumber { get; set; }
public int ExpectedEventCount { get; set; }
public string EntityType { get; set; } = null!;
public Guid EntityGuid { get; set; }
public string EntityId { get; set; } = null!;
public int TableId { get; set; }
public string Operation { get; set; } = null!;
public string Payload { get; set; } = null!;
public DateTime OccurredAt { get; set; }
public DateTime CreatedAt { get; set; }
public bool Published { get; set; }
public bool IsSent { get; set; }
public DateTime LastStatusChange { get; set; }
public string? ProducerId { get; set; }
public DateTime? ServerPublishedAt { get; set; }
```

Preserve additional proven required fields only when they do not duplicate these concepts.

`EventGuid`, `TenantGuid`, `TransactionGuid`, and `EntityGuid` must be non-nullable UUIDs.

`SequenceNumber` and `ExpectedEventCount` must be integers suitable for contiguous group ordering.

Do not use a zero-GUID database default.

### TblEventDelivery

Required concepts:

```csharp
public long Id { get; set; }
public long EventSequence { get; set; }
public Guid TenantGuid { get; set; }
public string SourceClientId { get; set; } = null!;
public string DestinationClientId { get; set; } = null!;
public string SubscriberId { get; set; } = null!;
public Guid TransactionGuid { get; set; }
public int SequenceNumber { get; set; }
public int ExpectedEventCount { get; set; }
public string EntityType { get; set; } = null!;
public Guid EntityGuid { get; set; }
public string Operation { get; set; } = null!;
public string PayloadJson { get; set; } = null!;
public byte Status { get; set; }
public int AttemptCount { get; set; }
public int RetryCount { get; set; }
public DateTime CreatedAt { get; set; }
public DateTime OccurredAt { get; set; }
public DateTime? NextAttemptAt { get; set; }
public DateTime? DeliveredAt { get; set; }
public DateTime? AcknowledgedAt { get; set; }
public DateTime LastStatusChange { get; set; }
```

Preserve existing EmployeeGuid/CustomerGuid fields only when current routing still requires them.

### PostgreSQL concurrency decision

The current `byte[] RowVersion` + `IsRowVersion()` contract must not survive as a false SQL Server rowversion assumption.

Choose and prove exactly one:

```text
C1. No optimistic concurrency field is required for TblEventDelivery; remove the false rowversion field/mapping.
C2. PostgreSQL xmin concurrency is required; use the correct Npgsql-compatible uint/xmin mapping.
C3. An explicit application-managed version column is required; add and increment it deliberately.
```

Do not retain a `bytea` pseudo-rowversion.

Do not change unrelated entities' concurrency strategy.

### TblEventDeliveryGroupAck — new table

Create a real entity/table for complete destination group acknowledgement:

```csharp
public sealed class TblEventDeliveryGroupAck
{
    public long Id { get; set; }
    public Guid TenantGuid { get; set; }
    public string DestinationClientId { get; set; } = null!;
    public Guid TransactionGuid { get; set; }
    public int ExpectedEventCount { get; set; }
    public DateTime AcknowledgedAt { get; set; }
    public string AcknowledgedBy { get; set; } = null!;
}
```

Add a DbSet and EF mapping in ExternalDbContext.

This task creates schema only. Do not add the ACK endpoint/service yet.

## Phase 5 — PostgreSQL-native EF mappings

Update the real mappings in ExternalDbContext.

For the touched transaction-group entities, active mappings must contain:

```text
no newid()
no sysutcdatetime()
no SQL Server rowversion byte[] mapping
no UseSqlServer fallback
```

Use the established API PostgreSQL timestamp policy.

Do not silently mix timestamp with time zone and timestamp without time zone contracts. Document the selected physical types and DateTime policy.

### TblEventLog constraints and indexes

Required checks:

```text
SequenceNumber >= 1
ExpectedEventCount >= 1
SequenceNumber <= ExpectedEventCount
```

Required unique indexes:

```text
TenantGuid + SourceClientId + TransactionGuid + SequenceNumber
TenantGuid + EventGuid
```

Retain useful existing publication indexes when valid.

### TblEventDelivery constraints and indexes

Required checks:

```text
SequenceNumber >= 1
ExpectedEventCount >= 1
SequenceNumber <= ExpectedEventCount
```

Required unique index:

```text
EventSequence + DestinationClientId
```

Required pull/retry indexes:

```text
TenantGuid + DestinationClientId + Status + CreatedAt + TransactionGuid + SequenceNumber
Status + NextAttemptAt + AttemptCount + CreatedAt
```

A PostgreSQL partial index for pending/retry states is acceptable when generated DDL is valid and physically proven.

Preserve the FK:

```text
TblEventDelivery.EventSequence -> TblEventLog.EventSequence
```

### TblEventDeliveryGroupAck constraints and indexes

Required check:

```text
ExpectedEventCount >= 1
```

Required unique index:

```text
TenantGuid + DestinationClientId + TransactionGuid
```

Use explicit maximum lengths for client and actor IDs.

## Phase 6 — Establish or extend the API PostgreSQL migration chain

Create a real attached EF Core migration for ExternalDbContext.

Choose based on the Phase 1 audit:

```text
M1. Extend a proven existing PostgreSQL migration chain.
M2. Create a clean dedicated initial PostgreSQL baseline because no valid chain exists and the dev DB is disposable.
```

For M2, use a semantic migration name such as:

```text
InitialApiPostgreSqlV001
```

For M1, use a semantic name such as:

```text
AddTransactionGroupEventDeliveryV001
```

Required characteristics:

```text
attached to ExternalDbContext
real model snapshot
Npgsql annotations only
applies from an empty PostgreSQL DB through the full selected chain
creates the final event/delivery/group-ack schema
no backfill SQL for old development rows
no zero-GUID compatibility defaults
no SQL Server default expressions
no unattached SQL-only artifact
```

If the existing API chain contains unrelated but valid schema, retain it.

Do not delete historical migration source blindly.

Document every migration-chain decision.

## Phase 7 — Add a canonical API migration executor boundary

Identify the authorized startup/deployment boundary for applying API migrations.

Implement the smallest reusable service or command that can execute:

```csharp
await db.Database.MigrateAsync(cancellationToken);
```

Requirements:

```text
not run against the operator's current API DB in this task
explicit connection/database selection
Npgsql only
protected DB-name rejection
safe/redacted logging
returns applied and pending migration identifiers
not hidden inside ordinary request handling
not coupled to PlatformAppV0 Phase 1 WPF installation behavior
```

Do not use EnsureCreated as the canonical migration mechanism.

## Phase 8 — Build and design-time proof

Run with actual project paths:

```text
dotnet build <ApiServer project>
dotnet ef dbcontext info --context ExternalDbContext
dotnet ef migrations list --context ExternalDbContext
```

The commands must not start the normal API host or require placeholder startup credentials.

Expected:

```text
provider = Npgsql.EntityFrameworkCore.PostgreSQL
ExternalDbContext model validates
TblBookingConsoleHandoffCode contract is valid and documented
migration list succeeds
selected API PostgreSQL migration chain appears exactly once
no current operator DB access
```

## Phase 9 — Disposable PostgreSQL migration-from-zero proof

Create one separately named versioned disposable PostgreSQL API database.

Do not use:

```text
recovery_api_day16_pg
obm_api_dev_v0_pg
any current/protected/production API database
```

Required proof:

```text
DB did not exist before test
create empty UTF8 PostgreSQL DB
apply ExternalDbContext migrations from zero
pending migrations = 0
__EFMigrationsHistory contains the selected migration chain exactly once
no manual table creation before migration
```

Inspect information_schema and pg_catalog to prove:

### TblEventLog

```text
ExpectedEventCount NOT NULL
SequenceNumber integer NOT NULL
TransactionGuid uuid NOT NULL
EventGuid uuid NOT NULL
EntityGuid uuid NOT NULL
CreatedAt present
three sequence/count check constraints
source-group sequence unique index
TenantGuid + EventGuid unique index
```

### TblEventDelivery

```text
DestinationClientId NOT NULL
TransactionGuid uuid NOT NULL
SequenceNumber integer NOT NULL
ExpectedEventCount integer NOT NULL
EntityGuid uuid NOT NULL
CreatedAt present
AcknowledgedAt present
three sequence/count check constraints
EventSequence + DestinationClientId unique index
complete-group pull index
retry index
valid FK to TblEventLog
no bytea SQL Server-style rowversion unless a separately justified non-rowversion payload field exists
```

### TblEventDeliveryGroupAck

```text
table exists
ExpectedEventCount NOT NULL
positive expected-count check
TenantGuid + DestinationClientId + TransactionGuid unique index
```

## Phase 10 — Physical constraint/index tests

Using sanitized generated identities, prove PostgreSQL rejects:

```text
TblEventLog SequenceNumber = 0
TblEventLog ExpectedEventCount = 0
TblEventLog SequenceNumber > ExpectedEventCount
duplicate source group sequence
duplicate TenantGuid + EventGuid

TblEventDelivery SequenceNumber = 0
TblEventDelivery ExpectedEventCount = 0
TblEventDelivery SequenceNumber > ExpectedEventCount
duplicate EventSequence + DestinationClientId
orphan EventSequence FK

TblEventDeliveryGroupAck ExpectedEventCount = 0
duplicate TenantGuid + DestinationClientId + TransactionGuid
```

Prove valid rows succeed:

```text
one two-event source group with sequences 1 and 2
one destination delivery row for each source event
one complete group ACK row
```

This is schema proof only. Do not claim runtime upload, delivery, SignalR, pull, apply, or ACK behavior.

After evidence capture, drop only the disposable API proof DB.

The operator's current API DB must remain untouched.

## Phase 11 — Focused tests

Add focused tests for:

```text
design-time factory uses Npgsql
missing migration connection fails closed
protected API DB name is rejected
ExternalDbContext model validates
TblBookingConsoleHandoffCode mapping contract is correct
TblEventLog group properties/constraints/indexes exist
TblEventDelivery group properties/constraints/indexes exist
TblEventDeliveryGroupAck model/table/index exists
PostgreSQL concurrency decision is correctly mapped
migration applies from empty PostgreSQL DB
invalid rows are rejected
valid two-event group/deliveries/ACK rows succeed
pending migrations = 0
```

No soft skip is allowed when disposable PostgreSQL prerequisites are available.

Do not count build success as migration proof.

## Required private evidence artifact

Create a new versioned local artifact. Never overwrite earlier evidence.

Suggested folder:

```text
E:\Project2026\RecoveryReports\ApiPostgreSqlTransactionGroupSchemaV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
CONTEXT_MIGRATION_AUDIT.md
DESIGN_TIME_FACTORY.md
BOOKING_CONSOLE_HANDOFF_KEY_CONTRACT.md
EVENT_LOG_MODEL.md
EVENT_DELIVERY_MODEL.md
GROUP_ACK_MODEL.md
EF_MAPPING.md
CONCURRENCY_DECISION.md
MIGRATION_CHAIN.md
MIGRATION_CODE.md
MODEL_SNAPSHOT.md
GENERATED_SQL.sql
DISPOSABLE_DB_PROOF.md
CONSTRAINT_PROOF.md
INDEX_PROOF.md
TEST_OUTPUT.txt
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

## Mandatory private code/migration handoff

Return complete actual code, not excerpts, for:

```text
IDesignTimeDbContextFactory<ExternalDbContext>
TblBookingConsoleHandoffCode entity and mapping decision
TblEventLog entity
TblEventDelivery entity
TblEventDeliveryGroupAck entity
DbSet additions
all three EF mapping blocks
PostgreSQL concurrency mapping
migration executor service/command
migration Up
migration Down
model snapshot sections for all touched entities
focused test methods
```

Include:

```text
repository-relative path
line range
complete BEFORE body
complete AFTER body
actual unified diff
actual generated PostgreSQL migration SQL
```

Do not expose credentials, full connection strings, passfile contents, raw proof GUIDs, tenant/device identities, or private payload values.

## Public report

Create and push only:

```text
report/report098.md
```

Public report must be redacted and minimal.

It must include:

```text
Verdict
Design-time Npgsql factory yes/no
TblBookingConsoleHandoffCode contract resolved yes/no
API PostgreSQL migration chain yes/no
Disposable empty PostgreSQL apply yes/no
Pending migrations count
TblEventLog group schema yes/no
TblEventDelivery group schema yes/no
TblEventDeliveryGroupAck schema yes/no
Constraint proof yes/no
Index proof yes/no
PostgreSQL-only touched mappings yes/no
Focused test totals
API build errors/warnings
Current operator API DB mutated yes/no
Private evidence artifact yes/no
Aggregate SHA-256
```

Do not publish source code, schema contents, SQL, credentials, paths containing secrets, raw identifiers, or private row values.

## Source and coordination repository rules

The OBM source trees remain local/private.

Do not commit or push OBM source changes to the coordination repository.

Only commit and push the redacted public coordination report:

```text
report/report098.md
```

Preserve all dirty unrelated local source changes.

Do not reset, clean, checkout, or overwrite unrelated work.

## Scope exclusions

Must remain behaviorally unchanged:

```text
checkout/payment
invoice settlement
terminal/Dejavoo
gift cards
customer/booking
BookingConsole runtime behavior
WPF installation Phase 1
WPF migration baseline from prompt097
Price Rule UI and Save runtime
Employee Weight
Weird Tip
TurnEngine calculations
existing current operator databases
```

## Final verdicts

PASS only when all physical schema/migration gates pass:

```text
OBM_API_POSTGRESQL_TRANSACTION_GROUP_SCHEMA_READY_FOR_LOCAL_SENDER
```

Use a narrow blocker otherwise, for example:

```text
BLOCKED_API_BOOKING_CONSOLE_HANDOFF_KEY_CONTRACT
BLOCKED_API_MIGRATION_INFRASTRUCTURE
BLOCKED_API_POSTGRESQL_MODEL_VALIDATION
BLOCKED_API_TRANSACTION_GROUP_MIGRATION
BLOCKED_API_DISPOSABLE_DB_PROOF
BLOCKED_API_TRANSACTION_GROUP_CONSTRAINTS
```

Build success alone is not PASS.

Do not proceed to Price Rule sender/runtime implementation inside this task.
