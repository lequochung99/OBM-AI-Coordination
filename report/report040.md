# REPORT 040 - InstallationV0 to Main OBM-POS Startup Handoff

## 1. Verdict

OBM_POS_MAIN_STARTUP_HANDOFF_READY_FOR_USER_TEST

Prompt: `prompt/prompt040.md`

## 2. Exact Executable / Project Distinction

Source audit proved two Visual Studio launch routes in `E:\Project2026\4POS\NailSalonNet8\Properties\launchSettings.json`:

- `OBM-POS InstallationV0 Phase1` sets `SPACEPOS_INSTALLATION_MODULE=InstallationV0`; opening `InstallationV0Window` is expected for this diagnostic harness route.
- `OBM-POS Runtime Development` is the normal POS route and now points to the completed V0 ProductRoot without setting `SPACEPOS_INSTALLATION_MODULE`.

The normal POS project is:

`E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj`

The installation library/harness project is:

`E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj`

Codex did not physically launch WPF in this prompt, so the operator's active executable path was not observed live.

## 3. Startup Decision Tree Before Correction

Before prompt040 correction:

1. `App.xaml.cs` acquired the single-instance mutex.
2. If `SPACEPOS_INSTALLATION_MODULE=InstallationV0`, it opened `InstallationV0Window`.
3. Otherwise it ran the runtime startup assessment.
4. The runtime assessment considered an activated runtime profile sufficient for `InstalledHealthy`.
5. The main app did not verify the Phase 1 checkpoint plus Phase 2 v002 marker/invariants before normal startup.
6. If bootstrap mode was selected, `StartNormalApplicationAsync` returned without a clear installer/recovery handoff.

## 4. Root Cause

The persistent installer experience had two causes:

- using the InstallationV0 Visual Studio launch profile intentionally opens the installer;
- the main startup readiness predicate did not include the completed InstallationV0 v002 proof, so the source had no canonical handoff from completed InstallationV0 diagnostics into normal `MainWindow`.

## 5. InstalledHealthy Predicate

For the InstallationV0 V0 lane, normal startup now requires read-only proof of:

- ProductRoot is the V0 development ProductRoot lane;
- database locator is `obm_pos_dev_v0_pg`;
- environment is `Development`;
- Phase 1 `ApiAuthorized` checkpoint exists and is safe;
- `dbo.Phase2TrialCompletionMarker` contains `phase2-reference-driven-trial-v002-employees` exactly once;
- `TblTenant` matches Phase 1 identity;
- `TblPosLocal` matches Phase 1 POS/slot;
- `TblPosRuntimeProfile` matches Phase 1 identity and is `Activated`;
- v002 counts exist: 7 permissions and 20 starter employees;
- v002 canonical outbox evidence exists: 4 permission rows and 20 employee rows.

`InstalledHealthy` remains an assessment result, not a runtime enum.

## 6. ProductRoot / Database Locator Proof

The normal Visual Studio profile now includes:

- `SPACEPOS_PRODUCT_ROOT=E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot`
- no `SPACEPOS_INSTALLATION_MODULE`
- `DOTNET_ENVIRONMENT=Development`

Read-only DB proof after changes:

- `TblEmployeePermission=7`
- `TblEmployee=20`
- `TblLocalOutbox=62`
- v002 marker count = 1
- `RuntimeState=Activated`

## 7. MainWindow Handoff Implementation

Changed `App.xaml.cs` so normal startup:

- records whether the InstallationV0 diagnostic profile was requested;
- runs startup assessment before opening normal POS;
- routes `InstalledHealthy` to `MainWindow`;
- assigns `Application.Current.MainWindow = win`;
- routes incomplete/inconsistent startup to `InstallationV0Window` instead of silently returning.

## 8. InstallationV0 Open OBM-POS Diagnostic Handoff

`InstallationV0Window` now shows a non-mutating `Open OBM-POS` button after Phase 2 v002 hydration is complete.

The button calls the main-process readiness handler:

- reruns startup assessment;
- requires the same `InstalledHealthy` predicate;
- opens `MainWindow` in the same process when healthy;
- closes the diagnostic window;
- shows a safe blocked message if readiness is not healthy.

## 9. Fail-Closed Behavior

The V0 lane fails closed as `InstallationIncomplete` if:

- Phase 1 checkpoint is missing or invalid;
- ProductRoot/database/environment do not match the V0 lane;
- v002 marker is absent or duplicated;
- identity spine does not match;
- runtime state is not `Activated`;
- permissions/employees/outbox evidence is incomplete.

The main app does not rerun Phase 2, redeem pairing codes, or mutate outbox/markers during startup.

## 10. Source Files Changed

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\Properties\launchSettings.json`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\InstallationV0CompletedReadinessService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\RuntimeProfileStartupAssessmentService.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0Module.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

## 11. Builds And Tests

Executed:

- `dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj`
  - result: PASS
  - warnings: 0
  - errors: 0
- `dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj`
  - result: PASS
  - final incremental warnings: 0
  - errors: 0
- `dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap"`
  - result: PASS
  - passed: 94
  - failed: 0
  - skipped: 0

## 12. No DB Mutation / Outbox / Marker Delta Proof

Read-only verification was executed in a `BEGIN TRANSACTION READ ONLY` transaction followed by `ROLLBACK`.

Post-change state remained:

- `TblEmployeePermission=7`
- `TblEmployee=20`
- `TblLocalOutbox=62`
- v002 marker count = 1
- `RuntimeState=Activated`

No seed, migration, marker rewrite, pairing redeem, or outbox insert was run.

## 13. Operator Test Steps

1. Stop debugging.
2. In Visual Studio, right-click `NailSalonNet8`.
3. Select `Set as Startup Project`.
4. Choose `OBM-POS Runtime Development`.
5. Confirm it points to `E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot`.
6. Press F5.
7. Verify the launched binary is `NailSalonNet8.exe`, not an InstallationV0-only diagnostic route.
8. Confirm `MainWindow` opens instead of `InstallationV0Window`.

## 14. Main POS Smoke Steps

After `MainWindow` opens:

- confirm DB lane is `obm_pos_dev_v0_pg`;
- confirm tenant/POS is `OBMDEVV0 / POS1`;
- open employee management and verify the 20 starter employees load;
- open checkout/staff selector and verify existing Staff filtering works;
- close and reopen and confirm normal startup repeats;
- verify no outbox/marker delta appears merely from opening POS.

## 15. Active InstallationV0 Label Proof

InstallationV0 diagnostics now use:

- build label: `prompt040`
- window title: `OBM InstallationV0 Phase 1/2 - prompt040`

## 16. Safety

- No WPF process was launched by Codex in prompt040.
- No database mutation was performed.
- No secret, password, token, pairing code, JWT, connection string, or raw protected credential was printed.
- OBM source was not committed or pushed.

## 17. Coordination Commit SHA

Final pushed commit SHA is reported by Codex after commit/push. Embedding it here would change the commit hash.

