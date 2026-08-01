# Report 056 - InstallationV0 Service Naming Cleanup

## 1. Verdict

`OBM_POS_SERVICE_NAMES_AND_LEGACY_SHIMS_CLEAN_READY_FOR_USER_RETEST`

## 2. DOCS_READ_BEFORE_CODE_GATE

`DOCS_READ_BEFORE_CODE_GATE=PASS`

Read before source edits:

- `E:\Project2026\4POS\NailSalonNet8\AGENTS.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `report/report054.md`
- `report/report055.md`
- `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\installation-flow-subgraph.json`
- `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\edge-inventory.csv`
- `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\symbol-action-inventory.csv`
- `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\redundant-link-candidates.md`
- `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\dynamic-edge-checklist.md`

Gate hashes matched before edits:

- CanonicalDocVersion: `V001`
- CanonicalDocSha256: `A70BE08FE143A34775A1F71844B7A6B672DDD28C24CB041B841E0604585B3915`
- AgentsSha256: `20CB5060FC42ADDA2DE5EEBF90F4630BFBFCAA9ED195615E0B5B8DA035905114`
- CurrentTaskSha256: `2871956A986F46E96F2C00BF0FAD7B0DE6DCEA02EEC0871159BDB52AED33A652`
- CurrentResultSha256: `7E3093B8E9F4E59B55705D564C71BCEF73F363D5044A792927E366086E3B95F0`

## 3. External-Contract/Reference Audit

No external serialized or binary contract was found that required retaining the old startup/API shim names. DI, constructor, factory, XAML/config, reflection-style search, tests, and Graphify/source checks were updated to final names. Preserved backup files such as `App.xaml.cs.codex-bak-*` were excluded from active-source reasoning.

## 4. Old Name To Final Name/Action Table

| Old name | Final name/action |
| --- | --- |
| `ApplicationStartupCoordinator` | `DELETE` |
| `DatabaseStartupAssessmentService` | `LocalPosStartupService` |
| `DatabaseStartupAssessment` | `LocalPosStartupResult` |
| `DatabaseStartupMode` / `DatabaseStartupState` | `LocalPosStartupDecision` |
| `InstalledHealthy` | `Ready` |
| `BootstrapRepairRequired` | `RecoveryRequired` |
| `POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED` | `POS_RUNTIME_RECOVERY_REQUIRED` |
| `RepairBootstrap` | `RepairLocalRuntime` |
| `NeedsBootstrapRepair` | `NeedsLocalRecovery` |
| `IAppJwtBootstrapper` | `IApiSessionInitializer` |
| `AppJwtBootstrapper` | `ApiSessionInitializer` |
| `InstallationV0CompletedReadinessService` | `InstallationV0VerificationService` |
| `PosStartupRouteResult` | `MainWindowOpenResult` |

## 5. Deleted Files/Types/DI/Tests

Deleted or removed from active use:

- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\ApplicationStartupCoordinator.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\DatabaseStartupAssessment.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\DatabaseStartupAssessmentService.cs`
- `E:\Project2026\4POS\NailSalonNet8\ConnectService\AppJwtBootstrapper.cs`
- `E:\Project2026\4POS\NailSalonNet8\ConnectService\IAppJwtBootstrapper.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\InstallationV0CompletedReadinessService.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\PosStartupRouteResult.cs`

Tests were updated away from compatibility-shim expectations; stale test filename `DatabaseStartupAssessmentTests.cs` was renamed.

## 6. Renamed/Moved Files/Types/Result Models

- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\LocalPosStartupService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\LocalPosStartupResult.cs`
- `E:\Project2026\4POS\NailSalonNet8\ConnectService\ApiSessionInitializer.cs`
- `E:\Project2026\4POS\NailSalonNet8\ConnectService\IApiSessionInitializer.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\InstallationV0VerificationService.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\MainWindowOpenResult.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\Startup\LocalPosStartupResultTests.cs`

## 7. Compatibility Shims Retained

None.

## 8. Final Direct Runtime Path

`Local PostgreSQL usable -> LocalPosStartupService -> MainWindow -> ApiSessionInitializer`.

Evidence: `E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\direct-runtime-path.mmd`.

## 9. Final InstallationV0 Handoff Path

`InstallationV0 proof -> Open OBM-POS -> MainWindowOpenResult -> LocalPosStartupService -> MainWindow`.

Evidence: `E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\installation-handoff-path.mmd`.

## 10. API Initializer Is Post-MainWindow

`App.xaml.cs` resolves and calls `IApiSessionInitializer.StartAsync()` after `ShowMainWindowForActivatedRuntimeAsync(...)` succeeds and after startup validation records `mainWindowOpened: true`.

## 11. Installation Verification Has No Normal-Runtime Caller

`InstallationV0VerificationService` exists as an installation/verification service only. Source scan found active references in the service file and documentation only, with no normal-runtime callsite.

## 12. Runtime PostgreSQL Username Source

Runtime PostgreSQL username remains `hung` in the canonical runtime connection path. No password, connection string, or protected credential was printed in this report.

## 13. Naming/Architecture Regression Guard

Added focused guard:

- `E:\Project2026\4POS\NailSalonNet8.Tests\Naming\CanonicalNamingGuardTests.cs`

The guard fails active source/current docs on old startup/API names and on `employee password` / `manager password` terminology.

## 14. Graphify/Source Evidence

Evidence folder:

`E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001`

Key hashes:

- `README.md`: `53B630EE686BFB26ADC1C82B3908010770F9959344CC56276CE7868BAA02001A`
- `SHA256SUMS.txt`: `98BEB5453A5377F13237D3C85D4B11CF4A8443E87591D4322ED6037C55082E1C`
- `forbidden-name-scan.txt`: `79D2585DD3B1E007D667B73292ECCA82A2A4C536E7CF57687E3EE96630C18998`
- `final-symbol-inventory.csv`: `ADE29BFB09FC3C1605165196E2AB92F2BD674E32A416ED7C1CDE9BD16B250670`
- `graphify\graphify-out\graph.json`: `B24181A1694E8CD66766B310DBC734D173D0FABCF6F6A405A37643106E833EC9`

Graphify completed: 13,445 nodes and 30,860 edges.

Forbidden active-source/current-doc scan result: `ZERO_ACTIVE_MATCHES`.

## 15. Canonical V001

Canonical architecture remains V001. The canonical doc text was updated only to replace forbidden password terminology with operational PIN terminology, so the final hash changed:

- `INSTALLATION_RUNTIME_CANONICAL.md`: `7044A02F29FE349FE531DE6800BA739E6B29EA473B9A867881B283DB8743BC72`
- `AGENTS.md`: `90D09B8058381663A4EF317A5707256FA01B1DCDE5FE9F5BD3D4FA8F3CC10E8B`

No V002 architecture change was introduced.

## 16. History Preservation

Prompt055 current docs were preserved under:

- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V001\CURRENT_TASK_V001.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V001\CURRENT_RESULT_V001.md`

Hashes:

- `CURRENT_TASK_V001.md`: `2871956A986F46E96F2C00BF0FAD7B0DE6DCEA02EEC0871159BDB52AED33A652`
- `CURRENT_RESULT_V001.md`: `7E3093B8E9F4E59B55705D564C71BCEF73F363D5044A792927E366086E3B95F0`

## 17. Updated CURRENT_TASK/CURRENT_RESULT

- `CURRENT_TASK.md`: `093CA0C7D9040D52B94D065944A35DDE53A19C135090A4B42F993B9C5CBBC184`
- `CURRENT_RESULT.md`: `16D6B1DE20FED54988C51DEDEEED2F73086CDD62BD3487B8F1E30C355572A973`

The next task is only the physical MainWindow/local CRUD/API-offline retest and does not authorize ASP.NET Identity deletion.

## 18. Source/Docs/Tests/Project Files Changed

Prompt056 touched WPF startup/API naming files, InstallationV0 handoff files, focused tests, naming guard, current docs, history docs, and evidence artifacts. Important files include:

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\MainWindowOpenResult.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0Module.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8\ConnectService\ApiSessionInitializer.cs`
- `E:\Project2026\4POS\NailSalonNet8\ConnectService\IApiSessionInitializer.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\LocalPosStartupService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\LocalPosStartupResult.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\InstallationV0VerificationService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\RuntimeControl\RuntimeControlServices.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\RuntimeProfile\RuntimeProfileShadowStartupEvaluator.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Recovery\RuntimeRecoveryIdentityValidator.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Recovery\RuntimeBootstrapRepairService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Rollout\RuntimeRolloutExecutionBridge.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\Startup\LocalPosStartupResultTests.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\Naming\CanonicalNamingGuardTests.cs`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V001\CURRENT_TASK_V001.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V001\CURRENT_RESULT_V001.md`
- `E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\*`

The repo already had many unrelated dirty files; they were not reverted.

## 19. Build/Test Commands And Counts

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal
PASS: 0 warnings, 0 errors

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal
PASS: 0 warnings, 0 errors

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~MainWindow|FullyQualifiedName~Offline|FullyQualifiedName~Database|FullyQualifiedName~Naming|FullyQualifiedName~Documentation" -v minimal
PASS: 178 passed, 0 failed, 0 skipped
```

## 20. Prompt056 Label Proof

`E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs` now contains:

```text
CoordinationPromptLabel = "prompt056"
```

Window title path remains InstallationV0 Phase 1/2 and tests assert the prompt056 label.

## 21. No DB/API/PIN/Process/Source-Push Mutation

Confirmed:

- No PostgreSQL mutation.
- No migration/seed.
- No DB role/password/GRANT/REVOKE change.
- No Pairing Code redemption.
- No API token/contract change.
- No employee operational PIN value or validation rule change.
- No WPF launch.
- No process stop/start.
- No OBM source commit or push.

## 22. Operator Physical Retest Steps

```text
A. OBM-POS Runtime Development
   -> connect local DB as hung
   -> MainWindow opens directly

B. InstallationV0 -> Open OBM-POS
   -> same LocalPosStartupService
   -> MainWindow opens

C. API HTTP 401/unavailable
   -> MainWindow remains open
   -> local CRUD works
```

## 23. Coordination Commit SHA

The commit containing this report is created after the report file is written. The final Codex response records the exact coordination commit SHA.
