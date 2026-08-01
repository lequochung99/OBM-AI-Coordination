# Report 057 - Canonical Runtime Wiring

## 1. Verdict

`OBM_POS_CANONICAL_RUNTIME_WIRING_READY_FOR_PHYSICAL_LAUNCH`

## 2. DOCS_READ_BEFORE_CODE_GATE Evidence And Hashes

`DOCS_READ_BEFORE_CODE_GATE=PASS`

Read before source edits:

- `E:\Project2026\4POS\NailSalonNet8\AGENTS.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `report/report055.md`
- `report/report056.md`
- `E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\README.md`
- `E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\direct-runtime-path.mmd`
- `E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\installation-handoff-path.mmd`
- `E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\final-symbol-inventory.csv`
- `E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\forbidden-name-scan.txt`
- `E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\SHA256SUMS.txt`

Pre-edit hashes:

```text
CanonicalDocVersion=V001
CanonicalDocSha256=7044A02F29FE349FE531DE6800BA739E6B29EA473B9A867881B283DB8743BC72
AgentsSha256=90D09B8058381663A4EF317A5707256FA01B1DCDE5FE9F5BD3D4FA8F3CC10E8B
CurrentTaskSha256=093CA0C7D9040D52B94D065944A35DDE53A19C135090A4B42F993B9C5CBBC184
CurrentResultSha256=16D6B1DE20FED54988C51DEDEEED2F73086CDD62BD3487B8F1E30C355572A973
```

## 3. Pre-Change Wiring Table

| Responsibility | Active type/method | DI lifetime/registration | Direct runtime caller | Installation handoff caller | Problem |
| --- | --- | --- | --- | --- | --- |
| Local runtime readiness | `LocalPosStartupService.AssessAsync` | Not registered in production DI | `new LocalPosStartupService().AssessAsync` | `new LocalPosStartupService().AssessAsync` during retry | Two independent service instances; handoff did not use host-resolved service |
| API session | `IApiSessionInitializer -> ApiSessionInitializer` | Duplicate singleton interface mapping | After `MainWindow` visibility proof | Same normal route after handoff | Duplicate registration noise |
| MainWindow transition | `ShowMainWindowForActivatedRuntimeAsync` | `MainWindow` singleton | Direct startup | `OpenInstalledPosFromInstallationV0Async` then normal route | Correct owner, needed shared readiness input |
| Installation diagnostics | `InstallationV0Module` | Static module boundary | Installation profile only | Open OBM-POS callback | Correct boundary |
| Local configuration | Runtime launch profile and ProgramData split config | Configuration builder | App constructor | Retry after handoff | Profile already selected V0 DB/user/passfile |

## 4. Final Production DI Registration Table And Lifetimes

| Responsibility | Registration | Lifetime | Reason |
| --- | --- | --- | --- |
| Local runtime readiness | `services.AddSingleton(_localPosStartupService)` | Singleton instance owned by `App` | Direct startup and handoff share one readiness owner |
| Local runtime readiness interface | `services.AddSingleton<ILocalPosStartupService>(sp => sp.GetRequiredService<LocalPosStartupService>())` | Singleton mapping | Handoff resolves the same service through DI |
| API session initializer | `services.AddSingleton<ApiSessionInitializer>()` | Singleton | Owns API session startup state |
| API session interface | `services.AddSingleton<IApiSessionInitializer>(sp => sp.GetRequiredService<ApiSessionInitializer>())` | Singleton mapping | One final interface implementation |
| MainWindow | `services.AddSingleton<MainWindow>()` | Singleton | Existing WPF window ownership retained |

## 5. Final Local Configuration Chain And Sanitized DB/User Proof

```text
OBM-POS Runtime Development launch profile
-> SPACEPOS_ConnectionStrings__PostgreSqlConnection
-> Database=obm_pos_dev_v0_pg
-> Username=hung
-> Passfile-based credential source
-> LocalPosStartupService
```

Sanitized proof: database is `obm_pos_dev_v0_pg`, runtime username is `hung`, password source is a protected local credential/passfile. No password or passfile contents were read or printed.

## 6. Final LocalPosStartupService Decision Ownership

`LocalPosStartupService` remains the sole local readiness decision owner for normal runtime. It checks local configuration/database readiness and returns `LocalPosStartupResult.Ready` when local POS can open.

## 7. Final Direct Runtime Path

```text
App startup
-> resolve canonical local configuration
-> _localPosStartupService.AssessAsync
-> LocalPosStartupResult.Ready
-> StartNormalApplicationAsync
-> ShowMainWindowForActivatedRuntimeAsync
-> MainWindow visible
-> ApiSessionInitializer.StartAsync afterward
```

## 8. Final Shared MainWindow Transition

`ShowMainWindowForActivatedRuntimeAsync(IServiceProvider services)` remains the single WPF window transition. It constructs `MainWindow`, calls `Show`, verifies visibility, records `OPEN_POS_MAINWINDOW_SHOWN`, and restores prior WPF state on failure.

## 9. Final InstallationV0 Open OBM-POS Handoff Path

```text
InstallationV0 Open OBM-POS
-> TryApplyVerifiedInstallationV0ProductRoot
-> RetryStartupAssessmentAsync
-> AppHost?.Services.GetService<ILocalPosStartupService>() ?? _localPosStartupService
-> localPosStartupService.AssessAsync
-> StartNormalApplicationAsync(forceNormalApplication: true)
-> ShowMainWindowForActivatedRuntimeAsync
```

## 10. API Initializer Post-MainWindow And Non-Blocking Proof

`App.xaml.cs` order proof:

```text
ShowMainWindowForActivatedRuntimeAsync
-> visibleRouteResult success check
-> ProcessHandoffHelper.RecordStartupValidation
-> GetRequiredService<IApiSessionInitializer>()
-> await apiSession.StartAsync()
```

If API initialization throws, the diagnostic is `API session deferred; local POS remains open.` Local runtime and `MainWindow` are not blocked by HTTP 401 or API unavailable states.

## 11. Recovery/Updater Service Boundary Proof

No new recovery, updater, router, bootstrap, compatibility, security-flag, application-password, or API-prerequisite layer was introduced. Recovery/updater behavior remains outside the local runtime readiness owner and was not rewritten in this task.

## 12. Exact Files Changed

OBM source/doc files changed locally, not committed or pushed:

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\Wiring\CanonicalRuntimeWiringTests.cs`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V002\CURRENT_TASK_V002.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V002\CURRENT_RESULT_V002.md`

Coordination file created and pushed:

- `report/report057.md`

## 13. Tests Added/Updated

- Added `CanonicalRuntimeWiringTests` under `E:\Project2026\4POS\NailSalonNet8.Tests\Wiring`.
- Updated existing InstallationV0 source guard assertions for prompt057 label and shared service path.

## 14. Build/Test Commands And Counts

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal
Result: PASS, 0 warnings, 0 errors

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal
Result: PASS, 0 warnings, 0 errors

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~MainWindow|FullyQualifiedName~Offline|FullyQualifiedName~Database|FullyQualifiedName~Naming|FullyQualifiedName~Documentation|FullyQualifiedName~DependencyInjection|FullyQualifiedName~Wiring" -v minimal
Result: PASS, 185 passed, 0 failed, 0 skipped
```

## 15. Static Wiring Verification Results

```text
LocalPosStartupService production registration count: 1
ILocalPosStartupService production mapping count: 1
ApiSessionInitializer concrete registration count: 1
IApiSessionInitializer mapping count: 1
MainWindow registration count: 1
Forbidden compatibility-name scan over active WPF source/docs: PASS, no matches
Runtime profile DB/user/passfile scan: PASS
Pre-MainWindow protected API gate scan: PASS
```

## 16. Controlled Live Startup Smoke Result

```text
LIVE_STARTUP_SMOKE=DEFERRED_TO_OPERATOR
```

Safe deferral reason: the prompt057 code/build/test wiring is ready, but the live WPF launch should be performed by the operator against the known local database/passfile state. No WPF process was launched and no process was stopped.

## 17. Prompt057 Label Proof

`E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`:

```text
CoordinationPromptLabel = "prompt057"
```

`E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`:

```text
Title = $"OBM InstallationV0 Phase 1/2 - {InstallationV0BuildInfo.CoordinationPromptLabel}"
```

## 18. Canonical V001 Unchanged And Current Hash

```text
CanonicalDocVersion=V001
CanonicalDocSha256=7044A02F29FE349FE531DE6800BA739E6B29EA473B9A867881B283DB8743BC72
```

## 19. History Preservation And Updated CURRENT Hashes

Prompt056 snapshot preserved:

```text
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V002\CURRENT_TASK_V002.md
Hash=093CA0C7D9040D52B94D065944A35DDE53A19C135090A4B42F993B9C5CBBC184

E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V002\CURRENT_RESULT_V002.md
Hash=16D6B1DE20FED54988C51DEDEEED2F73086CDD62BD3487B8F1E30C355572A973
```

Updated current docs:

```text
CURRENT_TASK.md Hash=D2AE96BE77FCE9B1028810355CCE9FE2E043E37D83611381FC8FE970D086118A
CURRENT_RESULT.md Hash=D3342AB33DAA8EF8F8B8345988BDB43DAAFA2887806EFD2366FCB116525EFF0D
```

## 20. No Forbidden Mutation/Secret/Source-Push Proof

- No PostgreSQL mutation.
- No migration or seed.
- No DB role/password/GRANT/REVOKE change.
- No API token/contract change.
- No employee operational PIN change.
- No business records created.
- No ASP.NET Identity tables deleted.
- No User/Machine environment variables set.
- No password, passfile content, token, or secret printed.
- No OBM source commit or push.

## 21. Exact Operator Physical Retest Steps

```text
1. Launch OBM-POS Runtime Development from Visual Studio.
2. Verify it uses local DB obm_pos_dev_v0_pg as hung through passfile.
3. Verify MainWindow opens directly.
4. Verify the prompt057 title/build label where applicable.
5. Launch InstallationV0 profile and use Open OBM-POS.
6. Verify it reaches the same LocalPosStartupService readiness path.
7. Verify MainWindow opens.
8. Test API HTTP 401/unavailable behavior.
9. Verify MainWindow remains open and local CRUD works.
10. Do not authorize ASP.NET Identity deletion until this physical test passes.
```

## 22. Coordination Commit SHA

This report is committed in the coordination repository. The exact pushed commit SHA is recorded in the final Codex response for this task.
