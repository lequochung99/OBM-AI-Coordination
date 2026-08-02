# Prompt 097 — Create the canonical WPF PostgreSQL migration baseline and transaction-group outbox schema

## Operator approval and scope

The operator approves the next correction, but only for the WPF database migration boundary.

This task must establish a real, repeatable PostgreSQL migration lane for `eNailSalonDbContext` and make the transaction-group `TblLocalOutbox` schema physically real on a database created from zero.

Do not implement or modify in this task:

```text
Price Rule Save runtime
TblTurnPolicy/TblTurnAmountRule outbox staging
whole-group uploader
API event ingest
API event/delivery schema
SignalR publishing
POS pull/apply/ACK
BookingConsole runtime
checkout/payment runtime
operator's current development database
```

The next API-schema task will start only after this WPF migration lane passes.

## Authoritative clean-slate decision

The current development data is disposable.

There is no requirement to:

```text
backfill old TblLocalOutbox rows
preserve nullable EntityGuid rows
preserve incomplete transaction groups
support a mixed old/new WPF migration chain
apply this task to the operator's current DB
```

Because prompt096 proved that no active WPF migration chain exists for `eNailSalonDbContext`, create a clean dedicated PostgreSQL EF Core migration baseline for this context.

Do not create an unattached SQL file.
Do not use `EnsureCreated` as the production migration mechanism.
Do not start the WPF Application object during `dotnet ef` design-time operations.

## Proven prompt096 blocker

Prompt096 proved:

```text
Classification: WPF_MIGRATION_INFRASTRUCTURE_MISSING

dotnet ef migrations list
-> attempted application-host creation
-> Application object shutdown path
-> unable to construct eNailSalonDbContext
-> no PostgreSQL connection reached
-> no migration created
-> no source diff produced
```

The immediate WPF exception was:

```text
OBM-POS requires a configured PostgreSQL/Npgsql DbContext.
Use the application IDbContextFactory or TurnDbContextProvider.
```

The correction must therefore provide a design-time-only Npgsql DbContext factory that never invokes normal WPF startup.

## Required source evidence

Read completely before editing:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
prompt/prompt095.md
prompt/prompt096.md
report/report095.md
report/report096.md
```

Read every file in:

```text
E:\Project2026\RecoveryReports\CleanTransactionGroupSchemaV001
```

At minimum:

```text
PRIVATE_HANDOFF.md
WPF_ENTITY_MODEL.md
WPF_EF_MAPPING.md
WPF_MIGRATION.md
MIGRATION_CHAIN_DECISION.md
POSTGRESQL_MAPPING_CLEANUP.md
DISPOSABLE_DB_PROOF.md
TEST_OUTPUT.txt
UNIFIED_DIFF.patch
```

Record before code edits:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
TASK_SCOPE=WPF_MIGRATION_BASELINE_ONLY
PROVIDER=NPGSQL_ONLY
CURRENT_OPERATOR_DB_MUTATION=FORBIDDEN
LEGACY_DEV_DATA_PRESERVATION=NOT_REQUIRED
```

## Phase 1 — Audit the WPF context and migration assembly

Return exact proof for:

```text
eNailSalonDbContext namespace and assembly
WPF executable project
current EF Core version
current Npgsql EF provider version
current MigrationsAssembly setting, if any
existing migration folders for this context
existing __EFMigrationsHistory assumptions
actual PostgreSQL schema name used by the context
```

Search for all `IDesignTimeDbContextFactory<eNailSalonDbContext>` implementations.

Expected current count is zero. If one exists, prove why prompt096 did not use it before changing anything.

Audit whether another context already owns a migrations assembly. Do not merge unrelated context migrations into this WPF context.

## Phase 2 — Add a design-time PostgreSQL factory

Create a dedicated design-time factory, for example:

```text
MyData/DesignTimeENailSalonDbContextFactory.cs
```

The exact name/path may follow existing project conventions.

Requirements:

```text
implements IDesignTimeDbContextFactory<eNailSalonDbContext>
constructs DbContextOptions with UseNpgsql only
does not resolve WPF App, MainWindow, DI host, LocalPosStartupService, or runtime ProductRoot
does not call TurnDbContextProvider runtime guards
does not contain a password or connection string literal
does not fall back to SQL Server
does not target the operator's current DB by default
```

Connection resolution must be explicit and fail closed.

Use one dedicated environment variable name such as:

```text
OBM_WPF_MIGRATION_CONNECTION
```

or an already canonical equivalent if one exists.

The factory must:

```text
reject missing value
reject placeholder credentials
reject protected production/current DB names
require PostgreSQL/Npgsql
allow PGPASSFILE or existing protected credential mechanism
redact the connection string from diagnostics
```

Do not print passwords, host secrets, full connection strings, or raw identity GUIDs.

## Phase 3 — Correct the WPF transaction-group model

Update:

```text
MyData/TblLocalOutbox.cs
```

Final required transaction-group properties:

```csharp
public long OutboxId { get; set; }
public Guid TenantGuid { get; set; }
public Guid EventGuid { get; set; }
public Guid TransactionGuid { get; set; }
public string SourceClientId { get; set; } = null!;
public int SequenceNumber { get; set; }
public int ExpectedEventCount { get; set; }
public string EntityType { get; set; } = null!;
public int? TableId { get; set; }
public string? EntityId { get; set; }
public Guid EntityGuid { get; set; }
public string Operation { get; set; } = null!;
public string Payload { get; set; } = null!;
public string? Processor { get; set; }
public string? ClaimedBy { get; set; }
public DateTime? ClaimExpiresAt { get; set; }
public DateTime CreatedAt { get; set; }
public DateTime OccurredAt { get; set; }
public DateTime? NextAttemptAt { get; set; }
public DateTime? LastAttemptAt { get; set; }
public int Sent { get; set; }
public DateTime? SentAt { get; set; }
public long? ServerEventSequence { get; set; }
public int AttemptCount { get; set; }
public string? ErrorMessage { get; set; }
public DateTime? UpdatedAt { get; set; }
```

Preserve additional proven canonical properties only when they do not duplicate these concepts.

`EntityGuid` must be non-nullable in both C# and PostgreSQL.

Do not introduce a zero-GUID database default. Sender code must later supply a real entity key.

## Phase 4 — Correct the EF mapping

Update the `TblLocalOutbox` mapping in:

```text
MyData/eNailSalonDbContext.cs
```

Requirements:

```text
Npgsql/PostgreSQL mapping only
no sysutcdatetime()
no newid()
no SQL Server rowversion assumption
no UseSqlServer fallback
```

Use the project's canonical PostgreSQL timestamp policy.

The new fields must have explicit length/precision mapping where appropriate:

```text
ClaimedBy max length 100
ClaimExpiresAt timestamp without time zone or the established project equivalent
NextAttemptAt timestamp without time zone or established equivalent
SentAt timestamp without time zone or established equivalent
ExpectedEventCount integer
EntityGuid uuid NOT NULL
```

Add model-level constraints:

```text
SequenceNumber >= 1
ExpectedEventCount >= 1
SequenceNumber <= ExpectedEventCount
```

Add unique index:

```text
TenantGuid
SourceClientId
TransactionGuid
SequenceNumber
```

Add indexes supporting:

```text
oldest pending complete-group selection
retry scheduling
expired group-lease recovery
```

Use partial PostgreSQL indexes only when EF/Npgsql generates valid PostgreSQL DDL. Otherwise use migration SQL inside the attached EF migration while retaining equivalent model metadata where possible.

Do not add an index that permits duplicate sequence numbers within one source group.

## Phase 5 — Establish a clean WPF PostgreSQL migration baseline

Because no active migration chain exists and the databases will be recreated, create a new dedicated initial migration baseline for `eNailSalonDbContext` after the model changes above.

Required characteristics:

```text
one clearly named initial PostgreSQL baseline migration
one model snapshot for eNailSalonDbContext
dedicated migration namespace/folder
Npgsql annotations only
creates the complete current WPF schema from an empty PostgreSQL DB
includes the final transaction-group TblLocalOutbox shape directly
no backfill SQL
no old nullable EntityGuid compatibility branch
no SQL Server default expressions
```

Suggested semantic name:

```text
InitialWpfPostgreSqlV001
```

Use the actual timestamped EF migration identifier generated by tooling.

Do not create an `ALTER TABLE`-only migration that assumes an untracked pre-existing database. The acceptance database starts empty.

Do not delete unrelated historical migration source without explicit proof that it belongs to this same context and blocks the new dedicated chain.

If a separate migration assembly is required, configure it explicitly in both:

```text
design-time factory
canonical runtime schema/migration executor
```

The app must not silently use one model while `dotnet ef` uses another.

## Phase 6 — Add a canonical migration executor boundary

Identify where a future clean installation will apply WPF schema migrations.

Implement the smallest reusable service or command boundary that can execute:

```csharp
await db.Database.MigrateAsync(cancellationToken);
```

Requirements:

```text
not called during prompt097 against the operator's current DB
explicit connection/database selection
Npgsql only
safe logging
returns applied and pending migration identifiers
fails closed on protected DB names
separate from normal MainWindow startup unless canonical docs already authorize migration there
```

If installation Phase 2 is the authorized future call site, add the service now but do not connect it to Phase 1 and do not trigger DB work from the current installation Phase 1 flow.

Do not use `EnsureCreated` before or after `Migrate`.

## Phase 7 — Build and design-time proof

Required commands, with actual project paths:

```text
dotnet build <WPF project>
dotnet ef dbcontext info --context eNailSalonDbContext
dotnet ef migrations list --context eNailSalonDbContext
```

The commands must run without starting or shutting down the WPF Application object.

Expected:

```text
provider = Npgsql.EntityFrameworkCore.PostgreSQL
migration list succeeds
initial WPF PostgreSQL baseline appears exactly once
no placeholder credential path
no current operator DB access
```

## Phase 8 — Disposable PostgreSQL migration-from-zero proof

Create one separately named versioned disposable PostgreSQL DB.

Do not use:

```text
obm_pos_dev_v0_pg
obm_pos_v1_local_pos1_pg
enailsalon_phasee1_pos1_pg
any production/protected DB
```

Use a new safe name whose purpose is obvious, such as a versioned prompt097 proof DB.

Required proof:

```text
DB did not exist before the test
create empty UTF8 PostgreSQL DB
apply eNailSalonDbContext migrations from zero
pending migrations = 0
migration history contains the new baseline exactly once
no manual table creation before migration
```

Inspect `information_schema` and `pg_catalog` to prove `TblLocalOutbox` has:

```text
ExpectedEventCount NOT NULL
EntityGuid NOT NULL
ClaimedBy
ClaimExpiresAt
NextAttemptAt
SentAt
```

Prove constraints reject:

```text
SequenceNumber = 0
ExpectedEventCount = 0
SequenceNumber > ExpectedEventCount
EntityGuid NULL
```

Prove unique source-group sequence rejects a duplicate:

```text
same TenantGuid
same SourceClientId
same TransactionGuid
same SequenceNumber
```

Prove two rows in one valid group succeed:

```text
same TransactionGuid
ExpectedEventCount = 2
SequenceNumber = 1 and 2
non-empty EntityGuid values
```

Prove indexes exist by exact name and column order.

After evidence capture, drop only the disposable proof DB.

The operator's current DB must remain untouched.

## Phase 9 — Focused tests

Add focused tests for:

```text
design-time factory uses Npgsql
missing migration connection fails closed
protected DB name is rejected
model contains ExpectedEventCount
model EntityGuid is required
model has all three check constraints
model has source-group unique index
migration applies from empty PostgreSQL DB
invalid rows are rejected by PostgreSQL
valid two-event group succeeds
pending migrations = 0
```

No soft skip is permitted when the disposable DB prerequisites are available.

Do not count build success as migration proof.

## Required private evidence artifact

Create a new versioned local artifact. Never overwrite prompt096 evidence.

Suggested folder:

```text
E:\Project2026\RecoveryReports\WpfPostgreSqlMigrationBaselineV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
DESIGN_TIME_FACTORY.md
ENTITY_MODEL.md
EF_MAPPING.md
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

## Mandatory private C# and migration handoff

Return complete actual code, not excerpts, for:

```text
IDesignTimeDbContextFactory implementation
TblLocalOutbox entity
TblLocalOutbox EF mapping block
migration executor service/command
initial migration Up
initial migration Down
model snapshot sections for TblLocalOutbox
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

Do not expose credentials, full connection strings, raw proof GUIDs, or private database contents.

## Public report

Create and push only:

```text
report/report097.md
```

Public report must remain redacted and minimal.

Required fields:

```text
Verdict
Design-time Npgsql factory: yes/no
Dedicated WPF migration baseline: yes/no
Migration list proof: yes/no
Disposable empty PostgreSQL apply: yes/no
Pending migrations: count
TblLocalOutbox grouped schema: yes/no
Constraint proof: yes/no
Index proof: yes/no
PostgreSQL-only touched mappings: yes/no
Focused tests: passed/failed/skipped
WPF build: warnings/errors
Current operator DB mutated: yes/no
Private evidence artifact: yes/no
Aggregate SHA-256
```

## PASS verdict

Use exactly:

```text
OBM_WPF_POSTGRESQL_BASELINE_AND_OUTBOX_SCHEMA_READY_FOR_API_SCHEMA
```

Only when all are true:

```text
design-time factory works without WPF startup
initial migration and snapshot exist
migration list succeeds
empty disposable PostgreSQL DB is migrated from zero
pending migrations = 0
TblLocalOutbox final columns are physically present
all four negative constraint tests pass
unique group-sequence rejection passes
valid two-event group insertion passes
required indexes physically exist
no active SQL Server default remains in touched WPF mapping
focused tests pass with zero skip
operator's current DB is untouched
```

Otherwise use one narrow blocker, for example:

```text
BLOCKED_WPF_DESIGN_TIME_FACTORY
BLOCKED_WPF_INITIAL_POSTGRESQL_MIGRATION
BLOCKED_WPF_BASELINE_MODEL_VALIDATION
BLOCKED_WPF_POSTGRESQL_DDL
BLOCKED_WPF_DISPOSABLE_DB_PROOF
BLOCKED_WPF_OUTBOX_CONSTRAINT_PROOF
```

Do not claim sender, uploader, API or POS2 sync readiness in this task.

## Git coordination

Before work:

```text
pull origin/main in OBM-AI-Coordination
```

After work:

```text
add report/report097.md only to the coordination repository
commit
push origin/main
```

Do not commit or push the private OBM source repositories unless separately authorized.
