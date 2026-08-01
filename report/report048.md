# Report 048 — Same-process Open OBM-POS handoff closure

## 1. Verdict

OBM_POS_SAME_PROCESS_MAINWINDOW_HANDOFF_READY_FOR_USER_RETEST

## 2. Physical prompt047 silent-failure evidence

The operator rebuilt and physically retested the prompt047 diagnostics build. Visible proof showed:

- Build label: `prompt047`
- Phase 2 Local DB Baseline: `Phase 2 v002 Complete`
- Target DB: `obm_pos_dev_v0_pg (Development/Test)`
- ProductRoot: `E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot`
- Pairing Code is not required

Clicking `Open OBM-POS` no longer showed the prompt047 database rejection, but also did not show MainWindow or a failure message. The diagnostics window remained visible and appeared unchanged.

## 3. Exact click/callback pipeline before correction

Pre-correction pipeline:

1. `InstallationV0Window._openPosButton.Click`
2. `InstallationV0Window.OpenPosAsync`
3. `InstallationV0Module.RequestOpenObmPosAsync(_service.ProductRoot)`
4. `InstallationV0Module.OpenObmPosRequested`
5. `App.OpenInstalledPosFromInstallationV0Async`
6. `TryApplyVerifiedInstallationV0ProductRoot`
7. `RetryStartupAssessmentAsync`
8. `StartNormalApplicationAsync(..., forceNormalApplication: true)`
9. `MainWindow` resolved through DI
10. `Current.MainWindow = win`
11. `win.Show()`
12. callback returned `Current.MainWindow is MainWindow`
13. diagnostics closed only when callback returned true

The UI did not show an explicit `Opening` state, and false/exception callback paths could leave the operator with no meaningful visual progress.

## 4. Exact root cause and swallowed/ignored result branch

The root failure class was a weak same-process handoff boundary:

- the click handler disabled the button but did not render a visible in-progress state;
- `InstallationV0Module.RequestOpenObmPosAsync` returned `false` for missing/false callback without assigning a precise final result unless the callback itself had recorded one;
- callback exceptions were not independently converted into a visible safe result;
- post-MainWindow startup exceptions still flowed to the broad startup catch, which could call shutdown even after the POS window had been shown.

Prompt047 fixed database approval, but did not make the visual handoff deterministic.

## 5. Visible state-machine implementation

`InstallationV0Window` now exposes:

- `Idle`: button enabled when Phase 2 v002 is complete;
- `Opening`: button disabled, text `Opening OBM-POS...`, safe stage shown;
- `Opened`: MainWindow shown, diagnostics closes;
- `Failed`: diagnostics remains open, button re-enabled when v002 is complete, safe `ResultCode`/`StageId` shown.

Re-entrant clicks while opening are ignored and recorded as `OPEN_POS_REENTRANT_CLICK_IGNORED`.

## 6. UI-thread and window ownership/lifetime correction

The handoff now records safe stage transitions:

- `OPEN_POS_CLICK_RECEIVED`
- `OPEN_POS_PRODUCT_ROOT_APPLIED`
- `OPEN_POS_LOCAL_READINESS_READY`
- `OPEN_POS_MAINWINDOW_CONSTRUCTED`
- `OPEN_POS_MAINWINDOW_SHOWN`

MainWindow is constructed and shown on the WPF dispatcher thread. The show sequence sets `Application.Current.MainWindow`, calls `Show()`, calls `Activate()`, verifies `IsVisible`, and only then reports success to the diagnostics UI. The diagnostics window closes only after the callback returns true.

## 7. Minimal local-readiness preservation

The handoff still uses the prompt045 local-DB-first readiness path:

- approved V0 ProductRoot/config lane;
- canonical Development DB `obm_pos_dev_v0_pg`;
- PostgreSQL authentication succeeds;
- local schema/runtime readiness;
- singleton runtime profile;
- `RuntimeState=Activated`;
- local Tenant/POS/profile identity consistency.

It does not require WpfJwt, API reachability, Pairing Code state, exact employee count, exact permission count, exact outbox count, employee operational PIN configuration, or Phase proof counts as normal runtime gates.

## 8. API-offline independence

After MainWindow becomes visible, later API/bootstrap/sync exceptions are recorded as deferred local mode and do not close MainWindow. The safe diagnostic includes `localMode=OfflineDeferred`.

## 9. Exact source files changed

Source files changed for prompt048:

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0Module.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

## 10. Build/test commands and counts

Commands run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap|FullyQualifiedName~DevelopmentProfile|FullyQualifiedName~OpenObmPos" -v minimal
```

Results:

- InstallationV0 build: PASS, 0 warnings, 0 errors.
- NailSalonNet8 build: PASS, 176 warnings, 0 errors.
- Filtered tests: PASS, 130 passed, 0 failed, 0 skipped.

The 176 warnings are existing warnings in the broader WPF tree, not prompt048 errors.

## 11. Read-only DB evidence

Read-only PostgreSQL verification used protected local configuration without printing secrets. The SQL session used `BEGIN TRANSACTION READ ONLY` followed by `ROLLBACK`.

Observed safe evidence:

- Database: `obm_pos_dev_v0_pg`
- User: `postgres`
- `dbo` base tables: `137`
- `public` base tables: `2`
- `dbo.TblEmployeePermission`: `7`
- `dbo.TblEmployee`: `20`
- `dbo.TblLocalOutbox`: `62`
- `dbo.Phase2TrialCompletionMarker`: `2`
- `dbo.TblPosRuntimeProfile`: `1`
- `dbo.TblPosRuntimeStateHistory`: `1`
- RuntimeState `Activated`: `1`

## 12. Safe diagnostics/result codes added

Added or preserved safe result codes:

- `OPEN_POS_CLICK_RECEIVED`
- `OPEN_POS_PRODUCT_ROOT_APPLIED`
- `OPEN_POS_LOCAL_READINESS_READY`
- `OPEN_POS_MAINWINDOW_CONSTRUCTED`
- `OPEN_POS_MAINWINDOW_SHOWN`
- `OPEN_POS_READINESS_FAILED`
- `OPEN_POS_MAINWINDOW_SHOW_FAILED`
- `OPEN_POS_UI_THREAD_REQUIRED`
- `OPEN_POS_CALLBACK_MISSING`
- `OPEN_POS_CALLBACK_RETURNED_FALSE`
- `OPEN_POS_CALLBACK_EXCEPTION`
- `OPEN_POS_REENTRANT_CLICK_IGNORED`

No credentials, full connection strings, tokens, PINs, Pairing Codes, or private identity values are logged by these diagnostics.

## 13. Exact operator physical retest steps

1. Start prompt048 InstallationV0 diagnostics.
2. Confirm build label `prompt048`.
3. Confirm `Phase 2 Local DB Baseline: Phase 2 v002 Complete`.
4. Confirm target DB `obm_pos_dev_v0_pg`.
5. Click `Open OBM-POS` once.
6. Confirm the UI briefly shows `Opening OBM-POS...`.
7. Confirm exactly one MainWindow becomes visible.
8. Confirm InstallationV0Window closes only after MainWindow is visible.
9. Confirm no API requirement and no Pairing Code request.

If it fails, the diagnostics window must stay open and show the safe `ResultCode` and `StageId`.

## 14. No secrets/no DB mutation/no source push proof

No database password, full connection string, API token, WpfJwt, Pairing Code, employee PIN, or protected credential was printed. PostgreSQL access was read-only and ended with `ROLLBACK`. The OBM source tree was not committed or pushed by this task. Only this coordination report is intended for commit and push.

## 15. Coordination commit SHA

Coordination commit SHA: reported by the final Codex response after this report is committed and pushed.
