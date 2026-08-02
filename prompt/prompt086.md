# Prompt 086 — Make OBM-POS PostgreSQL-only, remove all active SQL Server legacy, and repair Price Weight

## Operator decision — authoritative architecture

OBM-POS previously used SQL Server, but the current product database is PostgreSQL.

The operator has now decided:

```text
OBM-POS active product code must be PostgreSQL-only.
Remove all active SQL Server provider code, packages, configuration, fallback branches, and tests.
Compile failures caused by removing obsolete SQL Server code are acceptable temporarily, but must be repaired correctly for PostgreSQL before completion.
```

The operator explicitly prefers exposing and fixing hidden legacy dependencies now rather than leaving dormant SQL Server paths that may fail later.

## Supersedes prompt085

This prompt supersedes `prompt/prompt085.md`.

Do not execute prompt085 separately. Its Price / Amount Rule load objective is incorporated here after provider cleanup.

## Physical evidence

`Employee Turn Settings -> Price Weight -> Edit` opens `Price / Amount Rule Settings`, but Load fails with a wrapper message suggesting:

```text
EnableRetryOnFailure
UseSqlServer
```

Do not treat that message as a valid fix. It may expose an old SQL Server execution strategy, wrong-provider context factory, package dependency, generic wrapper, or provider-specific query/schema defect.

## Scope boundary

Audit and correct:

```text
<WPF_ROOT>
<WPF_TEST_ROOT>
all projects directly referenced by the WPF solution/build graph
all runtime configuration used by WPF
all local database factories/repositories/migrations/helpers used by WPF
```

Do not modify unrelated ApiServer/Platform projects unless the WPF build directly references a shared project containing active SQL Server code. Record any out-of-scope SQL Server findings separately.

Do not mutate the operator's current database automatically.
Do not test checkout/payment.
Do not commit or push OBM source.

## Mandatory documentation gate

Read completely before edits:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report084.md
```

Read existing Price Weight/TurnEngine local evidence and the current prompt085 coordination text.

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
ARCHITECTURE_DECISION=POSTGRESQL_ONLY
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

Update canonical documentation only if required to state that SQL Server is no longer a supported runtime/test provider for OBM-POS.

## Phase A — complete SQL Server inventory before deletion

Search source-controlled files, excluding only generated `bin`, `obj`, `.git`, package cache, and binary artifacts.

Search case-insensitively for at least:

```text
UseSqlServer
Microsoft.EntityFrameworkCore.SqlServer
Microsoft.Data.SqlClient
System.Data.SqlClient
SqlConnection
SqlCommand
SqlParameter
SqlBulkCopy
SqlException
SqlDataReader
SqlTransaction
SqlDependency
SqlServer
MSSQL
LocalDB
SQLEXPRESS
Initial Catalog
Integrated Security
Trusted_Connection
MultipleActiveResultSets
Encrypt=
TrustServerCertificate
EnableRetryOnFailure
Server=.
Data Source=
providerName=System.Data.SqlClient
Relational:ProviderName
Microsoft.EntityFrameworkCore.SqlServer
```

Also inspect provider-specific SQL/syntax in active C#/SQL/resources:

```text
[dbo]
GETDATE()
NEWID()
IDENTITY
SCOPE_IDENTITY
TOP (
TOP 
ISNULL(
LEN(
DATEDIFF
DATEADD
MERGE
OUTPUT INSERTED
WITH (NOLOCK)
TRY_CONVERT
TRY_CAST
nvarchar
uniqueidentifier
bit
money
GO batch separators
```

Do not blindly rewrite text found in user-facing prose or historical audit files. Classify every match:

| Match | File/project | Active build/runtime | Test-only | Historical doc/archive | Delete | Convert to PostgreSQL | Keep with reason |
|---|---|---:|---:|---:|---:|---:|---|

Required result:

```text
active runtime SQL Server references = 0
active build-graph SQL Server package references = 0
active test SQL Server provider references = 0
active configuration SQL Server connection options = 0
```

Historical documentation may remain only when clearly labeled archival and excluded from build/runtime. Prefer moving it to an explicit legacy archive or deleting it when it has no audit value. No active code may reference it.

## Phase B — remove SQL Server packages and provider selection

Audit every `.csproj`, `Directory.Packages.props`, `packages.lock.json`, package manifest, and transitive package source.

Remove direct active references to at least:

```text
Microsoft.EntityFrameworkCore.SqlServer
Microsoft.Data.SqlClient
System.Data.SqlClient
SQL Server migration/design packages used only by the old provider
```

Do not remove shared EF Core packages required by Npgsql.

Canonical provider packages should be PostgreSQL/Npgsql only, for example the already-established versions of:

```text
Npgsql
Npgsql.EntityFrameworkCore.PostgreSQL
Microsoft.EntityFrameworkCore
Microsoft.EntityFrameworkCore.Relational
Microsoft.EntityFrameworkCore.Design where genuinely required
```

Run dependency inspection after restore and explain any remaining transitive SQL Server package. A transitive package is acceptable only if it is unavoidable, never loaded by the product, and has no SQL Server runtime path; otherwise replace/remove its parent dependency.

## Phase C — one canonical PostgreSQL DbContext/factory path

Inventory every DbContext constructor, factory, design-time factory, DI registration, service locator, static factory, repository-created context, test factory, and fallback connection path.

Final architecture:

```text
runtime local DB
-> one canonical PostgreSQL connection/settings resolver
-> UseNpgsql only
-> Npgsql provider name asserted
-> no provider switch
-> no SQL Server fallback
-> no automatic provider inference from connection-string shape
```

Remove patterns such as:

```csharp
if (provider == "SqlServer") UseSqlServer(...);
else UseNpgsql(...);
```

```csharp
try PostgreSQL; catch { use SQL Server; }
```

```csharp
if connection string contains ... select provider
```

A missing/invalid PostgreSQL configuration must fail with a clear PostgreSQL configuration result, not fall back to SQL Server.

Audit `App.AppHost`, dependency injection, WPF design-time services, migrations, import tools, background services, setup screens, and MiniEmployeeTurnApp/Turn settings for independently constructed contexts.

Add a startup/build-time or focused-test guard proving:

```text
ctx.Database.ProviderName == Npgsql.EntityFrameworkCore.PostgreSQL
```

for all production context factories.

## Phase D — remove SQL Server configuration and secrets contract

Inspect source-controlled configuration and configuration readers:

```text
appsettings*.json
*.config
launchSettings.json
user-secret key names referenced by source
PowerShell/batch scripts
installer/runtime profile files
DB locator/settings classes
environment-variable names
```

Remove SQL Server-specific keys, provider selectors, connection templates, sample connection strings, LocalDB defaults, and old retry configuration from the active WPF contract.

Do not print or inspect secret values. Inventory key names only.

Canonical WPF runtime must accept only the established protected PostgreSQL settings/credential flow.

A stale SQL Server environment variable or config file must not be silently selected.

## Phase E — provider-specific code conversion/removal

For each active SQL Server API or SQL fragment:

1. Decide whether it is dead legacy and delete it.
2. If functionality remains required, implement the PostgreSQL/Npgsql equivalent using the existing architecture.
3. Add focused tests for the converted path.

Examples:

```text
SqlConnection -> NpgsqlConnection only when raw ADO.NET is genuinely needed
SQL Server bulk copy -> PostgreSQL COPY or existing EF path
GETDATE() -> proven PostgreSQL timestamp contract or application timestamp helper
NEWID() -> application Guid or gen_random_uuid only when schema contract permits
TOP -> LIMIT
ISNULL -> COALESCE
SQL Server parameter syntax/types -> Npgsql/portable EF expressions
```

Prefer provider-neutral EF LINQ where it translates correctly to PostgreSQL. Do not introduce raw SQL unnecessarily.

Respect the established `timestamp without time zone` handling and helpers such as the proven local timestamp normalization. Do not reintroduce UTC `DateTime` mismatches.

## Phase F — migration/schema policy

Audit all migrations/schema helpers reachable by the active WPF build/runtime.

Final rules:

```text
PostgreSQL schema/migrations only
no SQL Server migration assembly selection
no SQL Server EnsureCreated fallback
no SQL Server DDL generator
no dual-provider migration branch
```

Do not automatically apply migrations or DDL to the operator's current database.

Old SQL Server migration files may be deleted from active source when no longer needed. If audit history requires preservation, move them into a clearly non-built/non-runtime archive and ensure no project includes or references them.

Do not convert historical SQL Server migrations mechanically into PostgreSQL migrations. Use the current canonical PostgreSQL schema/migration source of truth.

## Phase G — test infrastructure conversion

Remove SQL Server/LocalDB-dependent tests and fixtures from the active test suite.

Convert meaningful integration coverage to the approved disposable PostgreSQL/Npgsql harness.

Do not replace provider-sensitive tests with EF InMemory and claim parity.

Required tests include:

```text
all production context factories use Npgsql
no UseSqlServer symbol in active compiled source
no SQL Server package in active direct dependencies
no SQL Server connection/config fallback
invalid PostgreSQL config fails explicitly
approved disposable PostgreSQL database can be created/loaded/dropped
provider-specific timestamp, decimal, boolean, GUID, and nullable mapping contracts pass
```

Add a repository guard test/script that fails when prohibited SQL Server tokens reappear in active source/project/config files. It must exclude explicit archival directories and generated output only.

## Phase H — repair Price / Amount Rule Settings after provider purge

After completing the provider cleanup, return to the physical Price Weight defect.

Trace:

```text
Employee Turn Settings
-> Price Weight Edit
-> Price / Amount Rule Settings
-> tenant resolution
-> rule owner/policy context
-> canonical Npgsql DbContext factory
-> TblTurnAmountRule query
-> projection/binding
```

Obtain the exact post-cleanup inner exception if Load still fails:

```text
ExceptionType
InnerException chain
sanitized message
Npgsql/PostgreSQL SqlState when present
ProviderName
failing stage/query
stack trace
table/column/constraint when applicable
```

Do not add generic retry unless direct evidence proves a transient PostgreSQL failure and retry is appropriate for this local UI operation.

Required behavior:

```text
Price Weight checkbox checked or unchecked -> Edit opens
No Draft/No Active -> editor opens and loads
zero rules -> clear empty state, no mutation
existing rules -> tenant-safe deterministic load
Load button -> same canonical method, no duplicates
```

Audit Add Row, Delete Selected, Reset Defaults, Test Lookup, and Save sufficiently to ensure they use the same PostgreSQL context and cannot invoke deleted SQL Server code.

No automatic current-DB writes. Operator Save remains the only UI-authorized mutation.

## Phase I — build graph and zero-reference proof

Run clean restore/build/tests after deleting `bin/obj` for the WPF and test projects.

Provide direct evidence:

```text
SQLSERVER_ACTIVE_SOURCE_MATCHES=0
USESQLSERVER_MATCHES=0
SQLCLIENT_ACTIVE_MATCHES=0
SQLSERVER_DIRECT_PACKAGE_REFS=0
SQLSERVER_ACTIVE_CONFIG_KEYS=0
PRODUCTION_CONTEXT_PROVIDER=Npgsql.EntityFrameworkCore.PostgreSQL
```

Search again after the build to ensure generated files are not being mistaken for source evidence.

Build every project in the WPF solution/build graph, not only the startup project.

Run:

```text
focused PostgreSQL-only/provider guard tests
full WPF test suite when feasible
Price/Amount Rule focused tests
real disposable PostgreSQL integration tests
```

Do not hide unrelated pre-existing failures. Classify them separately.

## Required private handoff

Return directly to the operator:

1. Complete SQL Server inventory before cleanup.
2. Exact files/packages/config paths removed or converted.
3. Any archival files retained and why they cannot execute.
4. Final DbContext/factory/DI diagram.
5. Zero-reference scan commands and outputs.
6. Dependency/package output proving no active direct SQL Server package.
7. Exact Price Weight root cause after provider cleanup.
8. Before/after relevant C# blocks for the Price load path.
9. Build/test/integration commands and counts.
10. Explicit confirmation that no current DB was automatically mutated.
11. Explicit list of unrelated modules left unchanged.

Do not include passwords, connection strings, PINs, raw GUIDs, customer/employee/business rows, or private identifiers.

## Safety boundaries

- No automatic mutation of the current operator DB.
- Disposable PostgreSQL writes only through an approved isolated harness.
- No WPF automation/clicking.
- No checkout/payment test.
- No OBM source commit/push.
- No SQL Server fallback retained for convenience.
- No provider switch retained as dead code.
- No secrets printed or copied.

## Public report

Create and push only an ultra-minimal:

```text
report/report086.md
```

Allowed fields only:

```text
verdict
PostgreSQL-only active runtime proven yes/no
active SQL Server source references zero yes/no
active SQL Server direct packages zero yes/no
active SQL Server configuration zero yes/no
all production context factories Npgsql yes/no
Price Weight load corrected yes/no
build/test counts
current DB automatically mutated no
checkout tested no
one aggregate evidence SHA-256
```

Do not push file paths, source symbols, package inventory details, provider configuration, schema metadata, exceptions, or code blocks.

## Valid verdicts

```text
OBM_POS_POSTGRESQL_ONLY_AND_PRICE_WEIGHT_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_SQL_SERVER_ACTIVE_DEPENDENCY_REMAINS
```

```text
BLOCKED_POSTGRESQL_PROVIDER_CONVERSION
```

```text
BLOCKED_PRICE_WEIGHT_POSTGRESQL_LOAD
```

```text
BLOCKED_BUILD_OR_TEST
```
