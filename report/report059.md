# report059 - Prompt059 Execution Report

## 1. Verdict

`OBM_POS_CANONICAL_STATION_ASSIGNMENT_READY_FOR_PHYSICAL_RETEST`

Prompt059 removed the legacy post-MainWindow POS-station assignment gate and made the visible runtime consume the canonical station context already validated by `LocalPosStartupService`.

## 2. DOCS_READ_BEFORE_CODE_GATE Proof

`DOCS_READ_BEFORE_CODE_GATE=PASS`

Read before source edits:

- `E:\Project2026\4POS\NailSalonNet8\AGENTS.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `report/report056.md`
- `report/report057.md`
- `E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\direct-runtime-path.mmd`
- `E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\installation-handoff-path.mmd`
- `E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\final-symbol-inventory.csv`
- `E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\graphify\graphify-out\graph.json`

CanonicalDocVersion: `V001`

CanonicalDocSha256: `7044A02F29FE349FE531DE6800BA739E6B29EA473B9A867881B283DB8743BC72`

V001 still authorizes local-DB-first startup and post-MainWindow API/session deferral.

## 3. Physical Two-Dialog Evidence

Operator evidence after Prompt058:

- MainWindow opened physically.
- Immediately afterward, legacy station-assignment dialogs appeared:
  - `POS Station Required`
  - `POS Station Identity Mismatch`
- The same run had already proven local DB auth, schema, runtime profile activation, Tenant/POS consistency, and `LocalPosReady=True`.

## 4. POS Station Identity Is Not ASP.NET Identity

POS station identity/station assignment is the local POS station context for this physical machine. It is not ASP.NET Identity, not employee operational PIN, not PostgreSQL credential, and not API authentication.

## 5. Exact Modal Call Sites And Trigger Predicates

Pre-change normal-runtime call sites:

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
  - method: `StartNormalApplicationAsync`
  - timing: after `ShowMainWindowForActivatedRuntimeAsync` succeeded and after API session attempt
  - predicate: `ValidateRuntimePosStationIdentityAsync(showWarning: true)` returned false
  - source used: `pos-station.json` through `PosStationLocalSettings.LoadCurrentPosGuid`, `TblPosLocal`, and `CompanyInfo`
  - mutation: no PostgreSQL mutation; could show modal

- `E:\Project2026\4POS\NailSalonNet8\MainWindow.xaml.cs`
  - method: `MainWindow_Loaded`
  - timing: after MainWindow loaded
  - predicate: `ValidateRuntimePosStationIdentityAsync(showWarning: true)` returned false
  - source used: same legacy saved assignment plus `CompanyInfo`
  - mutation: no PostgreSQL mutation; could show modal

- `E:\Project2026\4POS\NailSalonNet8\Services\MainServices.cs`
  - method: `LoadSelectedPosStationIdentityAsync`
  - pre-change trigger: stale or missing saved station file could show `POS Station Required`
  - post-change: stale mirror is repaired from canonical runtime context when unambiguous; explicit repair wording only

- `E:\Project2026\4POS\NailSalonNet8\Services\MainServices.cs`
  - method: `ValidateRuntimePosStationIdentityAsync`
  - pre-change trigger: saved file missing/stale or not matching `CompanyInfo`
  - post-change: accepts canonical runtime `CompanyInfo` match, repairs mirror, and uses repair-only wording if explicitly invoked

Current line refs:

- `App.xaml.cs:1085`, `1095`, `1212`, `1354`
- `MainWindow.xaml.cs:840`, `846`, `851`
- `MainServices.cs:403`, `493`, `524`, `587`, `603`, `633`, `652`

## 6. Pre-Change Station-Source Inventory And Precedence

Pre-change effective precedence:

1. `TblPosRuntimeProfile` / `TblTenant` / `TblPosLocal` validated startup.
2. `LocalPosStartupResult.LocalIdentity` carried prompt058 payload.
3. `CompanyInfo` was hydrated before MainWindow.
4. stale or missing `pos-station.json` could still override the result through a later modal gate.
5. Advanced Settings manual assignment was demanded by normal startup dialogs.

Full inventory: `E:\Project2026\RecoveryReports\InstallationV0\StationAssignmentCleanupV001\station-source-inventory.csv`.

## 7. Canonical Station Authority And Final Precedence

Final precedence:

```text
Activated TblPosRuntimeProfile
-> matching TblTenant
-> matching TblPosLocal
-> LocalPosStartupResult.LocalIdentity
-> CompanyInfo runtime fields
-> pos-station.json mirror/cache
-> Advanced Settings repair UI only
```

The local DB/profile is authoritative for an installed Activated runtime.

## 8. Duplicate Gate Removed Or Repair-Only Classification

Removed from normal runtime:

- `App.StartNormalApplicationAsync` no longer calls `ValidateRuntimePosStationIdentityAsync(showWarning: true)` after MainWindow.
- `MainWindow.MainWindow_Loaded` no longer calls `ValidateRuntimePosStationIdentityAsync(showWarning: true)`.

Repair-only:

- `MainServices.ValidateRuntimePosStationIdentityAsync` remains callable but no longer owns normal MainWindow gating.
- Explicit manual/terminal settings UI can still show repair/configuration warnings.

## 9. Canonical Context Fields Carried Through LocalPosStartupResult

`LocalPosStartupResult.LocalIdentity` carries:

- `TenantGuid`
- `PosGuid`
- `PosSlotNumber`
- `PosName`
- `SourceClientId`
- `RuntimeState`

No secrets, tokens, employee PINs, or database passwords are carried in this result.

## 10. Runtime Consumers Hydrated

`App.ApplyLocalRuntimeIdentity` hydrates:

- `CompanyInfo.TenantGuid`
- `CompanyInfo.PosGuid`
- `CompanyInfo.PosName`
- `CompanyInfo.SourceClientId`
- `CompanyInfo.ClientType`

It is called before the pre-gateway path and again before post-MainWindow worker checks. Evidence:

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs:1085`
- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs:1095`
- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs:1212`
- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs:1354`

## 11. pos-station.json / Machine-Mirror Handling

`pos-station.json` is now a mirror/cache. If canonical local DB/profile context is valid and unambiguous:

- `App.ApplyLocalRuntimeIdentity` writes the mirror with the canonical station fields.
- `MainServices.LoadSelectedPosStationIdentityAsync` repairs stale mirror state from canonical runtime context.
- `MainServices.ValidateRuntimePosStationIdentityAsync` repairs missing/stale mirror when `CompanyInfo` matches a valid local station row.

No PostgreSQL mutation is performed by this correction.

## 12. Advanced Settings Repair-Only Proof

Advanced Settings POS Station remains available for repair/manual reassignment. It is no longer demanded by normal runtime startup after canonical local installation.

Repair-only wording now uses `POS Station Assignment Repair` in `MainServices`. Terminal/settings warnings remain terminal/configuration-specific and are not MainWindow startup gates.

## 13. TenantGuid / PosGuid / DeviceGuid Terminology Correction

Correct terminology:

- `TenantGuid` identifies tenant.
- `PosGuid` identifies the local POS station row.
- `PosGuid` is not `TenantGuid`.
- `PosGuid` is not device identity.
- `DeviceGuid`/device registration belongs to device/enrollment/token flows, not local DB MainWindow startup.
- `SourceClientId` is `POS:{PosGuid:D}`.

## 14. Exact Files Changed / Deleted / Moved

Changed in OBM source/docs/evidence, not committed or pushed:

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\MainWindow.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\MainServices.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\Wiring\CanonicalRuntimeWiringTests.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V004\CURRENT_TASK_V004.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V004\CURRENT_RESULT_V004.md`
- `E:\Project2026\RecoveryReports\InstallationV0\StationAssignmentCleanupV001\*`

Deleted/moved files: none.

Coordination file created:

- `report/report059.md`

## 15. Tests / Build Counts

Required commands:

```powershell
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~MainWindow|FullyQualifiedName~Database|FullyQualifiedName~Station|FullyQualifiedName~Identity|FullyQualifiedName~Naming|FullyQualifiedName~Wiring" -v minimal
```

Results:

- InstallationV0 build: PASS, 0 warnings, 0 errors.
- WPF build: PASS, 176 warnings, 0 errors. Warnings are existing project nullable/analyzer/scaffold warnings; Prompt059 introduced no build error.
- Filtered tests: PASS, 199 passed, 0 failed, 0 skipped.

## 16. Graphify / Evidence Folder And Hashes

Evidence folder:

`E:\Project2026\RecoveryReports\InstallationV0\StationAssignmentCleanupV001`

Hashes:

- `README.md` = `14DC0AF480B834CE60261F569A6F16D03EF53F65373DEEEE5AA1C6FDE3AF04E8`
- `SHA256SUMS.txt` = `06021C5D12CC795ADDA21F80355AFDC41807B586C9CEC18DAA202CC1FA63D26E`
- `station-source-inventory.csv` = `62AA2E9EE63EA493EBAD6104D17E5017B0C768B1D1382ED7F198783942783F45`
- `pre-change-station-flow.mmd` = `D8BE2F6A6801D117A8B2D9E5B54A11A5CC0DDC5C25A9FBC90792E140EE3355FC`
- `post-change-station-flow.mmd` = `A74FEC6F38B50FC49F9B391AE6614BBFAE8048DF78338967EE4875F55C1D6A2B`
- `modal-callsite-scan.txt` = `213DF3A9F6B694F3C1769C59F30100F89A40EE900BA7D344D21BA862823D1AE4`
- `runtime-consumer-hydration.csv` = `BA028806C31527149FBF60ED8A69AAECC6485D94CE5BC0F8F2985EE06B42724D`
- `graphify\graphify-out\graph.json` = `A4A916C64E305EC26A5F701BA00678F42E824C31E4CBA6BE6BC44F51068168EC`

Active WPF `.cs` source scan for:

```text
POS Station Identity Mismatch
POS station identity mismatch
ValidateRuntimePosStationIdentityAsync(showWarning: true)
POS_RUNTIME_ROUTE_STATIONIDENTITYMISSING
CoordinationPromptLabel = "prompt058"
```

Result: `ZERO_ACTIVE_MATCHES`.

## 17. Prompt059 Label Proof

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs:5`
  - `CoordinationPromptLabel = "prompt059"`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs:28`
  - title interpolates `OBM InstallationV0 Phase 1/2 - {InstallationV0BuildInfo.CoordinationPromptLabel}`

## 18. No DB/API/PIN/Process/Source-Push Mutation Proof

Confirmed:

- No PostgreSQL mutation.
- No migration or seed.
- No DB role/password/GRANT/REVOKE change.
- No Pairing Code redemption.
- No API token/contract change.
- No employee operational PIN change.
- No ASP.NET Identity table deletion.
- No WPF launch.
- No process stop/start.
- No OBM source commit or push.
- No secrets printed.

## 19. Exact Operator Physical Retest Steps

1. Launch InstallationV0 prompt059.
2. Confirm Phase 2/local POS state is Ready/Activated.
3. Click `Open OBM-POS`.
4. Verify MainWindow opens.
5. Verify no `POS Station Required` modal appears.
6. Verify no `POS Station Identity Mismatch` modal appears.
7. Launch `OBM-POS Runtime Development` directly.
8. Verify MainWindow opens directly without station-assignment modal.
9. Keep API unavailable or returning HTTP 401 if needed.
10. Verify MainWindow stays open and local CRUD works.
11. Do not redeem Pairing Code, rerun provisioning, mutate DB, or test manual Advanced Settings reassignment in this retest.

## 20. Coordination Commit SHA

This report is committed in the coordination repository after file creation. The exact pushed commit SHA is reported in the final Codex response because a Git commit cannot embed its own final object hash inside the file without changing that hash.

