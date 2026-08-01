# Prompt 052 — Prioritize Activated local runtime over API bootstrap repair routing

## Physical operator evidence

Read completely before changing source:

```text
report/report049.md
report/report050.md
report/report051.md
report/report044.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The operator physically rebuilt and ran build `prompt051`.

The diagnostics UI proves the local POS is fully ready:

```text
Build label: prompt051
Phase 2 Local DB Baseline: Phase 2 v002 Complete
Local POS status: Ready / Activated / LOCAL_POS_READY_ACTIVATED
LocalDatabaseConfigResolved=True
LocalDatabaseAuthenticationSucceeded=True
SchemaReady=True
RuntimeProfileCount=1
RuntimeState=Activated
TenantIdentityConsistent=True
PosIdentityConsistent=True
LocalPosReady=True
API status: Reauthorization Required / WPF_HELLO_HTTP_401
ProtectedCredentialPresent=True
ProtectedCredentialReadable=True
AccessCredentialAccepted=False
ApiReachable=True
```

The operator clicked `Open OBM-POS` once.

The structured result now correctly exposed the real blocking route:

```text
Open OBM-POS state: Failed
StageId=InstallationV0OpenObmPos
ResultCode=POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED
```

No MainWindow became visible.

This evidence is authoritative and proves:

```text
- prompt051 structured-result preservation works;
- local DB authentication succeeds;
- schema is ready;
- runtime profile is Activated;
- local Tenant/POS identity is consistent;
- LocalPosReady=True;
- API credential is unauthorized/expired;
- the startup router still gives API/bootstrap-repair state higher priority than the ready local POS state.
```

## Operator decision — canonical route precedence

The local POS working route is authoritative once durable local installation is complete.

Canonical precedence:

```text
1. Local DB/config/auth/schema/runtime identity
2. TblPosRuntimeProfile.RuntimeState
3. Local route decision
4. API/cloud credential state after MainWindow is visible
```

For this exact physical state:

```text
LocalPosReady=True
RuntimeState=Activated
API status=ReauthorizationRequired
```

The only valid route is:

```text
RouteDecision=OpenMainPos
ResultCode=OPEN_POS_MAINWINDOW_SHOWN
API status remains ReauthorizationRequired or OfflineDeferred
```

`BootstrapRepairRequired` may be relevant only when local runtime bootstrap needed to access PostgreSQL is missing, corrupt, unreadable, or inconsistent. It must not be used when the only failed item is the Phase 1/API protected credential.

## Critical terminology distinction

Audit and separate two concepts that may currently share the word `bootstrap`:

### Local database/runtime bootstrap

Examples:

```text
ProductRoot local runtime metadata
DatabaseHost
DatabasePort
DatabaseName
DatabaseUsername
protected PostgreSQL password reference
local Tenant/POS/runtime identity
```

Failure here may legitimately block local POS startup.

### API/cloud bootstrap credential

Examples:

```text
Phase 1 WpfJwt
protected hello credential
WPF_HELLO_HTTP_401
Pairing Code reauthorization
API bootstrap identity proof
```

Failure here must become:

```text
ApiStatus=ReauthorizationRequired
or
ApiStatus=OfflineDeferred
```

It must not route an already Activated local POS to installation/bootstrap repair.

Rename ambiguous result codes/classes/comments where necessary so future AI/developers cannot confuse these two domains.

## Objective

Correct the canonical router so:

```text
LocalPosReady=True
+ RuntimeState=Activated
+ local DB bootstrap usable
=> OpenMainPos
```

independent of:

```text
WpfJwt expired
protected hello HTTP 401
API unavailable
Pairing Code absent
API reauthorization required
```

The router may attach API status metadata to the structured result, but API status must not replace the local route decision.

## First task — locate the exact active branch

Trace every source location that can produce:

```text
POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED
BootstrapRepairRequired
RepairBootstrap
NeedsBootstrapRepair
CanEnterNormalApplication
```

Audit at least:

```text
App.OpenInstalledPosFromInstallationV0Async
App.StartNormalApplicationAsync
App.ShowMainWindowForActivatedRuntimeAsync
ApplicationStartupCoordinator
DatabaseStartupAssessment / DatabaseStartupMode
RuntimeProfileStartupAssessmentService
Phase1InstallationService
InstallationV0Window
InstallationV0Module
PosStartupRouteResult
```

Return the exact predicate and value source that selected `POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED` even though `LocalPosReady=True` and `RuntimeState=Activated`.

Do not infer. Report exact pre-change branch order.

## Required route model

Create or enforce a route composition model with two independent outputs:

### Local route

```text
OpenMainPos
OpenInstallation
OpenRecovery
Blocked
```

### API/cloud state

```text
Online
OfflineDeferred
ReauthorizationRequired
Unknown
```

Required combinations include:

| Local state | API state | Local route |
| --- | --- | --- |
| Activated/ready | Online | OpenMainPos |
| Activated/ready | ReauthorizationRequired | OpenMainPos |
| Activated/ready | OfflineDeferred | OpenMainPos |
| Installing | any | OpenInstallation |
| RecoveryRequired | any | OpenRecovery |
| Disabled | any | OpenRecovery or Blocked |
| DB bootstrap missing/corrupt | any | OpenInstallation/Recovery |

Do not overload one enum/result code to represent both domains.

## Corrected decision ordering

The shared startup router must use this order:

```text
1. Resolve local runtime ProductRoot/config.
2. Validate Development protected-lane safety.
3. Authenticate to local PostgreSQL.
4. Verify schema and local identity.
5. Read TblPosRuntimeProfile.RuntimeState.
6. Select local route.
7. If route is OpenMainPos, show MainWindow.
8. Initialize/probe API independently.
9. Attach Online/OfflineDeferred/ReauthorizationRequired state without changing OpenMainPos success.
```

An API credential failure observed before step 7 may be recorded for later UI status, but cannot override an already valid local route.

## MainWindow transition

Preserve prompt051 structured transition:

```text
ShowMainWindowForActivatedRuntimeAsync
-> dispatcher
-> preserve previous window/shutdown state
-> ShutdownMode.OnExplicitShutdown
-> resolve MainWindow
-> set Application.Current.MainWindow
-> Show
-> Activate
-> verify IsVisible
-> ShutdownMode.OnMainWindowClose
-> return structured success
```

For the current physical state, this method must be reached.

The expected final result is:

```text
RouteDecision=OpenMainPos
ResultCode=OPEN_POS_MAINWINDOW_SHOWN
StageId=PosStartupRouter
RuntimeState=Activated
LocalRuntimeReady=True
MainWindowConstructed=True
MainWindowShown=True
MainWindowVisible=True
ApiStatus=ReauthorizationRequired
```

## Direct Runtime Development route

Verify the primary startup route independently:

```text
NailSalonNet8 + OBM-POS Runtime Development
-> local runtime assessment
-> Activated/ready
-> MainWindow
-> API 401 handled after local UI
```

It must not open InstallationV0 or route to bootstrap repair solely because the Phase 1/API credential is unauthorized.

## Installation and recovery semantics to preserve

Initial clean installation may still require Phase 1/API authorization before local identity/database mutation.

Local bootstrap repair remains valid only for real local failures such as:

```text
runtime bootstrap file missing
PostgreSQL password cannot be read
local DB config invalid
DB unreachable
schema unusable
runtime profile missing/inconsistent
Tenant/POS identity inconsistent
RuntimeState Installing/RecoveryRequired/Disabled
```

Do not remove these protections.

## No-mutation requirements

This prompt must not:

```text
redeem Pairing Code
refresh/rotate tokens
mutate PostgreSQL
rerun seed
rewrite markers
change runtime state/history
change employees/PINs/outbox
change DB roles/passwords
set User/Machine environment variables
```

Startup and diagnostics checks remain read-only.

## Tests

Add focused tests for at least:

```text
LocalPosReady=True + RuntimeState=Activated + API HTTP 401
-> OpenMainPos

LocalPosReady=True + RuntimeState=Activated + API unavailable
-> OpenMainPos

LocalPosReady=True + RuntimeState=Activated + API reauthorization required
-> OPEN_POS_MAINWINDOW_SHOWN

API bootstrap credential invalid
-> ApiStatus=ReauthorizationRequired
-> local route unchanged

local DB bootstrap missing
-> BootstrapRepairRequired remains valid

PostgreSQL credential unreadable
-> local route blocked/recovery

RuntimeState=Installing
-> OpenInstallation regardless of API state

RuntimeState=RecoveryRequired
-> OpenRecovery regardless of API state

prompt051 structured-result preservation remains intact

no active branch emits POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED solely from WPF_HELLO_HTTP_401

prompt052 label
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~OpenObmPos|FullyQualifiedName~RuntimeState|FullyQualifiedName~MainWindow|FullyQualifiedName~Offline|FullyQualifiedName~BootstrapRepair" -v minimal
```

## Build label

Set:

```text
Build label: prompt052
Window title: OBM InstallationV0 Phase 1/2 - prompt052
```

## Physical execution policy

Do not redeem a new Pairing Code.

Do not mutate DB.

Do not automatically launch WPF if it may interfere with the operator session.

Leave final physical tests to the operator:

```text
Route A — prompt052 InstallationV0 with current HTTP 401
          -> Local POS Ready/Activated
          -> Open OBM-POS
          -> MainWindow visible
          -> API remains ReauthorizationRequired

Route B — Runtime Development with API credential invalid/unavailable
          -> MainWindow opens directly
          -> local CRUD available
```

## Report 052

Create and push:

```text
report/report052.md
```

Required sections:

1. Verdict.
2. Physical prompt051 `POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED` evidence.
3. Exact active branch/predicate root cause.
4. Local-runtime bootstrap versus API-bootstrap distinction.
5. Corrected route precedence.
6. Local route + API state composition model.
7. MainWindow transition preservation.
8. Direct runtime behavior.
9. Installation/recovery safeguards retained.
10. No-mutation proof.
11. Exact source files changed.
12. Build/test commands and counts.
13. Prompt052 label proof.
14. Exact operator retest steps.
15. Deferred refresh-token/PIN/Identity cleanup.
16. No secrets/no DB mutation/no source push proof.
17. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_ACTIVATED_LOCAL_RUNTIME_OVERRIDES_API_BOOTSTRAP_REPAIR_READY_FOR_USER_RETEST
```

```text
BLOCKED_OBM_POS_LOCAL_RUNTIME_ROUTE_PRECEDENCE
```