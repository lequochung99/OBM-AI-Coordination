# Prompt 046 — Remove unused ASP.NET Identity legacy traces after local-DB-first runtime PASS

## Operator decision

The operator has identified a major long-term maintenance risk: unused legacy security artifacts can mislead future developers and AI agents into assuming OBM-POS requires a complex ASP.NET Identity/password architecture.

The current local POS database appears to contain the standard seven ASP.NET Identity tables even though the WPF POS does not use ASP.NET Identity for employee access:

```text
AspNetUsers
AspNetRoles
AspNetUserRoles
AspNetUserClaims
AspNetUserLogins
AspNetRoleClaims
AspNetUserTokens
```

Employee access in OBM-POS uses a simple local operational PIN (`TblEmployee.LoginNumber`) for UI gating, role checks, actor attribution, and audit/log correlation. It is not an ASP.NET Identity username/password system.

The operator wants unused legacy traces removed so future AI work does not repeatedly infer that OBM-POS needs a complex password/Identity implementation.

## Execution order and hard gate

Do not execute destructive cleanup until prompt045 local-DB-first runtime has completed and the operator has physically verified that WPF opens and works locally.

Required prerequisite verdict/evidence:

```text
OBM_POS_LOCAL_DB_FIRST_RUNTIME_READY_FOR_USER_RETEST
```

plus operator physical proof that:

```text
MainWindow opens against obm_pos_dev_v0_pg
local DB CRUD works
employee/settings UI can load
API unavailability does not block local startup
```

If this prerequisite is absent, perform only the audit/design portion and stop with:

```text
BLOCKED_LEGACY_IDENTITY_CLEANUP_RUNTIME_NOT_YET_PROVEN
```

## Scope

This prompt has two distinct phases:

```text
Phase A — read-only legacy inventory and dependency proof
Phase B — versioned cleanup implementation only when Phase A proves safe and the runtime prerequisite has passed
```

Never delete tables manually in pgAdmin.

Do not use `DROP TABLE ... CASCADE` as a shortcut.

Do not remove any active PlatformAppV0, ApiServer, Google/OIDC, JWT, WpfJwt, access-token, or future refresh-token implementation. This prompt concerns only unused ASP.NET Identity legacy artifacts inside the OBM-POS/WPF local database and source paths proven unused.

## Read first

```text
report/report044.md
report/report045.md if present
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Inspect source under:

```text
E:\Project2026\4POS\NailSalonNet8
E:\Project2026\1ApiServer\ApiServer01
```

Approved target for read-only physical inspection:

```text
obm_pos_dev_v0_pg
Environment = Development
```

Reference database may be inspected read-only when needed:

```text
enailsalon_phasee1_pos1_pg
```

Do not mutate the reference database.

## Canonical architecture statement

The resulting source and documentation must make this distinction explicit:

```text
OBM-POS local employee access
-> TblEmployee operational PIN
-> local role/UI gate and audit attribution

OBM-POS local database authentication
-> PostgreSQL username/password

OBM-POS API authentication
-> API access token + refresh token lifecycle

ASP.NET Identity tables
-> not part of the canonical WPF POS runtime unless physical source evidence proves otherwise
```

## Phase A — exact database inventory

Inspect PostgreSQL catalogs read-only and report the exact physical table names, schemas, row counts, owners, indexes, sequences, constraints, triggers, policies, grants, and foreign-key relationships for all ASP.NET Identity-shaped objects.

Expected candidate set:

```text
AspNetUsers
AspNetRoles
AspNetUserRoles
AspNetUserClaims
AspNetUserLogins
AspNetRoleClaims
AspNetUserTokens
```

Do not assume all seven exist or use the default schema. Prove exact names and schema.

For each candidate object, report safely:

```text
exists
row count
referenced by incoming FKs
references outgoing FKs
indexes/unique constraints
sequences/defaults
triggers
row-level security/policies
grants
last known migration that created it
```

Do not print password hashes, security stamps, tokens, normalized emails, personal identities, or row payloads even if rows unexpectedly exist.

### Immediate stop conditions

Stop before any cleanup if any Identity table:

```text
contains rows;
is referenced by a business table;
is read/written by current WPF runtime;
is required by an active migration path;
is used by API/Platform code against the same local database;
has an unclear owner/dependency;
is needed for rollback compatibility with a deployed version.
```

Use verdict:

```text
BLOCKED_LEGACY_ASPNET_IDENTITY_TABLES_NOT_PROVEN_UNUSED
```

## Phase A — source dependency audit

Search the complete WPF source, tests, migrations, scripts, docs, generated manifests, startup paths, and project packages for:

```text
AspNetUsers
AspNetRoles
AspNetUserRoles
AspNetUserClaims
AspNetUserLogins
AspNetRoleClaims
AspNetUserTokens
IdentityDbContext
IdentityUser
IdentityRole
AddIdentity
AddDefaultIdentity
AddIdentityCore
UserManager<
RoleManager<
SignInManager<
PasswordHasher
IPasswordHasher
Microsoft.AspNetCore.Identity
```

Classify every hit:

```text
active runtime dependency
migration-only legacy artifact
test-only artifact
comment/documentation only
dead source
API/Platform dependency outside WPF local DB
unknown
```

Do not remove API/Platform identity code merely because a similar symbol exists. Database boundary matters:

```text
WPF local POS DB != Platform/API database
```

Prove which DbContext and connection each source hit uses.

## Migration-history audit

Identify exactly how the seven tables entered the local DB:

```text
migration name/version
migration project
DbContext
creation date/order
whether later migrations reference them
whether clean install currently creates them
whether downgrade/rollback code assumes they exist
```

Determine whether the canonical fix should be:

```text
A. new forward migration drops unused tables for existing DBs and removes creation from the clean-install model;
B. new forward migration only, while historical migrations remain immutable;
C. source/model cleanup plus versioned drop migration;
D. tables retained but explicitly documented as reserved — only if physical evidence requires retention.
```

Historical migrations should normally remain immutable for auditability. Do not rewrite old migration files unless the project already follows a different proven canonical policy.

## Source and terminology cleanup inventory

Audit misleading names and traces that may cause future AI confusion, including:

```text
Password in WPF employee-window class names
Login-With-Password labels
PIN/password mixed comments
unused Identity packages
unused DbSet declarations
unused Identity service registration
unused appsettings sections
unused migrations/scripts
obsolete docs saying employee password
legacy reports that imply WPF uses ASP.NET Identity
```

Classify each as:

```text
rename now
remove now
mark obsolete/superseded
retain for compatibility with explicit canonical comment
```

Do not perform a broad cosmetic rename that risks breaking XAML, reflection, serialization, routes, or public API contracts without tests.

## Safe cleanup design

When and only when Phase A proves the candidate tables are empty and unused, implement a versioned cleanup.

### Backup/rollback anchor

Before any database mutation, create a new versioned backup anchor. Do not overwrite V007/V008 or earlier anchors.

Example:

```text
E:\Project2026\RecoveryReports\InstallationV0\LegacyIdentityCleanupV001\PreCleanupBackup
```

Requirements:

```text
pg_dump custom format succeeds
non-zero file
pg_restore --list succeeds
SHA-256 manifest
exact DB name = obm_pos_dev_v0_pg
role/tool/version evidence
```

If backup fails, stop:

```text
BLOCKED_LEGACY_IDENTITY_CLEANUP_BACKUP_INVALID
```

### Forward migration

Use a new versioned migration in the canonical WPF/local DB migration path.

The migration must:

```text
verify each target table is empty;
verify no incoming business FKs;
verify expected table/constraint ownership;
drop child tables before parent tables;
drop only exact proven Identity indexes/constraints/sequences;
avoid CASCADE unless every cascaded object is enumerated and approved;
record the migration/schema version normally;
run transactionally where PostgreSQL permits;
fail closed on unexpected state.
```

Likely dependency order, to verify physically:

```text
AspNetUserTokens
AspNetUserLogins
AspNetUserClaims
AspNetUserRoles
AspNetRoleClaims
AspNetUsers
AspNetRoles
```

Do not trust this order without catalog proof.

### Clean-install model

Ensure a fresh empty database followed by canonical migrations no longer ends with unused ASP.NET Identity tables.

Do not simply drop them at the end while continuing to register an Identity model that recreates or expects them.

Remove only proven-unused local-WPF Identity model/package/config references.

## Canonical anti-confusion documentation

Create a versioned canonical document under an appropriate WPF/Installation documentation path, for example:

```text
E:\Project2026\CanonicalInstallationDocs\OBM-POS-RUNTIME-AUTHENTICATION-BOUNDARIES-V001.md
```

It must state clearly:

```text
1. Employee LoginNumber is an operational PIN, not ASP.NET Identity authentication.
2. PostgreSQL username/password authenticates WPF to the local DB.
3. API access/refresh tokens authenticate WPF to API services.
4. Platform Google/admin authentication is separate and not stored in WPF local Identity tables.
5. ASP.NET Identity tables are not part of canonical OBM-POS local runtime after cleanup.
6. Installation markers/runtime profile are state proofs, not user-password systems.
```

Also add a short source-level architecture note near the local DbContext/migration entry point so future AI/code review sees the boundary before inferring a security architecture.

Do not put secrets or raw PINs in documentation.

## Runtime acceptance after cleanup

Required physical/regression checks:

```text
clean existing target upgrade applies successfully;
all seven proven-unused tables are absent;
MainWindow opens local-DB-first;
local CRUD works;
employee management loads;
operational PIN paths still compile and behave unchanged;
API offline still does not block MainWindow;
no seed rerun;
no outbox/marker/runtime-history delta caused by cleanup startup;
restart repeats healthy path.
```

The cleanup migration itself may change schema-version/migration-history rows as expected. It must not change business data, employee PINs, API tokens, outbox rows, installation markers, runtime profile identity/state, or runtime history.

## Tests

Add focused tests for:

```text
current WPF runtime has no active ASP.NET Identity dependency;
local DbContext no longer maps unused Identity entities;
clean migrations do not leave candidate tables;
upgrade migration refuses non-empty Identity tables;
upgrade migration refuses unexpected FK dependencies;
upgrade migration drops exact empty tables in dependency order;
employee operational PIN paths remain independent of Identity;
PostgreSQL credential path remains unchanged;
API token path remains unchanged;
MainWindow local-first readiness remains unchanged;
no business/outbox/marker/runtime mutation;
canonical documentation exists and uses corrected terminology.
```

Run relevant builds/tests discovered in source, including at minimum:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~Migration|FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Employee"
```

Use the exact focused classes if the combined filter is unsuitable.

## Mutation policy

Phase A is read-only.

Phase B database cleanup is allowed only when:

```text
prompt045/runtime prerequisite physically passed;
all candidate tables are empty;
all dependencies are proven unused;
backup anchor is valid;
source migration and rollback path are implemented;
focused tests pass.
```

Do not mutate the reference DB.

Do not modify Platform/API databases.

Do not print secrets, DB passwords, Identity password hashes, tokens, raw employee PINs, emails, or private identities.

## Report 046

Create and push:

```text
report/report046.md
```

Required sections:

1. Verdict.
2. Runtime prerequisite evidence.
3. Exact candidate Identity-table inventory.
4. Row-count and dependency proof.
5. Source/package/DbContext dependency audit.
6. Migration-history origin.
7. Active versus dead legacy trace classification.
8. Cleanup safe/blocked decision.
9. Backup anchor evidence if mutation allowed.
10. Forward migration design and exact dropped objects.
11. Clean-install model correction.
12. Source terminology/package/docs cleanup.
13. Canonical authentication-boundary document.
14. Build/test counts.
15. Physical upgrade/clean-install evidence if executed.
16. No business/outbox/marker/runtime/PIN/token mutation proof.
17. Remaining legacy items and explicit rationale.
18. Source files changed.
19. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_LEGACY_ASPNET_IDENTITY_CLEANUP_READY_FOR_USER_RETEST
```

```text
OBM_POS_LEGACY_ASPNET_IDENTITY_CLEANUP_PHYSICAL_PASS
```

```text
BLOCKED_LEGACY_IDENTITY_CLEANUP_RUNTIME_NOT_YET_PROVEN
```

```text
BLOCKED_LEGACY_ASPNET_IDENTITY_TABLES_NOT_PROVEN_UNUSED
```

```text
BLOCKED_LEGACY_IDENTITY_CLEANUP_BACKUP_INVALID
```
