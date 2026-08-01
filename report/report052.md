# Report 052 - Minimal Local DB Startup Service Cleanup

## 1. Verdict

`OBM_POS_MINIMAL_STARTUP_SERVICES_READY_FOR_USER_RETEST`

The normal WPF startup path now uses the configured local PostgreSQL route directly. API/session status remains post-MainWindow behavior and no longer drives the local startup decision.

## 2. Exact Misleading Names That Caused API/Local-DB Confusion

- `ApplicationStartupCoordinator`: mixed ProductRoot bootstrap-file discovery with normal runtime startup.
- `BootstrapRepairRequired`: generic bootstrap wording made API/bootstrap-token and local database configuration look like one gate.
- `POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED`: surfaced a local/ProductRoot bootstrap diagnosis as a runtime route result.
- `AppJwtBootstrapper`: named like a startup prerequisite even though it runs after MainWindow is visible.
- `DatabaseStartupAssessmentService`: retained as compatibility, but the active normal path now uses `LocalPosStartupService`.
- `InstallationV0CompletedReadinessService`: remains installation/audit-only and is not called by normal runtime startup.

## 3. Active Caller Inventory Before Cleanup

- `App` constructor could call configured DB assessment, but the fallback path still called `ApplicationStartupCoordinator`.
- `OpenInstalledPosFromInstallationV0Async` used `RetryStartupAssessmentAsync`, which could call `ApplicationStartupCoordinator`.
- `RouteFromAssessment` could emit `POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED`.
- `StartNormalApplicationAsync` already showed MainWindow before API auth/session initialization.
- DI registered concrete `AppJwtBootstrapper` under `IAppJwtBootstrapper`.

## 4. Deleted Services/Files/Result Codes/DI Registrations

- Deleted active `ApplicationStartupCoordinator.CreateDefault().AssessAsync()` calls from `App.xaml.cs`.
- Deleted active normal-runtime generation of `POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED`; legacy state maps to `POS_RUNTIME_RECOVERY_REQUIRED` if it ever reaches the generic formatter.
- Deleted concrete `AppJwtBootstrapper` DI registration from normal startup registration and replaced it with `ApiSessionInitializer`.
- No source files were physically deleted in this bounded pass because compatibility callers/tests still reference old names.

## 5. Renamed or Merged Services/Types

- Added `LocalPosStartupService` as the active normal local PostgreSQL startup service.
- Kept `DatabaseStartupAssessmentService` as a compatibility subclass of `LocalPosStartupService`.
- Added `IApiSessionInitializer` and renamed the active implementation class to `ApiSessionInitializer`.
- Kept `IAppJwtBootstrapper` as a compatibility interface over the same API-session initializer.

## 6. Final Minimal Normal-Startup Code Path

```text
App builds runtime configuration
-> resolves DatabaseProvider + configured connection string
-> DevelopmentProfileLaunchPolicy approves ProductRoot/database lane
-> LocalPosStartupService.AssessAsync(...)
-> PostgreSQL authentication/schema/baseline/runtime profile/local identity checks
-> CanEnterNormalApplication
-> ShowMainWindowForActivatedRuntimeAsync
```

If no configured local PostgreSQL connection exists, the app returns `NewInstallationRequired`; it does not try ProductRoot bootstrap repair before MainWindow.

## 7. Final Installation-Only Verification Path

`InstallationV0CompletedReadinessService` remains outside ordinary normal startup. It is retained for installation diagnostics/audit proof and is not part of direct runtime entry or Open OBM-POS local readiness.

## 8. Final Post-MainWindow API Path

`StartNormalApplicationAsync` shows/activates MainWindow first. After that, it resolves `IApiSessionInitializer` and calls `StartAsync`. Exceptions are logged as `API_SESSION_DIAG` and local POS remains open/offline-deferred.

## 9. Runtime DB Username Proof (`hung`, Sanitized)

`Properties/launchSettings.json` now configures both official WPF profiles with:

```text
Database=obm_pos_dev_v0_pg
Username=hung
Password source=Passfile path only
```

The file does not contain `Password=`.

## 10. Open OBM-POS and Direct Startup Shared-Path Proof

Direct startup and `InstallationV0 -> Open OBM-POS` both call `LocalPosStartupService.AssessAsync(...)` through the same configured connection-string path. `OpenInstalledPosFromInstallationV0Async` calls `RetryStartupAssessmentAsync`, and that retry no longer calls `ApplicationStartupCoordinator`.

## 11. Deletion/Rename Table

| Old name | Active purpose before | Final action | Final name or replacement |
| --- | --- | --- | --- |
| `ApplicationStartupCoordinator` | ProductRoot bootstrap locator fallback in normal startup | Active App caller removed | `LocalPosStartupService` |
| `DatabaseStartupAssessmentService` | Normal local DB assessment | Compatibility shim retained | `LocalPosStartupService` |
| `AppJwtBootstrapper` | Post-MainWindow API credential/session initialization | Active implementation renamed | `ApiSessionInitializer` |
| `IAppJwtBootstrapper` | API credential/session contract | Compatibility interface retained | `IApiSessionInitializer` |
| `POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED` | Generic route code emitted from enum name | Active emission removed | `POS_RUNTIME_RECOVERY_REQUIRED` |
| `InstallationV0CompletedReadinessService` | Installation completion/audit checks | Retained outside normal runtime | Installation/audit-only verifier |

## 12. Exact Source Files Added/Changed/Deleted

Changed locally in OBM source:

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\ConnectService\AppJwtBootstrapper.cs`
- `E:\Project2026\4POS\NailSalonNet8\ConnectService\IAppJwtBootstrapper.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\DatabaseStartupAssessmentService.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\PosStartupRouteResult.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0Module.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8\Properties\launchSettings.json`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

Deleted files: none.

## 13. Build/Test Commands and Counts

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal
```

Result: passed, 0 warnings, 0 errors.

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal
```

Result: passed, 176 warnings, 0 errors.

```text
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~OpenObmPos|FullyQualifiedName~MainWindow|FullyQualifiedName~Offline|FullyQualifiedName~Database" -v minimal
```

Result: passed, 176 passed, 0 failed, 0 skipped.

## 14. Prompt052 Label Proof

`InstallationV0BuildInfo.CoordinationPromptLabel` is:

```text
prompt052
```

The window title format remains:

```text
OBM InstallationV0 Phase 1/2 - prompt052
```

## 15. Operator Physical Retest Steps

1. Open Visual Studio.
2. Select `OBM-POS Runtime Development` or `OBM-POS InstallationV0 Phase1`.
3. Confirm the launch profile targets ProductRoot `E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot`.
4. Start WPF.
5. With local PostgreSQL ready as `obm_pos_dev_v0_pg` / role `hung`, verify MainWindow opens.
6. Keep API unavailable or returning 401 and verify MainWindow still opens while API/session is offline or reauthorization-required.
7. From InstallationV0, click `Open OBM-POS` once and verify it uses the same local DB startup path.

## 16. No Secrets/No DB Mutation/No Source Push Proof

No password, token, cookie, Pairing Code, ClientSecret, protected credential, or full connection string with password was printed.

No PostgreSQL database was created, dropped, migrated, seeded, or modified. No Pairing Code was redeemed. No WPF process was launched automatically. No OBM source commit or push was performed.

Only this coordination report is committed and pushed.

## 17. Coordination Commit SHA

Commit containing this report: recorded in final Codex response after push to `origin/main`.
