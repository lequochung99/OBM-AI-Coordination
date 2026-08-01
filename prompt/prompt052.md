# Prompt 052 — Remove API/bootstrap repair from normal POS startup; local PostgreSQL is authoritative

## Operator correction — authoritative and intentionally simple

The previous prompts began adding too many status and route layers again.

For normal OBM-POS operation, the rule is simple:

```text
1. Resolve the configured local PostgreSQL database.
2. Connect successfully using the runtime PostgreSQL role:
   Username = hung
   Password = the protected valid PostgreSQL password.
3. Verify the local schema is usable.
4. Open MainWindow.
```

OBM-POS does **not** have an application-user password requirement for startup.

Employee `LoginNumber` is an operational PIN for local UI gating and audit attribution. It is not a startup credential.

API authentication is independent:

```text
API token valid
-> API/sync online

API token expired, HTTP 401, or API unavailable
-> MainWindow still opens
-> local PostgreSQL CRUD continues
-> API/sync is Offline or Reauthorization Required
```

The current physical build `prompt051` already proves:

```text
LocalDatabaseConfigResolved=True
LocalDatabaseAuthenticationSucceeded=True
SchemaReady=True
RuntimeProfileCount=1
RuntimeState=Activated
TenantIdentityConsistent=True
PosIdentityConsistent=True
LocalPosReady=True
```

Yet `Open OBM-POS` returns:

```text
POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED
```

That result is incorrect for a usable local database. Do not add another override layer. Remove the incorrect dependency.

## Read first

```text
report/report044.md
report/report045.md
report/report049.md
report/report050.md
report/report051.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

## Exact objective

For the current installed database:

```text
Database = obm_pos_dev_v0_pg
Runtime PostgreSQL role = hung
PostgreSQL authentication = successful
Schema = usable
```

the only valid local startup result is:

```text
OPEN_POS_MAINWINDOW_SHOWN
```

The following must never block MainWindow after local DB readiness succeeds:

```text
WpfJwt expiration
protected hello HTTP 401
API reachability
Pairing Code absence
API bootstrap repair
API reauthorization requirement
Phase1 current-token status
employee operational PIN state
exact employee/permission/outbox counts
launch provenance
```

## Canonical startup code path

Normal startup must be reduced to this:

```text
Start NailSalonNet8
-> resolve ProductRoot/config
-> load local DB host/port/database/user/password
-> connect PostgreSQL
-> verify required local schema
-> load local Tenant/POS context
-> show MainWindow
-> start API/auth/sync work afterward and non-blockingly
```

Do not require a protected API call before `MainWindow.Show()`.

Do not require Pairing Code redemption before `MainWindow.Show()`.

Do not route to InstallationV0 merely because API authentication is expired.

## PostgreSQL runtime credential

Audit the active runtime bootstrap/config and prove safely:

```text
DatabaseName = obm_pos_dev_v0_pg
DatabaseUsername = hung
Password source = protected local PostgreSQL credential
```

Do not print the password or connection string.

If active normal runtime still uses `postgres`, correct the runtime configuration/source so ordinary operation uses `hung`.

The `postgres` administrator role may remain provisioning/backup-only. It must not be the intended daily runtime role.

Do not create users, alter roles, change passwords, or issue GRANT/REVOKE in this prompt.

## Minimal local readiness check

Keep only checks that prevent an unusable DB from crashing the app:

```text
- local DB config exists;
- PostgreSQL authentication succeeds;
- expected application schema exists;
- essential tables/columns needed by startup exist;
- one local Tenant/POS context can be resolved.
```

`TblPosRuntimeProfile` may provide local identity/status metadata, but it is not an authentication system and must not be coupled to API token state.

For the current physical database, `RuntimeState=Activated` is consistent evidence that the local POS is installed. Do not add additional Phase/API proof gates after this point.

## Remove the incorrect BootstrapRepair route

Find every active branch producing:

```text
POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED
BootstrapRepairRequired
RepairBootstrap
NeedsBootstrapRepair
```

Classify each branch as either:

```text
A. Local PostgreSQL configuration/credential failure
B. API/cloud credential failure
```

Required behavior:

```text
A -> installation/recovery may be appropriate
B -> never blocks MainWindow; API status only
```

If the code uses one ambiguous `BootstrapRepairRequired` value for both domains, split or rename it. Prefer deletion from the normal local startup path rather than another precedence/override layer.

After this prompt, `WPF_HELLO_HTTP_401` must not be able to produce:

```text
POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED
OpenInstallation
OpenRecovery
Blocked
```

when local PostgreSQL is usable.

## MainWindow transition

Preserve the structured transition from prompt051:

```text
PosStartupRouteResult
ShowMainWindowForActivatedRuntimeAsync
```

But make the route decision directly from local DB readiness.

Success must be:

```text
RouteDecision=OpenMainPos
ResultCode=OPEN_POS_MAINWINDOW_SHOWN
StageId=PosStartupRouter
LocalRuntimeReady=True
MainWindowConstructed=True
MainWindowShown=True
MainWindowVisible=True
```

Then API status may be recorded separately as:

```text
Online
OfflineDeferred
ReauthorizationRequired
```

API status must not rewrite the successful local route.

## InstallationV0 diagnostics button

`Open OBM-POS` must call the same local DB startup method as direct runtime startup.

It must not call a separate API/bootstrap readiness path.

Required result for the current physical state:

```text
click once
-> local DB connection succeeds
-> schema usable
-> MainWindow visible
-> InstallationV0 closes
-> API remains ReauthorizationRequired
```

## Direct runtime startup

Verify independently:

```text
NailSalonNet8
+ OBM-POS Runtime Development
-> connect obm_pos_dev_v0_pg as hung
-> MainWindow opens directly
```

Even with API stopped or WpfJwt expired:

```text
MainWindow opens
local CRUD remains available
API/sync is offline only
```

## Do not add more architecture

Do not add new route engines, provenance layers, hidden security flags, installation status tables, authentication abstractions, or password systems.

Prefer deleting conditions and simplifying methods.

The final normal-startup decision should be understandable as:

```csharp
if (await LocalDatabaseIsUsableAsync())
{
    return await ShowMainWindowAsync();
}

return OpenInstallationOrRecoveryForLocalDatabaseFailure();
```

API initialization belongs after the successful `ShowMainWindowAsync()` result.

## No-mutation requirements

Do not:

```text
- mutate PostgreSQL;
- rerun seed;
- rewrite markers;
- update runtime state/history;
- redeem Pairing Code;
- refresh/rotate tokens;
- change employee PINs;
- create/alter PostgreSQL roles;
- change DB passwords;
- set User/Machine environment variables.
```

## Required source audit

Inspect and simplify only the necessary paths, including:

```text
App.xaml.cs
ApplicationStartupCoordinator
RuntimeProfileStartupAssessmentService
DatabaseStartupAssessment / DatabaseStartupMode
StartNormalApplicationAsync
OpenInstalledPosFromInstallationV0Async
ShowMainWindowForActivatedRuntimeAsync
InstallationV0Module
InstallationV0Window
AppJwtBootstrapper / post-MainWindow API startup
runtime bootstrap/config credential provider
```

Report the exact branch that incorrectly translated API HTTP 401 into local bootstrap repair.

## Tests

Add focused tests proving:

```text
local DB usable + user hung credential accepted + API HTTP 401
-> OPEN_POS_MAINWINDOW_SHOWN

local DB usable + API unavailable
-> OPEN_POS_MAINWINDOW_SHOWN

local DB usable + WpfJwt expired
-> OPEN_POS_MAINWINDOW_SHOWN

local DB usable + Pairing Code absent
-> OPEN_POS_MAINWINDOW_SHOWN

API 401
-> API status ReauthorizationRequired
-> local route unchanged

local DB config missing
-> installation/recovery route

PostgreSQL authentication fails
-> installation/recovery route

schema missing/unusable
-> installation/recovery route

no active API branch produces POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED

runtime uses configured role hung, not postgres, for normal operation

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

## Physical execution policy

Do not redeem Pairing Code.

Do not mutate DB.

Leave final physical tests to the operator:

```text
A. InstallationV0 prompt052, API still HTTP 401
   -> Open OBM-POS
   -> MainWindow opens

B. Runtime Development, API unavailable
   -> MainWindow opens directly
   -> local CRUD works
```

## Report 052

Create and push:

```text
report/report052.md
```

Required sections:

1. Verdict.
2. Evidence that local DB was ready while API/bootstrap repair blocked startup.
3. Exact incorrect branch removed.
4. Final minimal local PostgreSQL startup rule.
5. Runtime DB username proof (`hung`, sanitized).
6. API-offline behavior.
7. MainWindow transition proof.
8. Conditions deleted from normal startup.
9. Conditions retained only for real local DB failure.
10. Exact source files changed.
11. Build/test commands and counts.
12. Prompt052 label proof.
13. Operator retest steps.
14. No secrets/no DB mutation/no source push proof.
15. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_SIMPLE_LOCAL_DATABASE_STARTUP_READY_FOR_USER_RETEST
```

```text
BLOCKED_OBM_POS_SIMPLE_LOCAL_DATABASE_STARTUP
```
