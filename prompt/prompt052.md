# Prompt 052 — Delete misleading startup services; keep one local-DB startup path and one API path

## Operator correction — simplify by deletion, not by another layer

The recurring startup failures are being amplified by misleading service names and legacy abstractions. Names such as `BootstrapRepairRequired`, `ApplicationStartupCoordinator`, `DatabaseStartupAssessment`, `RuntimeProfileStartupAssessmentService`, launch-provenance services, and API bootstrap terms make developers and AI agents treat unrelated concepts as startup security requirements.

The correct OBM-POS behavior is simple:

```text
Local PostgreSQL is usable
-> open MainWindow

API token valid
-> API/sync online

API token expired, HTTP 401, or API unavailable
-> MainWindow still opens
-> local CRUD continues
-> API/sync is Offline or Reauthorization Required
```

Employee `LoginNumber` is an operational PIN. It is not an application password and is not a startup credential.

Do not add another router, precedence engine, context object, wrapper, compatibility adapter, or hidden flag. Prefer deletion and direct code.

## Read first

```text
report/report044.md
report/report045.md
report/report049.md
report/report050.md
report/report051.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

## Physical state to preserve

Current physical evidence already proves:

```text
Database = obm_pos_dev_v0_pg
LocalDatabaseConfigResolved=True
LocalDatabaseAuthenticationSucceeded=True
SchemaReady=True
RuntimeState=Activated
TenantIdentityConsistent=True
PosIdentityConsistent=True
LocalPosReady=True
API status=WPF_HELLO_HTTP_401 / Reauthorization Required
```

For that state, the only valid runtime outcome is:

```text
OPEN_POS_MAINWINDOW_SHOWN
```

## Canonical final architecture

Normal runtime should have no more than these responsibilities:

### 1. `LocalPosStartupService`

One service/method owns normal startup:

```text
- resolve ProductRoot/config;
- load PostgreSQL host/port/database/username/password;
- authenticate to PostgreSQL;
- verify essential local schema;
- resolve local Tenant/POS context;
- show MainWindow.
```

Preferred result type:

```text
LocalPosStartupResult
```

with only practical outcomes:

```text
Ready
InstallationRequired
RecoveryRequired
Failed
```

### 2. `ApiSessionService` or existing equivalent

Runs only after MainWindow is visible:

```text
- token accepted -> Online;
- token expired/401 -> ReauthorizationRequired;
- API unavailable -> Offline;
- never changes the local startup result.
```

### 3. `InstallationV0VerificationService`

Installation/audit only. It may verify Phase 1, Phase 2, markers, seed counts, and identity-spine evidence. It must not be called by ordinary runtime startup.

Do not create these three services if existing classes can be directly renamed/merged. The goal is fewer active classes and fewer decisions, not new wrappers.

## Mandatory semantic cleanup

Audit all active callers, then delete or rename misleading startup types.

### Delete from the normal runtime path

Remove active runtime dependencies on:

```text
BootstrapRepairRequired
POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED
RepairBootstrap
NeedsBootstrapRepair
LaunchProvenanceContext
EffectiveProductRootContext as authorization
InstallationV0CompletedReadinessService in normal startup
Phase1 checkpoint/token checks before MainWindow
API protected-hello checks before MainWindow
exact employee/permission/outbox counts before MainWindow
```

If a type becomes unused after caller cleanup, delete the source file, registration, tests, enum member, result code, comments, and documentation reference. Do not retain a compatibility shim without a proven external caller.

### Rename or merge ambiguous active types

Audit these exact names and simplify them when they remain active:

```text
ApplicationStartupCoordinator
RuntimeProfileStartupAssessmentService
DatabaseStartupAssessment
DatabaseStartupMode
InstalledHealthy
AppJwtBootstrapper
InstallationV0CompletedReadinessService
```

Preferred direction:

```text
ApplicationStartupCoordinator
+ RuntimeProfileStartupAssessmentService
+ normal-runtime portions of DatabaseStartupAssessment
-> merge into LocalPosStartupService

DatabaseStartupAssessment / DatabaseStartupMode
-> LocalPosStartupResult / LocalPosStartupDecision

AppJwtBootstrapper
-> ApiSessionInitializer or ApiSessionService

InstallationV0CompletedReadinessService
-> InstallationV0VerificationService
-> installer/diagnostics callers only
```

Use the word `Bootstrap` only when the exact domain is explicit:

```text
LocalDatabaseConfiguration
InstallationBootstrapToken
ApiSession
```

Do not leave the generic word `Bootstrap` in a normal-runtime result code.

## Runtime code must become obvious

The final normal startup should be understandable in a few lines:

```csharp
var local = await localPosStartupService.StartAsync();

if (local.IsReady)
{
    await ShowMainWindowAsync();
    _ = apiSessionService.StartAsync();
    return;
}

ShowInstallationOrRecovery(local);
```

No API call may occur before the local `ShowMainWindowAsync()` decision.

No Phase 1/Phase 2 detailed proof may be required before ordinary MainWindow startup.

## PostgreSQL runtime credential

Prove safely that normal runtime uses:

```text
Database = obm_pos_dev_v0_pg
Username = hung
Password source = protected local PostgreSQL credential
```

Do not print the password or connection string.

If normal runtime still uses `postgres`, correct configuration/source selection to use `hung`. Do not create users, change passwords, alter roles, or issue GRANT/REVOKE in this prompt.

## Open OBM-POS button

`InstallationV0 -> Open OBM-POS` must call the exact same local startup method used by direct `NailSalonNet8` startup.

It must not call:

```text
protected hello
WpfJwt validation
Pairing Code logic
Phase1 current-token proof
API bootstrap repair
InstallationV0 detailed verification again
```

For the current DB:

```text
click once
-> PostgreSQL usable
-> MainWindow visible
-> InstallationV0 closes
-> API remains ReauthorizationRequired
```

## Source/dependency cleanup procedure

For each candidate service/type:

```text
1. Find all references, DI registrations, factories, tests, reflection/XAML references.
2. Identify whether it belongs to local runtime, API session, installation only, or is dead.
3. Merge/rename only when needed.
4. Delete dead files and registrations.
5. Do not leave obsolete aliases or adapters.
6. Build and run focused tests after deletion.
```

The report must include a deletion/rename table:

| Old name | Active purpose before | Final action | Final name or replacement |
| --- | --- | --- | --- |

## Do not touch these deferred items

Do not in this prompt:

```text
- implement refresh tokens;
- change employee PIN values or 4/6-digit rules;
- drop ASP.NET Identity tables;
- migrate outbox rows;
- mutate PostgreSQL;
- rerun seed;
- rewrite markers;
- change runtime state/history;
- redeem Pairing Code;
- set User/Machine environment variables.
```

Legacy ASP.NET Identity table cleanup remains a separate step after MainWindow and local CRUD physically pass.

## Tests

Add focused tests proving:

```text
local DB usable as hung + API HTTP 401 -> MainWindow shown
local DB usable + API unavailable -> MainWindow shown
local DB usable + Pairing Code absent -> MainWindow shown
API failure does not change LocalPosStartupResult.Ready
Open OBM-POS and direct runtime call the same local startup method
no active normal-runtime call to InstallationV0VerificationService
no active normal-runtime result named BootstrapRepairRequired
no active bool wrapper in the MainWindow handoff
local DB missing/auth failed/schema missing -> installation or recovery
normal runtime uses configured role hung, not postgres
prompt052 label
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~OpenObmPos|FullyQualifiedName~MainWindow|FullyQualifiedName~Offline|FullyQualifiedName~Database" -v minimal
```

## Build label

Set:

```text
Build label: prompt052
Window title: OBM InstallationV0 Phase 1/2 - prompt052
```

## Report 052

Create and push:

```text
report/report052.md
```

Required sections:

1. Verdict.
2. Exact misleading names that caused the API/local-DB confusion.
3. Active caller inventory before cleanup.
4. Deleted services/files/result codes/DI registrations.
5. Renamed or merged services/types.
6. Final minimal normal-startup code path.
7. Final installation-only verification path.
8. Final post-MainWindow API path.
9. Runtime DB username proof (`hung`, sanitized).
10. Open OBM-POS and direct startup shared-path proof.
11. Deletion/rename table.
12. Exact source files added/changed/deleted.
13. Build/test commands and counts.
14. Prompt052 label proof.
15. Operator physical retest steps.
16. No secrets/no DB mutation/no source push proof.
17. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_MINIMAL_STARTUP_SERVICES_READY_FOR_USER_RETEST
```

```text
BLOCKED_OBM_POS_STARTUP_SERVICE_SIMPLIFICATION
```
