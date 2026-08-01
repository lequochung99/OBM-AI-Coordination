# Report 051 — Structured MainWindow Transition

## 1. Verdict

OBM_POS_STRUCTURED_MAINWINDOW_TRANSITION_READY_FOR_USER_RETEST

Prompt051 removed the remaining bool-wrapper regression from the InstallationV0 Open OBM-POS path and made the MainWindow transition return a preserved `PosStartupRouteResult` end to end.

## 2. Physical prompt050 evidence

Operator evidence for build `prompt050`:

- Phase 2 Local DB Baseline: `Phase 2 v002 Complete`.
- Local POS status: `Ready / Activated / LOCAL_POS_READY_ACTIVATED`.
- `RuntimeProfileCount=1`.
- `RuntimeState=Activated`.
- Tenant and POS identity consistency: true.
- LocalPosReady: true.
- API status: `Reauthorization Required / WPF_HELLO_HTTP_401`.
- Open OBM-POS button enabled.
- Clicking Open OBM-POS still produced `OPEN_POS_CALLBACK_RETURNED_FALSE` and no visible MainWindow.

This proved local/API separation was fixed, and the remaining defect was only in the callback/window transition contract.

## 3. Exact callback signature inventory

Current active signatures:

- `InstallationV0Window.OpenPosAsync`: input UI click, return `Task`; receives `PosStartupRouteResult` from module and derives UI state from structured fields.
- `InstallationV0Module.OpenObmPosRequested`: `Func<string, Task<PosStartupRouteResult>>?`.
- `InstallationV0Module.RequestOpenObmPosAsync`: input `string verifiedProductRoot`, return `Task<PosStartupRouteResult>`.
- `App.OpenInstalledPosFromInstallationV0Async`: input `string verifiedProductRoot`, return `Task<PosStartupRouteResult>`.
- `App.RetryStartupAssessmentAsync`: return `Task<DatabaseStartupAssessment>`.
- `App.StartNormalApplicationAsync`: return `Task<PosStartupRouteResult>`.
- `PosStartupRouteResult`: structured route contract with route decision, result code, stage, safe message, runtime state, local readiness, and MainWindow flags.

No active `Func<string, Task<bool>>` callback remains in the InstallationV0 handoff path.

## 4. Exact stale bool/wrapper branch root cause

Two active prompt050 branches still synthesized the wrapper result:

1. `InstallationV0Module.RequestOpenObmPosAsync` formatted the returned structured result, then attempted a stale bool-style fallback to `OPEN_POS_CALLBACK_RETURNED_FALSE` for non-success results.
2. `InstallationV0Window.SetOpenPosFailed()` hard-coded `OPEN_POS_CALLBACK_RETURNED_FALSE` as the visible failure result instead of showing the callback's structured result.

That meant any structured failure, and possibly the post-click physical failure, could still collapse into the old wrapper string.

## 5. Structured-result preservation correction

`InstallationV0Module.RequestOpenObmPosAsync` now returns the callback result unchanged except for true boundary failures:

- `OPEN_POS_CALLBACK_MISSING`
- `OPEN_POS_CALLBACK_EXCEPTION`

`InstallationV0Window` now passes the actual `PosStartupRouteResult` into `SetOpenPosFailed(routeResult)` and renders every structured field. It no longer fabricates the bool-wrapper result.

## 6. Exact MainWindow lifecycle root cause

The lifecycle weakness was that MainWindow construction/show was embedded inside `StartNormalApplicationAsync`, while the handoff layer performed additional post-return checks and stage/result rewrites. The diagnostics path could therefore still report a wrapper failure instead of the exact transition result.

Prompt051 centralizes the visible MainWindow transition in `ShowMainWindowForActivatedRuntimeAsync`, so construction/show/visibility proof is owned by one method and returned as `PosStartupRouteResult`.

## 7. Shared direct/handoff MainWindow transition method

Added shared method:

`ShowMainWindowForActivatedRuntimeAsync(IServiceProvider services)`

Both direct runtime startup and InstallationV0 handoff reach this method through `StartNormalApplicationAsync`. The handoff no longer wraps or rewrites the returned success/failure result after `StartNormalApplicationAsync` returns.

## 8. Window/shutdown-mode ordering

The transition now follows the prompt051 ordering:

1. execute on WPF dispatcher;
2. activate existing visible MainWindow if present;
3. preserve previous `Application.Current.MainWindow` and `ShutdownMode`;
4. set `ShutdownMode.OnExplicitShutdown` before replacing the diagnostics window as main window;
5. resolve MainWindow from DI;
6. set `Application.Current.MainWindow` to MainWindow;
7. call `Show()`;
8. call `Activate()`;
9. verify `IsVisible`;
10. set normal `ShutdownMode.OnMainWindowClose` after visibility proof;
11. return `OPEN_POS_MAINWINDOW_SHOWN`.

On failed show/constructor exception, it restores the previous window/shutdown state and returns a structured blocked result.

## 9. Final structured success/failure examples

Success:

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

Structured failure example:

```text
RouteDecision=Blocked
ResultCode=OPEN_POS_MAINWINDOW_CONSTRUCTION_FAILED
StageId=PosStartupRouter
RuntimeState=Activated
LocalRuntimeReady=True
MainWindowConstructed=False
MainWindowShown=False
MainWindowVisible=False
```

## 10. UI rendering behavior

On failure, diagnostics stays open and renders:

- `RouteDecision`
- `ResultCode`
- `StageId`
- `SafeMessage`
- `RuntimeState`
- `LocalRuntimeReady`
- `MainWindowConstructed`
- `MainWindowShown`
- `MainWindowVisible`

The retired wrapper literal is no longer present in active InstallationV0 source/tests/UI text.

## 11. API-offline independence

Post-show API/bootstrap/sync failures remain deferred local-mode work. They do not reverse `OPEN_POS_MAINWINDOW_SHOWN`, do not close MainWindow, and do not force Pairing Code redemption.

## 12. No-mutation proof

No PostgreSQL access was performed for prompt051. No WPF launch was performed. No Pairing Code was redeemed. No seed, migration, marker rewrite, runtime-state/history update, outbox insert, employee PIN update, role/password change, API credential mutation, or User/Machine environment write was performed.

## 13. Exact source files changed

OBM source files changed locally only:

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0Module.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

No OBM source commit or push was performed.

## 14. Build/test commands and counts

Commands run:

```powershell
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal -clp:ErrorsOnly
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal -clp:ErrorsOnly
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~OpenObmPos|FullyQualifiedName~RuntimeState|FullyQualifiedName~MainWindow|FullyQualifiedName~Offline" -v minimal -clp:ErrorsOnly
```

Results:

- InstallationV0 build: passed, 0 warnings, 0 errors.
- WPF build: passed, 176 warnings, 0 errors.
- Filtered tests: passed, 143 passed, 0 failed, 0 skipped.

## 15. Prompt051 label proof

`InstallationV0BuildInfo.CoordinationPromptLabel` is now `prompt051`.

The title remains:

`OBM InstallationV0 Phase 1/2 - {InstallationV0BuildInfo.CoordinationPromptLabel}`

The visible build label remains:

`Build label: {InstallationV0BuildInfo.CoordinationPromptLabel}`

## 16. Exact operator retest steps

Route A — prompt051 diagnostics with API HTTP 401:

1. Start prompt051 InstallationV0 diagnostics with the existing V0 ProductRoot.
2. Confirm Phase 2 remains `Phase 2 v002 Complete`.
3. Confirm Local POS status is `Ready / Activated`.
4. Confirm API status may remain `Reauthorization Required / WPF_HELLO_HTTP_401`.
5. Click `Open OBM-POS` once.
6. Require MainWindow visible.
7. Require diagnostics closes only after MainWindow is visible.
8. If it fails, require structured fields and no wrapper false result.

Route B — Runtime Development direct:

1. Start `NailSalonNet8` with `OBM-POS Runtime Development`.
2. Require direct MainWindow open from Activated local runtime.
3. API status may defer/require reauthorization after local UI is visible.

## 17. Deferred refresh-token/PIN/Identity cleanup work

Still deferred:

- access-token plus refresh-token runtime lifecycle;
- PIN normalization;
- legacy Identity cleanup;
- production-grade offline/reauthorization UX polish.

## 18. No secrets/no DB mutation/no source push proof

No secrets were printed or persisted in report evidence:

- no PostgreSQL password;
- no connection string;
- no Pairing Code;
- no WpfJwt;
- no access token;
- no refresh token;
- no raw credential;
- no customer/employee data.

No database mutation occurred. No OBM source was committed or pushed. Only this coordination report is intended to be committed and pushed.

## 19. Coordination commit SHA

Final pushed coordination commit SHA is reported in the Codex final response; embedding the commit's own SHA into this file would change that SHA.
