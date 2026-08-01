# Report 052 - Activated Local Runtime vs API Bootstrap Repair Routing

## 1. Verdict

`BLOCKED_OBM_POS_LOCAL_RUNTIME_ROUTE_PRECEDENCE`

Source was tightened for prompt052, but the physical ProductRoot currently does not contain the local runtime bootstrap files required by the normal startup router. Therefore the observed `BootstrapRepairRequired` branch is not proven to come from `WPF_HELLO_HTTP_401`; it is currently explained by missing local runtime bootstrap material.

## 2. Physical Prompt051 Evidence

Operator-provided prompt051 UI evidence showed:

- Build label: `prompt051`
- Phase 2 Local DB Baseline: `Phase 2 v002 Complete`
- Local POS status: `Ready / Activated / LOCAL_POS_READY_ACTIVATED`
- RuntimeState: `Activated`
- API status: `Reauthorization Required / WPF_HELLO_HTTP_401`
- Open OBM-POS result: `POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED`

Read-only ProductRoot file checks during prompt052 found:

- `E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot\Config\database-settings.json`: absent
- `E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot\Secrets\database-password.dpapi`: absent

No secret file content was read.

## 3. Exact Active Branch / Predicate Root Cause

Pre-change active branch:

`App.OpenInstalledPosFromInstallationV0Async`
-> `RetryStartupAssessmentAsync`
-> `ApplicationStartupCoordinator.CreateDefault().AssessAsync()`
-> `RuntimeBootstrapLocator.Locate()`
-> `bootstrap is null && Directory.Exists(SpacePosConfigurationPaths.ProductRoot)`
-> `DatabaseStartupState.BootstrapRepairRequired`
-> `RouteFromAssessment`
-> `POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED`

Exact predicate:

```csharp
if (bootstrap is null && Directory.Exists(SpacePosConfigurationPaths.ProductRoot))
```

Exact technical summary:

```text
Product root exists, but the minimal database bootstrap locator could not be loaded.
```

## 4. Local-runtime Bootstrap vs API-bootstrap Distinction

Local runtime bootstrap means ProductRoot database metadata and protected PostgreSQL credential material required to open the local POS database.

API bootstrap means Phase 1 WpfJWT/protected hello/bootstrap identity material used for Platform API authorization.

Prompt052 source assertions now prove `WPF_HELLO_HTTP_401` is handled in `InstallationV0ApiCloudStatus` and is not present in `App.xaml.cs`, `ApplicationStartupCoordinator.cs`, or `RuntimeProfileStartupAssessmentService.cs`.

## 5. Corrected Route Precedence

The normal startup route remains local DB first:

1. ProductRoot/runtime database bootstrap.
2. Protected PostgreSQL credential readback.
3. PostgreSQL authentication.
4. Schema/runtime profile/Tenant/POS identity.
5. `TblPosRuntimeProfile.RuntimeState`.
6. Local route decision.
7. MainWindow transition.
8. API/cloud state reported independently.

## 6. Local Route + API State Composition Model

`PosStartupRouteResult` now carries:

- `RouteDecision`
- `ResultCode`
- `RuntimeState`
- `LocalRuntimeReady`
- `ApiStatus`
- `ApiResultCode`
- MainWindow construction/show/visibility fields

`ApiStatus=ReauthorizationRequired` can coexist with `RouteDecision=OpenMainPos` in the result model.

## 7. MainWindow Transition Preservation

The prompt051 MainWindow transition path was preserved:

`ShowMainWindowForActivatedRuntimeAsync`
-> UI dispatcher
-> previous window/shutdown preservation
-> `ShutdownMode.OnExplicitShutdown`
-> resolve `MainWindow`
-> set `Application.Current.MainWindow`
-> `Show`
-> `Activate`
-> `IsVisible`
-> `ShutdownMode.OnMainWindowClose`
-> structured route result.

## 8. Direct Runtime Behavior

No automatic WPF launch was performed. The source path still requires local runtime bootstrap before entering direct normal runtime. API 401 is not part of the direct local runtime predicate.

## 9. Installation / Recovery Safeguards Retained

Retained safeguards:

- missing runtime bootstrap -> `BootstrapRepairRequired`
- unreadable PostgreSQL credential -> `CredentialRecoveryRequired`
- missing database -> `DatabaseRecoveryRequired`
- missing runtime profile -> `ExistingDatabaseValidationRequired`
- identity mismatch -> `RuntimeIdentityMismatch`
- `Installing` -> `InstallationIncomplete`
- `RecoveryRequired` -> recovery route
- `Disabled` -> recovery/blocked route

## 10. No-mutation Proof

No WPF launch was performed. No Pairing Code was redeemed. No token was refreshed or rotated. No PostgreSQL write/migration/seed/marker/runtime-history/outbox/PIN/role/password mutation was performed. No User or Machine environment variable was set.

## 11. Exact Source Files Changed

Local OBM source changes only:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\PosStartupRouteResult.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0Module.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

No OBM source commit or push was performed.

## 12. Build / Test Commands and Counts

Commands run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~OpenObmPos|FullyQualifiedName~RuntimeState|FullyQualifiedName~MainWindow|FullyQualifiedName~Offline|FullyQualifiedName~BootstrapRepair" -v minimal
```

Results:

- InstallationV0 build: 0 warnings, 0 errors
- WPF build: 176 warnings, 0 errors
- Focused tests: 145 passed, 0 failed, 0 skipped

The first WPF build attempt hit a transient file lock because it was run in parallel with the InstallationV0 build; the serial rerun passed.

## 13. Prompt052 Label Proof

`InstallationV0BuildInfo.CoordinationPromptLabel` is now:

```text
prompt052
```

Window title remains:

```text
OBM InstallationV0 Phase 1/2 - {InstallationV0BuildInfo.CoordinationPromptLabel}
```

## 14. Exact Operator Retest Steps

1. Rebuild from Visual Studio or `dotnet build`.
2. Launch InstallationV0 V0 ProductRoot lane.
3. Confirm build label `prompt052`.
4. Confirm route diagnostics include separate `ApiStatus` and `ApiResultCode`.
5. If ProductRoot runtime bootstrap files are still absent, expect `BootstrapRepairRequired`.
6. After approved ProductRoot runtime bootstrap materialization/repair, retry Open OBM-POS with API still 401; expected route is `OpenMainPos`.

## 15. Deferred Refresh-token / PIN / Identity Cleanup

Deferred and not performed:

- WpfJWT refresh/rotation
- Pairing Code reauthorization
- employee PIN work
- Platform identity cleanup
- ProductRoot bootstrap repair/materialization

## 16. No Secrets / No DB Mutation / No Source Push Proof

No password, token, cookie, ClientSecret, connection string with credential, Pairing Code, or protected credential was printed or persisted in this report.

The only pushed artifact for this task is this coordination report. OBM source edits remain local and uncommitted.

## 17. Coordination Commit SHA

Commit containing this report: recorded in the final Codex response after `origin/main` push.

