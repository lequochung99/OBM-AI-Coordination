# Report 030 — Phase 2 v002 All Reference Employees

Generated: 2026-07-31

## 1. Verdict

`PHASE2_REFERENCE_DRIVEN_V002_ALL_EMPLOYEES_READY_FOR_USER_TEST`

The v002 employee upgrade source path is implemented and verified by build/tests. The physical employee seed was not executed in this Codex run; it remains behind the explicit WPF operator action.

## 2. Prompt029 Superseded

Confirmed. `prompt/prompt029.md` was not executed. `prompt/prompt030.md` is the active instruction.

## 3. Pre-v002 Backup Path And Hashes

Created backup anchor:

`E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV008\PreV002Backup`

Files:

- `obm_pos_dev_v0_pg.pre-v002.dump`
- `pg_restore-list.txt`
- `pre-v002-selected-counts.tsv`
- `sanitized-database-metadata.txt`
- `RESTORE-NOTES.md`
- `SHA256SUMS.txt`

Selected SHA-256 entries:

| File | SHA-256 |
| --- | --- |
| `obm_pos_dev_v0_pg.pre-v002.dump` | `688505879FB7FF59DE44DE7F8CB856CBFED4D3B199DF14D6430B94A1F8D5423B` |
| `pg_restore-list.txt` | `9408E9B9F7889ACB37489172EEFA0E1372E662BBB38B6EA8F16A195ADA910A8B` |
| `pre-v002-selected-counts.tsv` | `186222DCA2D00E9D08283A95EC07BD2924C9CA4B96C0B6A45158857FB9212808` |

Pre-v002 selected counts:

| Table | Count |
| --- | ---: |
| Phase2TrialCompletionMarker | 1 |
| TblEmployee | 0 |
| TblEmployeePermission | 3 |
| TblInvoice | 0 |
| TblLocalOutbox | 21 |
| TblOutputInfo | 0 |
| TblPosLocal | 2 |
| TblTenant | 2 |
| TblTurnLog | 0 |

## 4. Reference Employee Count And Type Distribution

Reference database read was performed with `BEGIN TRANSACTION READ ONLY` and `ROLLBACK`.

Safe proof:

- transaction_read_only: `on`
- database: `enailsalon_phasee1_pos1_pg`
- user: `hung`
- employee count: 20

Sanitized EmployeeType/permission distribution:

| EmployeeType | PermissionName | Count |
| ---: | --- | ---: |
| 1 | Admin | 2 |
| 1 | AI_Assistant | 1 |
| 1 | Manager | 1 |
| 1 | Owner | 2 |
| 1 | Staff | 9 |
| 1 | Sub_Manager | 1 |
| 2 | VirtualAnyTechnician | 4 |

No employee names, PINs, contacts, or private values are printed.

## 5. Employee Column Transformation Matrix

| Column class | Action |
| --- | --- |
| TenantGuid | replace from Phase 1 identity |
| EmployeeGuid | generate deterministic target GUID from v002 + Tenant/POS + reference stable key |
| EmployeePermissionGuid | remap to deterministic target permission row by PermissionName |
| NickName | preserve local trial display value, not reported publicly |
| EmployeeType/PermissionName/status/order/UI flags | preserve for WPF management and checkout filtering |
| UpdatedDate/UpdatedAt/CreatedAt | derive at execution time |
| BaoLuong | reset to zero |
| reference Tenant/row identity | never copy as target identity |

## 6. Private/Security Fields Excluded

Fields reset or excluded from v002 employee payload:

- `LoginNumber`
- `BirthDay`
- `Email`
- `CellPhone`
- `PhotoPath`
- `SecurityNumber`
- `SettingDate`
- password/pass/secret/token/auth/API/connection-string-like fields
- machine-private bindings

No shared hard-coded PIN is introduced.

## 7. Employee Dependency/Child-Table Classification

| Table | Classification |
| --- | --- |
| TblTenant | required parent |
| TblEmployeePermission | required parent |
| TblEmployee | selected v002 data table |
| TblEmployeeServiceCapability | deferred, depends on service/catalog/queue context |
| TblOutputInfo/TblOutputInfoTam | runtime/history, excluded |
| TblTurnLog/TblTurnState/TblTurnQueueEmployee | runtime/history, excluded |
| TblQueueTicket/TblQueueWorkUnit | runtime/history, excluded |
| TblPayrollPaycheck | payroll history/private, excluded |
| TblWebEmployeeWorkingHour | optional child, deferred until UI proves needed |

## 8. Inserted/Adopted/Conflicting Counts

Physical v002 seed was not executed.

Expected behavior implemented:

- absent employee stable key: insert transformed employee row;
- compatible existing employee: adopt/verify;
- incompatible stable key/type/identity: rollback with conflict;
- extra target employees outside v002: preserve.

## 9. Employee Outbox Mapping And Delta

Static mapping implemented:

- inserted/materially updated `TblEmployee` rows require deterministic `TblLocalOutbox` entries;
- outbox payload is sanitized and does not include PIN/contact/private/payroll values;
- compatible no-op adoption does not duplicate outbox.

Physical outbox delta is pending operator action.

## 10. One-Transaction/Marker-Last Proof

Static/source proof:

- executor uses one target `NpgsqlConnection`;
- executor uses one serializable target transaction;
- advisory transaction lock uses Tenant/POS/version;
- employee data, included outbox, and v002 marker share the same transaction;
- rollback path rolls back selected employee data, outbox, and marker;
- marker version is `phase2-reference-driven-trial-v002-employees` and is written last.

Physical marker-last proof is pending operator action.

## 11. Startup Marker Hydration Proof

WPF state copy now exposes:

- `v001 complete; Employee v002 Upgrade Available`
- `Phase 2 v002 Complete` after successful action result

Physical restart/hydration proof is pending operator action.

## 12. Management UI And Checkout Filtering Proof

Static/UI proof:

- management screens filter non-staff/management rows by `EmployeeType` and `PermissionName`;
- checkout/staff paths filter Staff rows through existing logic;
- prompt030 keeps the existing filtering logic intact.

Physical WPF management/checkout proof is pending v002 seed and operator test.

## 13. Same-Version Replay Zero-Delta Proof

Static idempotency model is implemented through deterministic row/outbox/marker identities.

Physical replay was not run.

## 14. Excluded Runtime/History Table Proof

Pre-v002 runtime/history samples were zero for:

- `TblInvoice`
- `TblOutputInfo`
- `TblTurnLog`

Prompt030 source classifies employee child/history/runtime tables as excluded or deferred.

## 15. Source Files Changed

Prompt030-related source files:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TrialConstants.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2ReferenceSeedOptions.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2ReferenceDrivenManifest.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\PostgreSqlPhase2ReferenceSeedExecutor.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\CURRENT.md`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\reference-driven-v002-employees\README.md`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

## 16. Build/Test Commands And Counts

Commands:

- `dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj`: PASS, 0 warnings, 0 errors.
- `dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj`: PASS, 176 warnings, 0 errors.
- `dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"`: PASS, 40 passed, 0 failed, 0 skipped.

## 17. Prompt030 Label Proof

Active build label:

`prompt030`

Active title pattern:

`OBM InstallationV0 Phase 1/2 - prompt030`

Focused tests assert prompt028/prompt029 are absent from the active build-info/window label.

## 18. No Reference Mutation / No Secret Leakage / No Source Push

Confirmed:

- reference DB mutation: none;
- target DB seed mutation: none;
- backup used approved admin pgpass path without reading or printing credential content;
- no employee names, PINs, contact data, payroll values, tokens, passwords, or connection strings are printed in this report;
- OBM source repo was not committed or pushed;
- only this coordination report is pushed.

## 19. Remaining Missing Dependency For v003

Pending physical WPF proof may reveal whether safe employee child/default rows are required.

Likely deferred areas:

- employee service capabilities, because services/catalog are not seeded;
- web working hours, because they are optional until the WPF UI proves a startup dependency;
- service/catalog import remains a later v003 topic if needed.

## 20. Coordination Commit SHA

Final pushed commit SHA is reported by Codex after commit/push. Embedding the final SHA inside this file would change the commit hash.
