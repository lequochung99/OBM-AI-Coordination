# report058 - Prompt058 Execution Report

## 1. Verdict

`OBM_POS_LOCAL_STATION_IDENTITY_WIRING_READY_FOR_PHYSICAL_RETEST`

The Prompt057 physical failure `POS_RUNTIME_ROUTE_STATIONIDENTITYMISSING` was traced to a local runtime startup gate that still depended on ProgramData station/device identity after the local PostgreSQL runtime profile was already activated. Prompt058 rewires normal runtime identity to the canonical local database state and carries that identity into the MainWindow route.

## 2. DOCS_READ_BEFORE_CODE_GATE

Read before edits:

- `E:\Project2026\4POS\NailSalonNet8\AGENTS.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `C:\Users\lequo\Documents\Codex\2026-07-29\files-mentioned-by-the-user-codex\work\OBM-AI-Coordination\report\report056.md`
- `C:\Users\lequo\Documents\Codex\2026-07-29\files-mentioned-by-the-user-codex\work\OBM-AI-Coordination\report\report057.md`

Pre-edit anchor hashes:

- `INSTALLATION_RUNTIME_CANONICAL.md` = `7044A02F29FE349FE531DE6800BA739E6B29EA473B9A867881B283DB8743BC72`
- `CURRENT_TASK.md` before Prompt058 = `D2AE96BE77FCE9B1028810355CCE9FE2E043E37D83611381FC8FE970D086118A`
- `CURRENT_RESULT.md` before Prompt058 = `D3342AB33DAA8EF8F8B8345988BDB43DAAFA2887806EFD2366FCB116525EFF0D`

Gate result: PASS.

## 3. Physical Prompt057 Evidence

Operator evidence from Prompt057 showed:

- Phase 1 and Phase 2 completed.
- Local database was created/provisioned.
- Runtime state was activated.
- Clicking `Open OBM-POS` did not open MainWindow.
- Route result surfaced `POS_RUNTIME_ROUTE_STATIONIDENTITYMISSING`.
- API calls could still return HTTP 401, but the failure was in the local WPF runtime route.

## 4. Exact STATIONIDENTITYMISSING Predicate And Root Cause

The blocking predicate was in local runtime startup after database readiness checks:

- startup had already confirmed schema/baseline/runtime profile readiness;
- then it required ProgramData station identity via `PosStationLocalSettings.LoadStationIdentity()`;
- missing `DeviceRegistrationId` / `PosDeviceGuid` caused `LocalPosStartupDecision.StationIdentityMissing`;
- `RuntimeRouteResultCode` formatted that enum as `POS_RUNTIME_ROUTE_STATIONIDENTITYMISSING`.

Root cause: both duplicate gate and dropped payload.

## 5. Classification

Classification: BOTH

- Duplicate gate: a second identity owner existed after canonical local activation.
- Dropped payload: `LocalPosStartupResult` did not carry canonical Tenant/POS runtime identity forward to `App.xaml.cs` / `MainWindow`.

## 6. Canonical Local Identity Source

The canonical normal runtime identity source is now:

- activated singleton `dbo."TblPosRuntimeProfile"`;
- matching `dbo."TblTenant"`;
- matching `dbo."TblPosLocal"`;
- `SourceClientId == POS:{PosGuid:D}`;
- runtime state `Activated`.

ProgramData station/device registration identity is not required to open MainWindow after local activation.

## 7. Source/Result/Context Wiring Correction

Changed wiring:

- `LocalPosStartupResult` now carries `LocalPosRuntimeIdentity`.
- `LocalPosStartupService` loads Tenant/POS identity from local runtime profile, `TblTenant`, and `TblPosLocal`.
- missing or mismatched local identity returns `RuntimeIdentityMismatch`, not station identity missing.
- `App.xaml.cs` applies local runtime identity into `CompanyInfo` before normal MainWindow startup and reapplies it after station-load fallback if needed.
- `RuntimeRouteResultCode` no longer emits the physical string `POS_RUNTIME_ROUTE_STATIONIDENTITYMISSING`.

## 8. Direct Runtime Path

Direct runtime path:

`StartupAssessment=InstalledHealthy` -> local runtime identity loaded from PostgreSQL -> `CompanyInfo` populated from runtime profile payload -> MainWindow route continues without ProgramData station/device gate.

## 9. InstallationV0 Handoff Path

InstallationV0 `Open OBM-POS` handoff path:

verified ProductRoot -> retry local startup assessment -> local runtime identity loaded from PostgreSQL -> `CompanyInfo` populated from runtime profile payload -> MainWindow route continues.

## 10. API 401 Remains Non-Blocking

API 401 is not used as a normal runtime MainWindow gate. Local activated database state controls MainWindow eligibility. API/bootstrap token paths may remain unavailable without blocking the already activated local runtime route.

## 11. No Second Readiness/Identity Owner Remains

Normal MainWindow readiness no longer depends on ProgramData station/device identity. The single readiness owner for this route is local runtime state:

- `TblPosRuntimeProfile`
- `TblTenant`
- `TblPosLocal`
- v001/local startup assessment

## 12. Deleted Result Codes/Branches/Files

No files were deleted for Prompt058.

The generated result-code path for `StationIdentityMissing` was neutralized:

- old physical string: `POS_RUNTIME_ROUTE_STATIONIDENTITYMISSING`
- current route fallback for that enum: `POS_RUNTIME_RECOVERY_REQUIRED`

`LocalPosStartupService` no longer emits `StationIdentityMissing` for activated local runtime startup.

## 13. Read-Only DB Evidence

No physical PostgreSQL query was used in this task. Prompt058 source correction was based on Prompt057 physical evidence and static source/test verification only. No database was connected to, created, migrated, seeded, or mutated.

## 14. Exact Files Changed

OBM source files changed, not committed or pushed:

- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\LocalPosStartupResult.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\LocalPosStartupService.cs`
- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\Startup\LocalPosStartupResultTests.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\Wiring\CanonicalRuntimeWiringTests.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V003\CURRENT_TASK_V003.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V003\CURRENT_RESULT_V003.md`

Coordination repo file created:

- `C:\Users\lequo\Documents\Codex\2026-07-29\files-mentioned-by-the-user-codex\work\OBM-AI-Coordination\report\report058.md`

## 15. Build/Test Commands And Counts

Commands executed:

```powershell
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~MainWindow|FullyQualifiedName~Offline|FullyQualifiedName~Database|FullyQualifiedName~Wiring|FullyQualifiedName~Identity" -v minimal
```

Results:

- InstallationV0 build: PASS, 0 warnings, 0 errors.
- WPF build: PASS, 0 warnings, 0 errors.
- Filtered tests: PASS, 198 passed, 0 failed, 0 skipped.
- Test compile warnings were pre-existing nullable/analyzer warnings in test/support files.

## 16. Active-Source Station-Identity Result Scan

Command:

```powershell
rg -n "POS_RUNTIME_ROUTE_STATIONIDENTITYMISSING" E:\Project2026\4POS\NailSalonNet8 --glob "*.cs" --glob "!bin/**" --glob "!obj/**" --glob "!**/*.codex-bak-*"
```

Result: `ZERO_ACTIVE_MATCHES`.

Note: documentation may retain historical evidence text. The active WPF `.cs` source no longer contains the physical result-code literal.

## 17. Prompt058 Label Proof

Build label source:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `CoordinationPromptLabel = "prompt058"`
- `Title = "OBM InstallationV0 Phase 1/2 - prompt058"`

Prompt label scan confirmed prompt058 assertions in focused tests and source.

## 18. Updated CURRENT_TASK/CURRENT_RESULT History And Hashes

Post-update hashes:

- `INSTALLATION_RUNTIME_CANONICAL.md` = `7044A02F29FE349FE531DE6800BA739E6B29EA473B9A867881B283DB8743BC72`
- `CURRENT_TASK.md` = `35673783D17A1E45C3D58F2B97154756E32CF9233CE82A3B98B0F16F7D91C4E1`
- `CURRENT_RESULT.md` = `D00BFB934267CA2CABA6336E88EE0F752FCB20BBBA555B5838179D9B643DFC2A`
- `history\V003\CURRENT_TASK_V003.md` = `D2AE96BE77FCE9B1028810355CCE9FE2E043E37D83611381FC8FE970D086118A`
- `history\V003\CURRENT_RESULT_V003.md` = `D3342AB33DAA8EF8F8B8345988BDB43DAAFA2887806EFD2366FCB116525EFF0D`

History snapshot preserved Prompt057 docs before Prompt058 updates.

## 19. No DB/API/PIN/Process/Source-Push Mutation Proof

Preserved constraints:

- WPF was not launched.
- No Pairing Code was redeemed.
- PostgreSQL was not connected to or mutated.
- No migrations, schema, baseline, seed, or outbox operations were run.
- No database roles, passwords, GRANTs, or ownership were changed.
- No API token contract was changed.
- No employee PIN or login behavior was changed.
- No ApiServer, PlatformAppV0, or WPF process was stopped or started by this task.
- OBM source was not committed or pushed.
- Only the coordination report is committed/pushed.

## 20. Operator Physical Retest Steps

Required next physical retest:

1. Launch the normal installed Prompt058 WPF build.
2. Use the same activated local installation ProductRoot and database.
3. Verify `InstallationV0` can click `Open OBM-POS` and MainWindow opens.
4. Verify direct runtime startup opens MainWindow.
5. Verify no Setup wizard opens.
6. Verify no recovery UI opens for station identity.
7. Verify API HTTP 401 or unavailable state does not block MainWindow when local runtime is activated.
8. Verify local CRUD works against the activated local database.
9. Do not redeem Pairing Code or rerun provisioning during this retest.

## 21. Coordination Commit SHA

This report is committed in the coordination repository after file creation. The exact pushed commit SHA is reported in the final Codex response because a Git commit cannot embed its own final object hash inside the file without changing that hash.

