# Report 060 — Final Station Enabled Gate Closure

## 1. Verdict

`OBM_POS_FINAL_STATION_ENABLED_GATE_CLOSED_READY_FOR_PHYSICAL_RETEST`

## 2. DOCS_READ_BEFORE_CODE_GATE Proof

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V001
CanonicalDocSha256=7044A02F29FE349FE531DE6800BA739E6B29EA473B9A867881B283DB8743BC72
```

Read before source edits:

- `E:\Project2026\4POS\NailSalonNet8\AGENTS.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `report/report057.md`
- `report/report059.md`
- StationAssignmentCleanupV001 evidence files

## 3. Physical Remaining-Modal Evidence

Operator evidence from prompt060:

```text
POS Station Assignment Repair
The local station mirror references a POS station that is not enabled for this local tenant: <sanitized PosGuid>.
```

No raw GUID was copied into this report.

## 4. Exact Remaining Modal Callsite/Caller Chain

Modal owner:

```text
E:\Project2026\4POS\NailSalonNet8\Services\MainServices.cs
LoadSelectedPosStationIdentityAsync
```

Remaining automatic caller before correction:

```text
MainServices.Integration_Signature_Printers_CashDrawerAsync
-> await LoadSelectedPosStationIdentityAsync()
-> default showWarning=true
-> MessageBox "POS Station Assignment Repair" could be displayed by an automatic runtime path
```

Timing: post-MainWindow business/runtime initialization, not operator-initiated Settings repair UI.

## 5. Read-Only DB Station-State Evidence

Database audit was read-only:

```text
BEGIN TRANSACTION READ ONLY;
...
ROLLBACK;
```

Sanitized result:

```text
TblPosRuntimeProfile row count = 1
RuntimeState = Activated
TenantGuid present = true
PosGuid present = true
Slot present = true
TblTenant matching row count = 1
TblTenant active = true
TblPosLocal matching row count = 1
TblPosLocal IsActive = true
TblPosLocal PosEnabled = true
TblPosLocal BookingEnabled = true
Same-tenant station rows = 1
```

This rules out `INSTALLATION_TBLPOSLOCAL_STATE_NOT_MATERIALIZED` and `GENUINE_LOCAL_STATION_AMBIGUITY`.

## 6. Mirror Path/Source Evidence

Canonical ProductRoot mirror:

```text
E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot\State\pos-station.json
exists = true
Tenant matches runtime = true
POS matches runtime = true
```

Legacy stale mirrors found:

```text
C:\Users\lequo\AppData\Local\NailSalonNet8\pos-station.json
C:\ProgramData\SpacePOS\State\pos-station.json
```

Both legacy mirrors did not match runtime identity by sanitized equality checks. The official launch profile points at the canonical ProductRoot.

## 7. Root-Cause Classification

`OTHER_PROVEN_CAUSE`

The canonical DB row and canonical ProductRoot mirror were valid. The remaining defect was a residual automatic callsite using the default warning-enabled station load. That automatic path could still display the repair modal even though normal startup had already proven local runtime identity.

## 8. Exact Pre-Change Predicate

Station query predicate:

```csharp
.Where(x => x.TenantGuid == localTenantGuid && x.PosEnabled)
```

Pre-change automatic caller:

```csharp
await LoadSelectedPosStationIdentityAsync();
```

Because the method default is `showWarning = true`, this was still a modal-capable automatic runtime caller.

## 9. Minimal Correction

Changed automatic runtime caller to:

```csharp
await LoadSelectedPosStationIdentityAsync(showWarning: false);
```

No DB repair plan was required because the canonical `TblPosLocal` row was already active/enabled.

## 10. Final Startup Outcomes And No-Post-MainWindow-Modal Proof

Expected outcomes after source correction:

```text
canonical station valid -> MainWindow -> no station modal
canonical station invalid/ambiguous -> recovery/installation decision before MainWindow, not a post-MainWindow modal
```

Source scan after correction found no automatic source call:

```text
await LoadSelectedPosStationIdentityAsync();
LoadSelectedPosStationIdentityAsync(showWarning: true)
ValidateRuntimePosStationIdentityAsync(showWarning: true)
```

Only test assertions contain those strings.

## 11. Exact Files Changed

OBM source/test/docs/evidence changed:

- `E:\Project2026\4POS\NailSalonNet8\Services\MainServices.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\Wiring\CanonicalRuntimeWiringTests.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V005\CURRENT_RESULT.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V005\CURRENT_TASK.md`
- `E:\Project2026\RecoveryReports\InstallationV0\StationEnabledClosureV001\*`

Coordination report changed:

- `report/report060.md`

## 12. Builds/Tests Counts

Required builds/tests:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal
PASS, 0 warnings, 0 errors

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal
PASS, 176 warnings, 0 errors

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~MainWindow|FullyQualifiedName~Database|FullyQualifiedName~Station|FullyQualifiedName~Naming|FullyQualifiedName~Wiring" -v minimal
PASS, 189 passed, 0 failed, 0 skipped
```

Focused pre-check:

```text
CanonicalRuntimeWiringTests: PASS, 12 passed, 0 failed
```

## 13. Graphify/Evidence Folder/Hashes

Evidence folder:

```text
E:\Project2026\RecoveryReports\InstallationV0\StationEnabledClosureV001
```

Files:

- `README.md`
- `SHA256SUMS.txt`
- `remaining-modal-call-chain.mmd`
- `station-enabled-predicate-before-after.md`
- `physical-db-station-state.txt`
- `mirror-path-inventory.csv`
- `automatic-modal-scan.txt`
- `graphify\graphify-out\graph.json`

Hash manifest:

```text
E:\Project2026\RecoveryReports\InstallationV0\StationEnabledClosureV001\SHA256SUMS.txt
```

## 14. Prompt060 Label Proof

```text
Build label: prompt060
Window title: OBM InstallationV0 Phase 1/2 - prompt060
```

Source:

```text
InstallationV0BuildInfo.CoordinationPromptLabel = "prompt060"
InstallationV0Window title composes from InstallationV0BuildInfo.CoordinationPromptLabel
```

## 15. No DB/API/PIN/Process/Source-Push Mutation Proof

Confirmed:

- No PostgreSQL mutation.
- No seed/migration run.
- No Pairing Code redemption.
- No API token/contract change.
- No employee operational PIN change.
- No DB role/password/GRANT/REVOKE change.
- No WPF launch.
- No process restart/stop/start.
- No OBM source commit or push.
- No ASP.NET Identity table deletion.
- No raw GUID, password, token, or connection string printed.

## 16. Operator Physical Retest

Next operator retest:

```text
A. InstallationV0 prompt060 -> Open OBM-POS -> MainWindow visible -> no station modal.
B. OBM-POS Runtime Development -> same ProductRoot -> MainWindow direct -> no station modal.
C. API HTTP 401/unavailable -> MainWindow remains open -> local CRUD works.
```

## 17. Coordination Commit SHA

Pending until this report is committed and pushed. The final response will provide the exact pushed commit SHA.
