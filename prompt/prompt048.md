# Prompt 048 — Close silent `Open OBM-POS` same-process handoff failure

## Physical operator evidence

Read completely before changing source:

```text
report/report045.md
report/report047.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The operator rebuilt and physically retested the current diagnostics build.

Visible proof:

```text
Build label: prompt047
Phase 2 Local DB Baseline: Phase 2 v002 Complete
Target DB: obm_pos_dev_v0_pg (Development/Test)
ProductRoot: E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
Pairing Code is not required
```

The operator clicked:

```text
Open OBM-POS
```

Observed result:

```text
- no Development database rejection dialog;
- no other error dialog;
- no MainWindow visible;
- InstallationV0Window remains open;
- screen appears unchanged;
- repeated clicking must not be required.
```

This proves prompt047 closed the visible database-approval rejection, but the same-process UI handoff still fails silently or returns without completing the transition.

Do not weaken local-DB readiness. Do not reintroduce Phase1/Phase2 detailed proof as normal runtime authentication. Do not mutate PostgreSQL. Do not change employee PINs, API tokens, roles, passwords, Pairing Codes, or User/Machine environment variables.

## Objective

Make this route physically deterministic:

```text
InstallationV0 diagnostics
-> click Open OBM-POS once
-> show visible in-progress state
-> run the same prompt045 minimal local-DB-first readiness assessment
-> create and show exactly one MainWindow
-> assign Application.Current.MainWindow
-> close InstallationV0Window only after MainWindow is visibly shown
```

If any step fails, the diagnostics window must remain open and show a precise sanitized failure result. Silent return is not allowed.

## First task — trace the complete physical click pipeline

Audit exact event and callback flow, including:

```text
Open OBM-POS Button.Click binding / handler
InstallationV0Window request method
RequestOpenObmPosAsync callback
InstallationV0Module callback wiring
App.OpenInstalledPosFromInstallationV0Async or current equivalent
TryApplyVerifiedInstallationV0ProductRoot
DevelopmentStartupGuard
ApplicationStartupCoordinator / RuntimeProfileStartupAssessmentService
readiness result mapping
MainWindow construction
MainWindow.Show
Application.Current.MainWindow assignment
InstallationV0Window.Close
exception handling
async/await return values
Dispatcher usage
ShutdownMode
```

For every stage, report:

```text
Stage entered
Stage completed
Returned boolean/result
Exception type classification, if any
Readiness classification/result code
Current dispatcher thread status
Current visible window count/type
```

Do not print connection strings, passwords, tokens, PINs, Pairing Codes, or private identity values.

## Likely fault classes — verify, do not assume

Investigate all of these:

1. The button handler is wired but ignores a `false` callback result.
2. The callback returns `false` after readiness without surfacing the result.
3. Readiness returns a non-healthy result that is not displayed.
4. `MainWindow` construction throws and an `async void` or broad catch swallows the exception.
5. `MainWindow.Show()` is never reached.
6. `MainWindow` is created behind the diagnostics window or immediately closed.
7. `Application.Current.MainWindow` remains the diagnostics window and WPF shutdown/window ownership behavior prevents the new window from remaining visible.
8. `InstallationV0Window` has an owner/modal relationship that blocks the new MainWindow.
9. `Dispatcher.Invoke/BeginInvoke` ordering returns before the window is shown.
10. A re-entrancy or `_isOpening` guard remains latched from a previous attempt.
11. The button command is disabled/re-enabled incorrectly while the task never completes.
12. Post-MainWindow API/bootstrap code throws before visual transition completes.
13. A `Task.Run` or non-UI-thread continuation attempts to construct/show WPF UI.
14. The app opens MainWindow off-screen or minimized, while diagnostics remains active.

Identify the exact physical root cause and exact source line/branch.

## Required visible state machine

Implement a small explicit handoff state visible in the diagnostics window:

```text
Idle
Opening
Opened
Failed
```

Behavior:

```text
Idle:
  button enabled

Opening:
  button disabled
  button text = Opening OBM-POS...
  status text shows current sanitized stage

Opened:
  MainWindow shown
  diagnostics closes

Failed:
  diagnostics remains open
  button re-enabled
  exact safe ResultCode and StageId shown
```

Do not require users to inspect Visual Studio Output to understand a failed click.

## Same-process window transition contract

Use one UI-thread sequence:

```text
await minimal local readiness
construct MainWindow on Dispatcher UI thread
set Application.Current.MainWindow = mainWindow
show/activate mainWindow
verify IsVisible == true
only then close InstallationV0Window
```

Recommended ordering to verify against current WPF behavior:

```csharp
var mainWindow = new MainWindow(...);
Application.Current.MainWindow = mainWindow;
mainWindow.Show();
mainWindow.Activate();
installationWindow.Close();
```

Do not close diagnostics before successful MainWindow construction/show.

Do not create a second process.

Do not call `Application.Shutdown()` during the handoff.

## Readiness contract to preserve

The same-process handoff must use prompt045 local-DB-first readiness only:

```text
- resolved approved ProductRoot/config lane;
- canonical Development DB = obm_pos_dev_v0_pg;
- PostgreSQL authentication succeeds;
- schema version ready;
- one usable runtime profile;
- RuntimeState = Activated;
- local Tenant/POS/profile identity consistent.
```

Do not require as ordinary runtime gates:

```text
- WpfJwt validity;
- API reachability;
- Pairing Code state;
- exact employee count;
- exact permission count;
- exact outbox count;
- employee operational PIN configuration;
- launch provenance after Development lane approval.
```

InstallationV0 may still show full Phase2 proof in diagnostics, but the handoff target readiness remains minimal local readiness.

## API independence

MainWindow must become visible before any API/auth/sync continuation can fail.

If API bootstrap, SignalR, or sync startup fails afterward:

```text
- MainWindow stays open;
- local mode becomes OfflineDeferred;
- no modal startup error;
- diagnostics does not reopen;
- local CRUD remains available.
```

## Diagnostics and logging

Add safe stage/result diagnostics, for example:

```text
OPEN_POS_CLICK_RECEIVED
OPEN_POS_PRODUCT_ROOT_APPLIED
OPEN_POS_DATABASE_APPROVED
OPEN_POS_LOCAL_READINESS_READY
OPEN_POS_MAINWINDOW_CONSTRUCTED
OPEN_POS_MAINWINDOW_SHOWN
OPEN_POS_DIAGNOSTICS_CLOSED
```

Failure examples:

```text
OPEN_POS_READINESS_FAILED
OPEN_POS_MAINWINDOW_CONSTRUCTION_FAILED
OPEN_POS_MAINWINDOW_SHOW_FAILED
OPEN_POS_UI_THREAD_REQUIRED
OPEN_POS_CALLBACK_RETURNED_FALSE
```

Never log raw credentials, connection strings, tokens, PINs, Pairing Codes, or private identity values.

## Build label

Set diagnostics build label and title to:

```text
Build label: prompt048
OBM InstallationV0 Phase 1/2 - prompt048
```

## Focused tests

Add tests covering at least:

```text
button click invokes callback exactly once
second click while Opening is ignored
callback false produces visible Failed state
readiness failure produces exact safe status
MainWindow constructor exception is surfaced safely
MainWindow Show exception is surfaced safely
MainWindow is constructed on UI thread
Application.Current.MainWindow changes to the POS MainWindow
InstallationV0Window closes only after MainWindow IsVisible
successful click opens exactly one MainWindow
no second process is started
API bootstrap exception after Show does not close MainWindow
local DB ready does not require API/token/PIN/Phase exact counts
Development protected DB/root remains rejected
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap|FullyQualifiedName~DevelopmentProfile|FullyQualifiedName~OpenObmPos" -v minimal
```

## Database safety

Use only read-only evidence if DB inspection is required:

```sql
BEGIN TRANSACTION READ ONLY;
-- SELECT only
ROLLBACK;
```

Do not run seed, migration, insert, update, delete, role, GRANT/REVOKE, or marker operations.

## Physical operator retest

After builds/tests pass, leave WPF physical launch to the operator.

Expected test:

```text
1. Start prompt048 InstallationV0 diagnostics.
2. Confirm Phase2 v002 Complete and canonical DB/root.
3. Click Open OBM-POS once.
4. See Opening OBM-POS... briefly.
5. Exactly one MainWindow becomes visible.
6. InstallationV0Window closes.
7. No API requirement and no Pairing Code request.
```

If the handoff fails, the prompt048 diagnostics UI must show exact safe `ResultCode` and `StageId`; it must never remain visually unchanged.

## Report 048

Create and push:

```text
report/report048.md
```

Required sections:

1. Verdict.
2. Physical prompt047 silent-failure evidence.
3. Exact click/callback pipeline before correction.
4. Exact root cause and swallowed/ignored result branch.
5. Visible state-machine implementation.
6. UI-thread and window ownership/lifetime correction.
7. Minimal local-readiness preservation.
8. API-offline independence.
9. Exact source files changed.
10. Build/test commands and counts.
11. Read-only DB evidence.
12. Safe diagnostics/result codes added.
13. Exact operator physical retest steps.
14. No secrets/no DB mutation/no source push proof.
15. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_SAME_PROCESS_MAINWINDOW_HANDOFF_READY_FOR_USER_RETEST
```

```text
BLOCKED_OBM_POS_SAME_PROCESS_MAINWINDOW_HANDOFF
```
