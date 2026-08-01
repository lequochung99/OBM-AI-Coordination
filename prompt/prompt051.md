# Prompt 051 — Eliminate bool wrapper regression and complete Activated local POS → MainWindow transition

## Physical operator evidence

Read completely before changing source:

```text
report/report049.md
report/report050.md
report/report048.md
report/report045.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The operator physically rebuilt and ran build `prompt050`.

The diagnostics UI proves the intended local/API separation now works:

```text
Build label: prompt050
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

The `Open OBM-POS` button is correctly enabled despite API HTTP 401.

After clicking once, the UI still shows:

```text
Open OBM-POS state: Failed
StageId=InstallationV0OpenObmPos
ResultCode=OPEN_POS_CALLBACK_RETURNED_FALSE
```

No MainWindow becomes visible.

This evidence proves:

```text
- local DB readiness is fully successful;
- runtime state is Activated;
- API independence is successful;
- the button invokes the callback;
- the remaining failure is only inside the App/MainWindow transition contract;
- OPEN_POS_CALLBACK_RETURNED_FALSE is still a wrapper result and violates the structured-result design from prompt049.
```

Do not change PostgreSQL, runtime state, schema, seed, marker, outbox, employee PINs, API token data, Pairing Code state, roles/passwords, or environment variables.

## Objective

For the exact current physical state, this route must be deterministic:

```text
LocalPosReady=True
+ RuntimeState=Activated
+ RouteDecision=OpenMainPos
-> construct/show/activate exactly one MainWindow
-> verify MainWindow.IsVisible=True
-> return the complete PosStartupRouteResult unchanged
-> close InstallationV0Window only after success
```

A generic `OPEN_POS_CALLBACK_RETURNED_FALSE` result must not exist after this prompt when the callback uses `PosStartupRouteResult`.

## First task — audit compile-time callback types and every adapter

Inventory the exact current signatures and implementations for:

```text
InstallationV0Window.OpenPosAsync
InstallationV0Module.RequestOpenObmPosAsync
InstallationV0Module.OpenObmPosRequested
App.OpenInstalledPosFromInstallationV0Async
App.RetryStartupAssessmentAsync
App.StartNormalApplicationAsync
PosStartupRouteResult
```

For each boundary record:

```text
input type
return type
whether PosStartupRouteResult is preserved
whether it is converted to bool
whether false is synthesized
whether an exception is swallowed or replaced
whether ResultCode/StageId are overwritten
```

Search all source for:

```text
OPEN_POS_CALLBACK_RETURNED_FALSE
Task<bool>
Func<string, Task<bool>>
Current.MainWindow is MainWindow
CanEnterNormalApplication
RouteDecision
MainWindowVisible
```

There must be no stale bool-only adapter on the canonical prompt049/prompt050 path.

## Preserve the underlying structured result

The canonical callback contract must be exactly equivalent to:

```csharp
Func<string, Task<PosStartupRouteResult>>
```

`RequestOpenObmPosAsync` must return the callback's `PosStartupRouteResult` unchanged except for only these boundary failures:

```text
OPEN_POS_CALLBACK_MISSING
OPEN_POS_CALLBACK_EXCEPTION
```

It must not replace a returned structured failure with:

```text
OPEN_POS_CALLBACK_RETURNED_FALSE
```

If any compatibility bool overload still exists, remove it from the active route or clearly isolate it as unused legacy code and add a test proving the current UI cannot call it.

## Canonical startup route for the current physical state

The prompt049 router must return before MainWindow construction:

```text
RouteDecision=OpenMainPos
ResultCode=LOCAL_POS_READY_ACTIVATED or equivalent pre-show ready code
StageId=PosStartupRouter
RuntimeState=Activated
LocalRuntimeReady=True
```

Then the WPF lifecycle must produce the final result:

```text
RouteDecision=OpenMainPos
ResultCode=OPEN_POS_MAINWINDOW_SHOWN
StageId=PosStartupRouter
RuntimeState=Activated
LocalRuntimeReady=True
MainWindowConstructed=True
MainWindowShown=True
MainWindowVisible=True
```

If the route is not `OpenMainPos`, preserve and display the real underlying result. Do not fabricate a wrapper false.

## MainWindow lifecycle audit

Trace the exact current code and state at each point:

```text
Application.Current.ShutdownMode
Application.Current.MainWindow before transition
existing visible MainWindow count
InstallationV0Window count
_normalApplicationStarted
_isStartingNormalApplication
_startupAssessment
DI/service provider availability
WPF dispatcher access
MainWindow constructor entry/exit
MainWindow.Show entry/exit
MainWindow.Activate
MainWindow.IsVisible
MainWindow.IsLoaded
post-show exception handling
```

Identify the exact predicate that currently causes the callback not to return success even though local readiness is true.

Likely failure classes to verify, not assume:

1. `StartNormalApplicationAsync` returns a structured success but an outer method still checks a stale bool or `Current.MainWindow` before the dispatcher has shown it.
2. `Application.Current.MainWindow` remains the InstallationV0Window and the callback tests the wrong instance.
3. `_normalApplicationStarted` or another latch is already true from diagnostics startup and incorrectly skips MainWindow construction.
4. A visible MainWindow is created, but a later broad catch/shutdown path closes it before success is returned.
5. `ShutdownMode=OnMainWindowClose` or `OnLastWindowClose` interacts incorrectly when replacing the diagnostics window.
6. The MainWindow is resolved from DI but construction fails, and the failure is converted to wrapper false.
7. The prompt050 hydration/UI update reintroduced an old `Task<bool>` overload or registration.

Report the exact root cause and line/method.

## Required window transition

Run on the WPF dispatcher. Preserve the diagnostics window until MainWindow visibility is confirmed.

Required sequence:

```csharp
var previousMainWindow = Application.Current.MainWindow;

var mainWindow = ResolveOrCreateMainWindow();

Application.Current.ShutdownMode = ShutdownMode.OnExplicitShutdown;
Application.Current.MainWindow = mainWindow;

mainWindow.Show();
mainWindow.Activate();

if (!mainWindow.IsVisible)
{
    // restore safe state and return structured failure
}

// return OPEN_POS_MAINWINDOW_SHOWN first
// InstallationV0Window closes only after the UI receives success
```

After diagnostics closes, set the intended normal shutdown mode if required by the existing app lifecycle.

Do not close or hide InstallationV0Window before `MainWindow.IsVisible == true`.

Do not start a second process.

Do not let API bootstrap run before visibility proof. API work remains post-show and non-blocking.

## Direct startup must use the same route

Verify:

```text
NailSalonNet8 + OBM-POS Runtime Development
-> local status Activated
-> OpenMainPos
-> MainWindow visible
```

The direct route and InstallationV0 handoff must call the same MainWindow transition method. There must not be two separate implementations that can drift.

Create one clearly named method, for example:

```text
OpenMainPosFromReadyLocalRuntimeAsync
ExecutePosStartupRouteAsync
ShowMainWindowForActivatedRuntimeAsync
```

The method returns `PosStartupRouteResult` and owns MainWindow construction/show/visibility evidence.

## UI behavior

On click:

```text
Open OBM-POS state: Opening
```

On success, the diagnostics window closes after MainWindow is visible.

On failure, keep diagnostics open and render every structured field:

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

The UI must not display `OPEN_POS_CALLBACK_RETURNED_FALSE` after this prompt.

## No-mutation requirements

This prompt must not:

```text
- mutate PostgreSQL;
- run migration or seed;
- change Phase2 marker;
- change TblPosRuntimeProfile or history;
- insert outbox rows;
- redeem Pairing Code;
- alter API credentials;
- change employee PINs;
- change DB roles/passwords;
- set User/Machine environment variables.
```

## Tests

Add focused tests for at least:

```text
callback signature is PosStartupRouteResult end-to-end
no active Task<bool>/Func<string,Task<bool>> adapter
structured success is not replaced by wrapper false
structured failure preserves original ResultCode/StageId
LocalPosReady=True + Activated -> RouteDecision OpenMainPos
OpenMainPos -> MainWindow constructed/shown/visible flags true
Application.Current.MainWindow points to the visible MainWindow
InstallationV0 remains open until success result
InstallationV0 closes after visible MainWindow success
post-show API 401 does not reverse route success
existing visible MainWindow is activated and returns success
reentrant click does not create a second MainWindow
constructor exception -> MAINWINDOW_CONSTRUCTION_FAILED structured result
Show exception -> MAINWINDOW_SHOW_FAILED structured result
no OPEN_POS_CALLBACK_RETURNED_FALSE string remains in active source/tests/UI
prompt051 build label
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~OpenObmPos|FullyQualifiedName~RuntimeState|FullyQualifiedName~MainWindow|FullyQualifiedName~Offline" -v minimal
```

## Build label

Set:

```text
Build label: prompt051
Window title: OBM InstallationV0 Phase 1/2 - prompt051
```

## Physical execution policy

Do not launch WPF automatically if it may interfere with the operator.

Do not redeem Pairing Code.

Do not mutate DB.

Leave final physical tests to the operator:

```text
Route A — prompt051 diagnostics, API remains HTTP 401
          -> Local POS Ready / Activated
          -> click Open OBM-POS once
          -> MainWindow visible
          -> diagnostics closes

Route B — Runtime Development direct
          -> MainWindow opens directly
          -> API status deferred/reauthorization-required
```

## Report 051

Create and push:

```text
report/report051.md
```

Required sections:

1. Verdict.
2. Physical prompt050 evidence.
3. Exact callback signature inventory.
4. Exact stale bool/wrapper branch root cause.
5. Structured-result preservation correction.
6. Exact MainWindow lifecycle root cause.
7. Shared direct/handoff MainWindow transition method.
8. Window/shutdown-mode ordering.
9. Final structured success/failure examples.
10. UI rendering behavior.
11. API-offline independence.
12. No-mutation proof.
13. Exact source files changed.
14. Build/test commands and counts.
15. Prompt051 label proof.
16. Exact operator retest steps.
17. Deferred refresh-token/PIN/Identity cleanup work.
18. No secrets/no DB mutation/no source push proof.
19. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_STRUCTURED_MAINWINDOW_TRANSITION_READY_FOR_USER_RETEST
```

```text
BLOCKED_OBM_POS_STRUCTURED_MAINWINDOW_TRANSITION
```
