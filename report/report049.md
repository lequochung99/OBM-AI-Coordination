# Report 049 — Canonical POS Runtime Status Router

## 1. Verdict

OBM_POS_RUNTIME_STATE_ROUTER_READY_FOR_USER_RETEST

Scope completed for prompt049: the bool-only InstallationV0 handoff is replaced with a structured startup route result, the normal local startup predicate is aligned to the physical V0 runtime profile schema, and focused build/test evidence is green. No WPF physical launch was performed because prompt049 reserves physical retest for the operator.

## 2. Physical prompt048 failure evidence

Operator evidence from prompt049/report context:

- Build label observed: `prompt048`.
- Phase 2 Local DB Baseline observed: `Phase 2 v002 Complete`.
- Target DB observed: `obm_pos_dev_v0_pg`.
- ProductRoot observed: `E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot`.
- Pairing Code was not required.
- Clicking `Open OBM-POS` once produced `StageId=InstallationV0OpenObmPos` and `ResultCode=OPEN_POS_CALLBACK_RETURNED_FALSE`.
- No MainWindow became visible.

Conclusion: the prompt048 wrapper correctly observed callback reachability, but it hid the real local startup route decision.

## 3. Read-only physical runtime-status evidence

Physical target was queried read-only with a read-only PostgreSQL transaction and rollback. No credential, connection string, token, private GUID, employee data, or raw payload was printed.

Sanitized evidence:

- Database: `obm_pos_dev_v0_pg`.
- `dbo.TblPosRuntimeProfile` row count: `1`.
- RuntimeState exact value: `Activated`.
- Runtime profile identity keys present: yes.
- Tenant/POS identity keys present: yes.
- DatabaseName: `obm_pos_dev_v0_pg`.
- EnvironmentName: `Development`.
- SchemaVersion: `20260726.018`.
- SourceClientId shape: valid `POS:*` shape.
- Matching `dbo.TblTenant` count: `1`.
- Matching `dbo.TblPosLocal` count: `1`.
- Latest history summary: previous state empty/null -> `Activated`, reason `DEVELOPMENT_SYNTHETIC_PROFILE_READY`, timestamp present.
- Transaction ended with `ROLLBACK`.

## 4. Exact actual runtime-state enum/value inventory

Source enum `PosRuntimeState` contains exactly:

- `Activated`
- `Disabled`
- `Installing`
- `RecoveryRequired`

Physical value found in V0 database: `Activated`.

## 5. Exact underlying callback false predicate/root cause

Root cause:

- Physical runtime profile had `RuntimeState=Activated` and local identity was internally ready.
- Source required `MinimumSupportedSchemaVersion = 20260730.004` before prompt049.
- Physical V0 runtime profile has `SchemaVersion = 20260726.018`.
- `RuntimeProfileStartupAssessmentService.IsSupportedSchemaVersion(...)` therefore returned false.
- The assessment routed to `SchemaUpdateRequired`, not `InstalledHealthy`.
- `App.OpenInstalledPosFromInstallationV0Async` saw `assessment.CanEnterNormalApplication == false` and returned a failure to the InstallationV0 wrapper.
- The wrapper surfaced only `OPEN_POS_CALLBACK_RETURNED_FALSE`, masking the local result.

Required root-cause evidence format:

- UnderlyingResultCode: `POS_RUNTIME_SCHEMA_UPDATE_REQUIRED` before fix; structured result now preserves this instead of collapsing to bool false.
- UnderlyingStageId: `PosStartupRouter` / runtime readiness handoff.
- StartupDecision: `OpenRecovery` before schema correction; `OpenMainPos` after schema correction when local profile is healthy.
- RuntimeState: `Activated`.
- LocalRuntimeReady: false before fix because schema gate rejected the physical schema; true after correction.
- MainWindowConstructionAttempted: false in the prompt048 failure path.
- MainWindowShowAttempted: false in the prompt048 failure path.
- MainWindowVisible: false in the prompt048 failure path.
- FalseReturnPredicate: `assessment.CanEnterNormalApplication == false`, followed by `Current.MainWindow is MainWindow { IsVisible: true }` evaluating false.

## 6. Pre-change startup and handoff decision trees

Pre-change InstallationV0 handoff:

`InstallationV0Window.OpenPosAsync -> InstallationV0Module.RequestOpenObmPosAsync -> App.OpenInstalledPosFromInstallationV0Async -> RetryStartupAssessmentAsync -> RuntimeProfileStartupAssessmentService -> bool returned to UI`

The final UI only knew true/false. It did not preserve runtime state, local readiness, route decision, or MainWindow visibility evidence.

Pre-change normal startup:

`App startup -> startup assessment -> runtime mode gate -> diagnostics/recovery or normal startup`

The assessment already read local runtime status, but the schema version mismatch blocked the physically valid V0 runtime profile.

## 7. Structured startup result contract

Added `PosStartupRouteResult` with:

- `RouteDecision`
- `ResultCode`
- `StageId`
- `SafeMessage`
- `RuntimeState`
- `LocalRuntimeReady`
- `MainWindowConstructed`
- `MainWindowShown`
- `MainWindowVisible`

Route decisions:

- `OpenMainPos`
- `OpenInstallation`
- `OpenRecovery`
- `Blocked`

Success is now derived from `RouteDecision=OpenMainPos` and `MainWindowVisible=true`.

## 8. Canonical shared local startup router

The shared local DB first assessment remains `RuntimeProfileStartupAssessmentService` and is used through `ApplicationStartupCoordinator` / `RetryStartupAssessmentAsync`.

Prompt049 changes align both routes:

- Direct normal WPF startup returns a structured `PosStartupRouteResult` from `StartNormalApplicationAsync`.
- InstallationV0 `Open OBM-POS` calls the same local assessment/startup flow and receives the same structured route result.
- InstallationV0 detailed proof remains installation/audit UI; it is not the normal MainWindow router.

## 9. Runtime-state route table

Existing source states:

| RuntimeState | Route | Result behavior |
| --- | --- | --- |
| `Activated` + schema/identity/local DB ready | `OpenMainPos` | `OPEN_POS_MAINWINDOW_SHOWN` after MainWindow visible |
| `Installing` | `OpenInstallation` | `POS_RUNTIME_ROUTE_INSTALLATIONINCOMPLETE` |
| `RecoveryRequired` | `OpenRecovery` | `POS_RUNTIME_ROUTE_RECOVERYREQUIRED` |
| `Disabled` | `OpenRecovery` | `POS_RUNTIME_ROUTE_DISABLED` |
| Unknown/unparseable persisted value | blocked/fail closed | `POS_RUNTIME_STATE_UNKNOWN` remains deferred hardening; current enum parse does not silently reinterpret values |

## 10. Direct Runtime Development behavior

`StartNormalApplicationAsync` now returns structured route evidence while preserving direct normal startup as primary. When local assessment is `InstalledHealthy`, MainWindow is constructed, assigned to `Application.Current.MainWindow`, shown, activated, and verified visible before success is returned.

## 11. InstallationV0 Open OBM-POS behavior

`InstallationV0Module.OpenObmPosRequested` now uses `Func<string, Task<PosStartupRouteResult>>` instead of `Func<string, Task<bool>>`.

`InstallationV0Window.OpenPosAsync` keeps the diagnostics window open unless the structured route reports success. Blocked routes preserve the underlying `ResultCode`, `StageId`, route decision, runtime state, and MainWindow visibility flags.

## 12. MainWindow lifecycle/shutdown correction

MainWindow route behavior now records:

- UI dispatcher requirement.
- existing visible MainWindow activation success.
- construction start and completion.
- `Show()` and `Activate()`.
- `IsVisible=true` verification.
- post-MainWindow failures as deferred local mode, not route reversal.

The diagnostics window is only closed by the UI when the structured handoff result succeeds.

## 13. API-offline independence

Local startup no longer depends on API reachability, WpfJwt validity, pairing code, employee PIN normalization, employee/permission counts, outbox counts, Phase1 checkpoint, or full Phase2 proof counts.

After MainWindow is visible, API/bootstrap/token/sync failures remain deferred local-mode work and do not convert a successful local route into an open failure.

## 14. Exact source files changed

Source files changed in the OBM tree for prompt049:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\PosStartupRouteResult.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0Module.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\RuntimeProfileStartupAssessmentService.cs`
- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\Startup\RuntimeProfileStartupAssessmentServiceTests.cs`

No OBM source was committed or pushed.

## 15. Build/test commands and counts

Commands run:

```powershell
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal -clp:ErrorsOnly
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal -clp:ErrorsOnly
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap|FullyQualifiedName~DevelopmentProfile|FullyQualifiedName~OpenObmPos|FullyQualifiedName~RuntimeState" -v minimal -clp:ErrorsOnly
```

Results:

- InstallationV0 build: passed, 0 warnings, 0 errors.
- WPF build: passed, 176 warnings, 0 errors.
- Filtered tests: passed, 132 passed, 0 failed, 0 skipped.

## 16. Read-only/no-mutation DB proof

The PostgreSQL verification used `BEGIN TRANSACTION READ ONLY` and completed with `ROLLBACK`.

No seed, migration, marker rewrite, runtime-profile update, runtime history insert, outbox insert, pairing redemption, role/password change, or employee PIN change was performed.

## 17. Prompt049 label proof

`InstallationV0BuildInfo.CoordinationPromptLabel` is now `prompt049`.

The InstallationV0 title remains composed as:

`OBM InstallationV0 Phase 1/2 - {InstallationV0BuildInfo.CoordinationPromptLabel}`

The build label text remains composed as:

`Build label: {InstallationV0BuildInfo.CoordinationPromptLabel}`

## 18. Exact operator physical retest steps

Route A:

1. Start `NailSalonNet8` with the standard `OBM-POS Runtime Development` profile.
2. Confirm it uses `E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot`.
3. Confirm normal MainWindow opens directly.
4. Confirm no InstallationV0 window is required first.

Route B:

1. Start InstallationV0 diagnostics build `prompt049`.
2. Confirm the label shows `prompt049`.
3. Click `Open OBM-POS` once.
4. Require structured result `RouteDecision=OpenMainPos`, `ResultCode=OPEN_POS_MAINWINDOW_SHOWN`, `RuntimeState=Activated`, `LocalRuntimeReady=True`, `MainWindowVisible=True`.
5. Confirm MainWindow remains open after diagnostics closes.

Route C:

1. Make API unavailable without changing the local PostgreSQL database.
2. Start `OBM-POS Runtime Development`.
3. Confirm MainWindow still opens from local runtime state in offline/deferred mode.

## 19. Deferred work

Deferred by prompt049:

- refresh token handling;
- PIN normalization;
- legacy Identity cleanup;
- explicit unknown persisted runtime-state result code hardening beyond current enum parse behavior;
- operator physical WPF retest.

## 20. No secrets/no source push confirmation

No secrets, credentials, raw GUIDs, tokens, connection strings, customer data, employee data, or raw PostgreSQL password values are included in this report.

No OBM source was committed or pushed. Only this coordination report is intended for the coordination repository.

## 21. Coordination commit SHA

Final pushed coordination commit SHA is reported in the Codex final response; the SHA cannot be self-embedded into the same commit content without changing the commit hash.

