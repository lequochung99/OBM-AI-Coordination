# Prompt 049 — Make local POS runtime status the canonical startup router and close `OPEN_POS_CALLBACK_RETURNED_FALSE`

## Physical operator evidence

Read completely before changing source:

```text
report/report045.md
report/report047.md
report/report048.md
report/report044.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The operator physically rebuilt and retested build `prompt048`.

Visible state:

```text
Build label: prompt048
Phase 2 Local DB Baseline: Phase 2 v002 Complete
Target DB: obm_pos_dev_v0_pg
ProductRoot: E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
Pairing Code is not required
```

After clicking `Open OBM-POS` once, the diagnostics UI showed:

```text
Open OBM-POS state: Failed
StageId=InstallationV0OpenObmPos
ResultCode=OPEN_POS_CALLBACK_RETURNED_FALSE
```

No MainWindow became visible.

This evidence proves:

```text
- the click reached InstallationV0Module;
- the callback existed and completed without an uncaught exception;
- the callback returned false;
- prompt048 observability works;
- the underlying App/startup routing decision is still not exposed;
- OPEN_POS_CALLBACK_RETURNED_FALSE is a wrapper result, not the real root cause.
```

## Operator decision — canonical source of truth

The operator requires the normal WPF startup decision to come from the local POS runtime status in PostgreSQL.

Canonical principle:

```text
WPF starts
-> connect to configured local PostgreSQL
-> read current POS runtime status and local identity
-> if local POS is ready/Activated, open the normal POS MainWindow
-> otherwise route to installation/recovery according to the actual local status
```

The normal POS must not depend on a chain of InstallationV0 callbacks to discover that it is already installed.

The primary current local status source is expected to be:

```text
dbo.TblPosRuntimeProfile
```

with history in:

```text
dbo.TblPosRuntimeStateHistory
```

Audit the exact source/model enum and physical values before implementing. Do not invent state names that do not exist. The current physical target has previously been verified as one runtime profile with `RuntimeState=Activated`.

## Objective

Create one canonical, structured startup router shared by:

```text
A. normal NailSalonNet8 startup
B. InstallationV0 diagnostics -> Open OBM-POS
```

Both routes must use the same local-DB-first decision and return the same structured result.

Target behavior:

```text
Start NailSalonNet8
    -> resolve approved ProductRoot/config lane
    -> authenticate to local PostgreSQL
    -> verify schema
    -> read TblPosRuntimeProfile
    -> verify matching TblTenant and TblPosLocal identity
    -> evaluate RuntimeState

RuntimeState ready/Activated
    -> construct and show exactly one MainWindow

Installation incomplete state
    -> show InstallationV0Window

Recovery/disabled/invalid state
    -> show recovery/blocked UI with precise local result

API unavailable/token expired
    -> does not change the local startup route
    -> MainWindow opens in OfflineDeferred mode when local runtime is ready
```

## First task — prove the physical local status

Use PostgreSQL read-only access against `obm_pos_dev_v0_pg` and record sanitized evidence:

```text
TblPosRuntimeProfile row count
RuntimeState exact physical value
RuntimeProfileGuid present
TenantGuid/PosGuid present
DatabaseName
EnvironmentName
SourceClientId shape valid
matching TblTenant count
matching TblPosLocal count
latest TblPosRuntimeStateHistory transition summary
```

Use:

```text
BEGIN TRANSACTION READ ONLY
...
ROLLBACK
```

Do not print private GUID values, credentials, connection strings, employee data, or raw history payloads.

If the physical profile is not ready according to the actual state enum, report the exact state and stop with a precise blocker. Do not update the status manually.

## Second task — find the exact false-return branch

Trace every return path from:

```text
InstallationV0Window.OpenPosAsync
InstallationV0Module.RequestOpenObmPosAsync
InstallationV0Module.OpenObmPosRequested
App.OpenInstalledPosFromInstallationV0Async
App.RetryStartupAssessmentAsync
App.StartNormalApplicationAsync
ApplicationStartupCoordinator
RuntimeProfileStartupAssessmentService
MainWindow DI resolution / constructor / Show
```

Audit all relevant state/latches:

```text
_startupAssessment
_normalApplicationStarted or equivalent
_isStartingNormalApplication or equivalent
installation/diagnostics mode latch
LaunchProvenanceContext
EffectiveProductRootContext
Application.Current.MainWindow
Application.ShutdownMode
current dispatcher/thread
existing MainWindow instances
existing InstallationV0Window instances
```

Identify the exact source line and predicate that ultimately caused the callback to return `false`.

The report must not stop at `OPEN_POS_CALLBACK_RETURNED_FALSE`.

Required root-cause evidence format:

```text
UnderlyingResultCode
UnderlyingStageId
StartupDecision
RuntimeState
LocalRuntimeReady
MainWindowConstructionAttempted
MainWindowShowAttempted
MainWindowVisible
FalseReturnPredicate
```

Do not print secrets or private identifiers.

## Replace bool-only handoff with a structured result

A plain `Task<bool>` is no longer sufficient.

Create or reuse one minimal structured result, for example:

```text
PosStartupRouteResult
```

It must contain at least:

```text
RouteDecision
ResultCode
StageId
SafeMessage
RuntimeState
LocalRuntimeReady
MainWindowConstructed
MainWindowShown
MainWindowVisible
```

Allowed route decisions should map to existing application concepts, for example:

```text
OpenMainPos
OpenInstallation
OpenRecovery
Blocked
```

Use exact existing enum/types when available. Do not create duplicate competing state systems.

`InstallationV0Module` must return the structured result to `InstallationV0Window`. The UI may derive success from `RouteDecision=OpenMainPos` and `MainWindowVisible=true`, but must preserve the underlying result when blocked.

## One canonical local startup router

Refactor or identify one canonical router/service used by both direct startup and diagnostics handoff.

Preferred conceptual interface:

```text
AssessLocalPosRuntimeAsync(ProductRoot/config)
RouteAsync(result)
```

Do not maintain separate readiness rules in:

```text
App startup
InstallationV0 Open OBM-POS callback
Development profile guard
InstallationV0 completion verifier
```

The detailed InstallationV0 verifier may remain for installation/audit, but it must not be the normal MainWindow router.

### Canonical local readiness

Keep only the prompt045 local requirements:

```text
- explicit approved Development ProductRoot/config lane;
- canonical database obm_pos_dev_v0_pg in Development;
- PostgreSQL authentication succeeds;
- expected schema version exists;
- exactly one current runtime profile;
- runtime profile points to the configured database;
- runtime Tenant/POS identity is non-empty and internally consistent;
- matching TblTenant exists exactly once for the current identity;
- matching TblPosLocal exists exactly once for the current POS/slot;
- RuntimeState is the exact existing usable/Activated state;
- no explicit local RecoveryRequired/Disabled/blocking state.
```

Do not require for normal routing:

```text
WpfJwt validity
API reachability
Pairing Code
employee operational PIN configuration
exact employee count
exact permission count
outbox count
Phase1 checkpoint
full Phase2 proof count set
launch provenance as authentication
```

## Runtime-state route table

Inspect the actual runtime-state enum/constants and produce a route table.

At minimum, prove the route for the physically present `Activated` state:

```text
Activated + local identity/schema ready
-> OpenMainPos
```

For every other existing state found in source, explicitly map it to:

```text
Installation
Recovery
Blocked
```

Examples such as `Installing`, `RecoveryRequired`, or `Disabled` may be used only if they physically exist in source.

Unknown state must fail closed with:

```text
POS_RUNTIME_STATE_UNKNOWN
```

Do not silently reinterpret state values.

## Direct normal startup is primary

The standard Visual Studio/runtime route must work without first opening InstallationV0:

```text
NailSalonNet8 + OBM-POS Runtime Development
-> local startup router
-> RuntimeState Activated
-> MainWindow
```

InstallationV0 diagnostics is secondary. Its `Open OBM-POS` button must call the same router and produce the same decision.

Do not require an installation-window callback for normal startup.

## MainWindow lifecycle rules

When route decision is `OpenMainPos`:

```text
- execute on Application.Current.Dispatcher;
- prevent concurrent/reentrant startup;
- resolve or construct exactly one MainWindow;
- set Application.Current.MainWindow to that instance;
- call Show();
- call Activate();
- verify IsVisible=true;
- only then close InstallationV0Window;
- set the structured result to success;
- do not call Application.Shutdown after MainWindow is visible;
- do not let later API/bootstrap exceptions reverse the successful local route.
```

If a MainWindow already exists and is visible:

```text
activate the existing instance
return success
```

If MainWindow construction or Show fails:

```text
return precise safe exception classification
keep InstallationV0Window open
```

Do not swallow exceptions.

## Window-shutdown audit

Audit WPF shutdown behavior carefully:

```text
ShutdownMode
closing the current Application.MainWindow
whether InstallationV0Window is temporarily Application.MainWindow
whether closing it shuts down the application before MainWindow remains alive
startup broad catch blocks
post-MainWindow API/bootstrap catches
```

A likely failure class is that the application/window state is latched around the diagnostics window. Verify rather than assume.

The final sequence must ensure closing diagnostics cannot terminate the app after MainWindow is visible.

## API independence

Once local route returns `OpenMainPos`, API startup remains post-MainWindow and non-blocking.

These must not change the route to failure:

```text
API unavailable
WpfJwt expired
station access token missing/expired
SignalR unavailable
sync worker failure
```

Record them as:

```text
OfflineDeferred
ReauthorizationRequired
```

using existing mechanisms where possible.

Do not implement refresh tokens in prompt049.

## No database mutation

Prompt049 startup and diagnostics handoff must remain read-only.

Do not:

```text
run seed
run migrations
rewrite markers
update TblPosRuntimeProfile
insert runtime history
insert outbox
change employee PINs
redeem Pairing Code
change PostgreSQL roles/passwords
```

## Build label

Set InstallationV0 diagnostics label to:

```text
Build label: prompt049
Window title: OBM InstallationV0 Phase 1/2 - prompt049
```

## Tests

Add focused tests for at least:

```text
physical-style Activated runtime profile -> OpenMainPos
Activated + valid Tenant/POS identity -> MainWindow visible
Activated + API unavailable -> MainWindow visible / OfflineDeferred
normal Runtime Development startup uses canonical router directly
InstallationV0 Open OBM-POS uses the same canonical router
callback returns structured underlying result, not bool-only false
existing visible MainWindow is activated and treated as success
InstallationV0 as current Application.MainWindow does not prevent handoff
closing diagnostics after MainWindow visible does not shut down application
MainWindow constructor exception -> precise result and diagnostics remains open
MainWindow Show exception -> precise result and diagnostics remains open
Installing/incomplete existing state -> installation route
RecoveryRequired/Disabled existing state -> recovery/blocked route
unknown runtime state -> fail closed
local DB/schema/identity failure -> exact local result
Phase/API/PIN/count proofs not required for Activated local startup
no seed/outbox/marker/runtime-state mutation on startup
prompt049 label
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap|FullyQualifiedName~DevelopmentProfile|FullyQualifiedName~OpenObmPos|FullyQualifiedName~RuntimeState" -v minimal
```

## Physical execution policy

Do not automatically launch WPF if it may interfere with the operator's Visual Studio session.

Leave final physical tests to the operator:

```text
Route A:
OBM-POS Runtime Development -> MainWindow directly

Route B:
prompt049 InstallationV0 -> Open OBM-POS once -> MainWindow

Route C:
API unavailable -> Runtime Development -> MainWindow local mode
```

## Report 049

Create and push:

```text
report/report049.md
```

Required sections:

1. Verdict.
2. Physical prompt048 failure evidence.
3. Read-only physical runtime-status evidence.
4. Exact actual runtime-state enum/value inventory.
5. Exact underlying callback false predicate/root cause.
6. Pre-change startup and handoff decision trees.
7. Structured startup result contract.
8. Canonical shared local startup router.
9. Runtime-state route table.
10. Direct Runtime Development behavior.
11. InstallationV0 Open OBM-POS behavior.
12. MainWindow lifecycle/shutdown correction.
13. API-offline independence.
14. Exact source files changed.
15. Build/test commands and counts.
16. Read-only/no-mutation DB proof.
17. Prompt049 label proof.
18. Exact operator physical retest steps.
19. Deferred work: refresh token, PIN normalization, legacy Identity cleanup.
20. No secrets/no source push confirmation.
21. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_RUNTIME_STATE_ROUTER_READY_FOR_USER_RETEST
```

```text
BLOCKED_OBM_POS_RUNTIME_STATE_ROUTER
```
