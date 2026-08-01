# REPORT-042 - Close Persistent Development ProductRoot Handoff Failure

## 1. Verdict

OBM_POS_EFFECTIVE_PRODUCTROOT_HANDOFF_READY_FOR_USER_RETEST

## 2. Physical Prompt041 Failure Evidence

The operator physically retested after prompt041 and still received:

`DevelopmentDatabaseRejected`

with the generic Development guard message requiring an explicit isolated `SPACEPOS_PRODUCT_ROOT` and rejecting ProgramData fallback.

The approved lane remains:

- ProductRoot: `E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot`
- Database: `obm_pos_dev_v0_pg`
- Environment: `Development`
- Phase 2 v002: Complete
- RuntimeState: `Activated`

## 3. Stale/Current Binary Proof

No active NailSalon/InstallationV0 WPF process was observed during prompt042, so loaded assembly paths could not be physically inspected.

Current rebuilt output evidence:

- `E:\Project2026\4POS\NailSalonNet8\bin\net8.0-windows\NailSalonNet8.dll`
  - SHA-256: `19274FCF03ED60F56437C0440577807129F8550AD72769AF378455EB5F2E6555`
  - LastWriteTimeUtc: `2026-08-01 13:24:54Z`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\bin\net8.0-windows\InstallationV0.dll`
  - SHA-256: `79AE705296FD3240D11BB934EDB26974C943D32ED2FAB051911FF57C9367AE05`
  - LastWriteTimeUtc: `2026-08-01 13:24:42Z`

Current source build label is `prompt042`; prompt041 stale binary was not confirmed because no live WPF process was present for inspection.

## 4. Exact Rejection Call Site And StageId

The physical dialog text maps to:

- Class: `DevelopmentProfileLaunchPolicy`
- Method: `EvaluateStartupGuard`
- Call sites:
  - `App.xaml.cs` pre-configuration guard
  - `App.xaml.cs` post-configuration guard
- StageId after correction: `DevelopmentStartupGuard`

Prompt041 handoff happened after App construction, but at least one downstream reader still depended on process environment state rather than only the in-memory override.

## 5. Root Resolver And Caching Root Cause

Prompt041 passed the verified root through the Open OBM-POS callback and set `SpacePosConfigurationPaths.OverrideProductRoot`, but legacy readiness/bootstrap components still had paths that read `SPACEPOS_PRODUCT_ROOT` or had already evaluated a guard before the handoff path.

Prompt042 correction:

- creates an authoritative effective ProductRoot context;
- invalidates `_startupAssessment` before retry;
- sets the effective in-memory root;
- sets `SPACEPOS_PRODUCT_ROOT` for `EnvironmentVariableTarget.Process` only;
- reruns readiness through a fresh assessment path.

## 6. Canonical EffectiveProductRoot Resolver Design

Added `EffectiveProductRootContext` with:

- `EffectiveProductRoot`
- `EffectiveProductRootSource`
- `ResultCode`
- `RootPresent`
- `RootApproved`
- `HandoffProfileMismatch`

Precedence:

1. `InstallationHandoff`
2. `LaunchProfileEnvironment`
3. `Missing`
4. `Rejected`

Result codes include:

- `DEVELOPMENT_PRODUCT_ROOT_MISSING`
- `DEVELOPMENT_PRODUCT_ROOT_NOT_APPROVED`
- `DEVELOPMENT_PRODUCT_ROOT_HANDOFF_PROFILE_MISMATCH`
- `DEVELOPMENT_PRODUCT_ROOT_CACHED_ASSESSMENT_STALE`

## 7. Process-Scope Environment Bridge Proof

After strict validation of the handoff ProductRoot, `App.xaml.cs` now performs:

- `EffectiveProductRootContext.SetInstallationHandoffProductRoot(...)`
- `SpacePosConfigurationPaths.OverrideProductRoot = normalized`
- `Environment.SetEnvironmentVariable(..., EnvironmentVariableTarget.Process)`
- readback of both `SpacePosConfigurationPaths.ProductRoot` and process env value

No User or Machine environment variable is written.

## 8. Service Construction And Cache Ordering Correction

The same-process handoff now applies the root before readiness retry:

`InstallationV0Window -> RequestOpenObmPosAsync(_service.ProductRoot) -> OpenInstalledPosFromInstallationV0Async -> TryApplyVerifiedInstallationV0ProductRoot -> set context/process env -> _startupAssessment = null -> RetryStartupAssessmentAsync`

The existing cached `DevelopmentDatabaseRejected` assessment is not reused.

## 9. Runtime Development Profile Proof

`E:\Project2026\4POS\NailSalonNet8\Properties\launchSettings.json` contains:

- profile: `OBM-POS Runtime Development`
- `SPACEPOS_PRODUCT_ROOT=E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot`
- `DOTNET_ENVIRONMENT=Development`
- no `SPACEPOS_INSTALLATION_MODULE` in the runtime profile section

## 10. Same-Process Open OBM-POS Sequence

Corrected sequence:

1. InstallationV0 window obtains the service-resolved ProductRoot.
2. App validates the normalized root against the approved V0 root.
3. App sets effective context and process-scoped env bridge.
4. App clears cached startup assessment.
5. App reruns InstalledHealthy readiness.
6. App opens exactly one `MainWindow`.
7. App assigns `Application.Current.MainWindow`.
8. InstallationV0 diagnostics window closes on success.

## 11. Development Guard Remains Fail-Closed

The guard still rejects:

- null or empty ProductRoot;
- ProgramData fallback;
- forbidden production/Royal/2Platform paths;
- another unapproved lane;
- handoff/profile disagreement;
- protected database names.

The fix does not enable ProgramData fallback.

## 12. Source Files Changed

Source files changed by this prompt:

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0Module.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\DevelopmentProfileLaunchPolicy.cs`
- `E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\EffectiveProductRootContext.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\R4DevelopmentProfileLauncherTests.cs`

## 13. Build And Test Counts

Commands run:

- `dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj`
  - PASS
  - 0 warnings
  - 0 errors
- `dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj`
  - PASS
  - 0 warnings in final incremental build
  - 0 errors
- `dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap"`
  - PASS
  - 94 passed
  - 0 failed
  - 0 skipped

## 14. Read-Only DB No-Delta Proof

PostgreSQL was queried inside `BEGIN TRANSACTION READ ONLY` followed by `ROLLBACK`.

Observed:

- `TblEmployeePermission=7`
- `TblEmployee=20`
- `TblLocalOutbox=62`
- `Phase2TrialCompletionMarker v002=1`
- `RuntimeState=Activated`

No seed, migration, pairing redeem, marker rewrite, outbox insert, employee update, or runtime-history transition was performed.

## 15. Prompt042 Label Proof

`InstallationV0BuildInfo.CoordinationPromptLabel` is now:

`prompt042`

The window title remains derived from the label:

`OBM InstallationV0 Phase 1/2 - prompt042`

## 16. Operator Retest Steps

Route A:

1. Stop any old WPF debugging session.
2. In Visual Studio, set `NailSalonNet8` as startup project.
3. Select `OBM-POS Runtime Development`.
4. Press F5.
5. Expected: MainWindow opens directly.
6. Expected: no `DevelopmentDatabaseRejected`.
7. Expected: setup diagnostics does not open.

Route B:

1. Start the InstallationV0 diagnostics profile.
2. Verify `Phase 2 Local DB Baseline: Phase 2 v002 Complete`.
3. Verify ProductRoot is `E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot`.
4. Click `Open OBM-POS`.
5. Expected: one MainWindow opens in the same process.
6. Expected: InstallationV0Window closes.
7. Expected: no `DevelopmentDatabaseRejected`.

## 17. No Secrets / No Source Push Confirmation

- No secrets, tokens, passwords, connection strings, or protected credentials were printed.
- No WPF physical launch was performed.
- No database mutation was performed.
- No Platform mutation was performed.
- OBM source was not committed or pushed by this prompt.
- Only this coordination report is intended to be committed and pushed.

## 18. Coordination Commit SHA

Final pushed commit SHA is returned by Codex after commit and push. Embedding it here would change the commit hash.
