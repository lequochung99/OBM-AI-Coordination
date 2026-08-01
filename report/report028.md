# Report 028 — Phase 2 Reference-Driven Trial v001

Generated: 2026-07-31T21:53:39-04:00

## 1. Verdict

`PHASE2_REFERENCE_DRIVEN_TRIAL_V001_READY_FOR_USER_TEST`

Implementation is wired for the explicit WPF operator action. The physical seed was not executed outside the WPF action boundary.

## 2. Prompt027 Superseded

Confirmed. `prompt/prompt028.md` supersedes prompt027 as the active execution instruction for this report.

## 3. Reference Read-Only Proof

Reference database was queried with the approved read-only pgpass path and read-only transaction settings.

- transaction_read_only: `on`
- current_database: `enailsalon_phasee1_pos1_pg`
- current_user: `hung`
- reference transaction: `BEGIN TRANSACTION READ ONLY` followed by `ROLLBACK`
- reference mutation: none

Selected safe reference counts observed:

| Table | Count |
| --- | ---: |
| TblEmployeePermission | 7 |
| TblParameterSetting | 110 |
| TblSetupLoginMethod | 3 |
| TblSetupPaymentMethod | 6 |
| TblSetupPrinter | 5 |
| TblSetupServicesMethod | 1 |
| TblSetupWeird | 1 |

## 4. Valid V007 Rollback Anchor Proof

Rollback anchor verified as the current valid pre-seed anchor:

`E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV007\PreSeedBackup`

Key files present from the approved anchor:

- `obm_pos_dev_v0_pg.preseed.dump`
- `pg_restore-list.txt`
- `pre-seed-table-counts.tsv`
- `SHA256SUMS.txt`
- `RESTORE-NOTES.md`
- runtime privilege proof artifacts

Known SHA-256 entries from the accepted V007 anchor:

| Artifact | SHA-256 |
| --- | --- |
| preseed dump | `54E18F6C4DFE66D0F404D9A0FF984DCCCDD4A1B1C5FA01FFEC199AD8AD089684` |
| pg_restore list | `52460058A85AB1480FD91525D614562226572604AADCD197F1C8173EB82C37EA` |
| pre-seed counts | `1F322B2F3CFB83C4700BF5C63C0615018D55EDBDFDF7741B4F16FD64A0BBE3A3` |

## 5. Sanitized Reference-Driven Manifest

Implemented manifest source:

`E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2ReferenceDrivenManifest.cs`

Selected tables:

| Table | Stable key | Reference selected count rule | Outbox |
| --- | --- | --- | --- |
| TblTenant | Phase 1 TenantGuid | 1 local identity row | Yes |
| TblPosLocal | Phase 1 PosGuid/Station | 1 local identity row | No |
| TblSetupLoginMethod | LoginMethodName | all reference rows | Yes |
| TblSetupPaymentMethod | PaymentName | all reference rows | Yes |
| TblEmployeePermission | PermissionName | Owner, Admin, Sub_Manager | Yes |
| TblSetupWeird | SetupWeirdGuid | reference singleton | Yes |
| TblSetupServicesMethod | SetupServicesMethodGuid | reference singleton | Yes |
| TblParameterSetting | PropertyName | sanitized safe keys only | Yes |
| TblSetupPrinter | PrinterType | all logical reference rows | Yes |
| Phase2TrialCompletionMarker | stable version marker | 1 marker | No |

Explicit no-seed runtime/business tables are guarded in the manifest.

## 6. Parameter Classification

Final included safe-key policy is reference-driven, not arbitrary-count driven.

Observed safe reference key:

- `Date Hien Tai`: present in the reference selection and included by manifest rule.

Allowed safe startup/common key:

- `EnableTurnEngine`: included by manifest rule when present; not invented when absent from the reference query.

Excluded parameter categories:

- salon historical/custom values
- environment or machine-specific values
- secret/private/gateway/terminal credential values
- duplicate or ambiguous scope rows

## 7. Printer Row/Type List And Column Matrix

Reference printer row count: 5.

Safe logical rows observed:

| PrinterType | Logical name | Active |
| ---: | --- | --- |
| 0 | Customer Invoice From The Terminal | false |
| 1 | Merchant Invoice From The Terminal | false |
| 2 | Customer Invoice From The POS | true |
| 3 | Merchant Invoice From The POS | false |
| 4 | Employee Invoice From The POS | false |

Column transformation matrix:

| Column class | Action |
| --- | --- |
| TenantGuid | replace from Phase 1 identity |
| PosGuid/POS scope | replace from Phase 1 POS identity |
| row GUID | deterministic target GUID |
| FK GUID | remap to selected deterministic parent |
| logical print flags | preserve canonical logical default |
| physical printer path/share/IP/driver/machine binding | clear as machine-private |
| timestamps | derive at execution time |
| reference identity GUIDs | never copy to target identity |

## 8. Selected Rows

Reference selections recorded:

- Employee permissions selected: `Owner`, `Admin`, `Sub_Manager`
- Login methods: 3 reference rows
- Payment methods: 6 reference rows
- Setup weird singleton: 1 reference row
- Setup services method singleton: 1 reference row
- Setup printer: 5 reference rows

No employee private rows, PINs, customers, invoices, gift cards, services, service categories, or historical operational rows are selected.

## 9. Dependency/FK Order

Implemented high-level order:

1. Phase 1 identity revalidation.
2. `TblTenant`.
3. `TblPosLocal`.
4. independent lookup/default tables.
5. tenant/POS scoped singleton/config tables.
6. deterministic `TblLocalOutbox` rows for tables that require outbox.
7. `dbo."Phase2TrialCompletionMarker"` last.

The executor performs selected data and marker work inside one target transaction.

## 10. Identity/GUID/FK Transformation Rules

Rules implemented:

- Tenant identity comes from Phase 1.
- POS identity comes from Phase 1.
- target row GUIDs are deterministic from version, target identity, table, and stable key.
- selected FK values are remapped to deterministic/new parent rows.
- timestamps are execution-derived.
- reference Tenant/POS/row GUIDs are not copied as target identities.
- target database is hard-guarded to `obm_pos_dev_v0_pg`.
- reference database is hard-guarded to `enailsalon_phasee1_pos1_pg`.

## 11. Reused Legacy Save/Outbox Methods

The implementation preserves the proven legacy save/outbox policy rather than copying historical reference outbox rows.

Outbox policy implemented:

- `TblTenant`: outbox
- `TblPosLocal`: no outbox, local-only exception
- setup/login/payment/permission/weird/services/parameter/printer rows: outbox
- marker: no outbox

## 12. Real Executor And Operator Action

Implemented real PostgreSQL executor:

`E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\PostgreSqlPhase2ReferenceSeedExecutor.cs`

Implemented WPF explicit operator action:

`E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`

The UI action is labelled `Install Local Database Baseline` and is not auto-run on WPF startup.

## 13. Source Files Changed

Prompt028-related source files:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TrialConstants.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2ReferenceSeedOptions.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2ReferenceDrivenManifest.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\PostgreSqlPhase2ReferenceSeedExecutor.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\CURRENT.md`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\reference-driven-v001\README.md`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

## 14. One-Transaction/Marker-Last Proof

Static proof from source and tests:

- executor opens one target `NpgsqlConnection`
- target operation uses one transaction
- advisory transaction lock is taken in the target transaction
- selected data and outbox operations occur before marker
- completion marker is written last
- rollback path rolls back selected data, outbox, and marker together
- tests assert real PostgreSQL executor, read-only reference path, one transaction, and marker-last behavior

Physical marker-last proof is pending the explicit WPF operator action.

## 15. Physical Target Counts

Not executed in this run. The physical seed was intentionally left behind the explicit WPF operator action.

Status: pending operator click.

## 16. Inserted Versus Adopted Rows

Not physically measured yet.

Expected executor behavior:

- absent selected stable key: insert transformed target row and required outbox
- compatible existing row: adopt/verify and add missing deterministic outbox if required
- conflicting row: rollback with `PHASE2_BASELINE_CONFLICT`
- extra rows outside selected manifest: preserve

## 17. Outbox Mapping And Physical Deltas

Static outbox policy is implemented. Physical outbox deltas are pending operator action.

Same-version replay is designed to produce zero new outbox rows for existing deterministic baseline events.

## 18. Runtime/Excluded Table Zero Delta

Physical zero-delta proof is pending operator action.

Explicit no-seed tables remain excluded by source manifest and tests.

## 19. Same-Version Replay Zero Delta

Physical replay was not executed.

The implemented stable-key/deterministic-row/outbox model is intended to make same-version replay zero delta for selected data, outbox, marker, and excluded runtime tables.

## 20. Build/Test Counts

Commands run:

- `dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj`: PASS, 0 warnings, 0 errors.
- `dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj`: PASS, 0 warnings, 0 errors.
- `dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"`: PASS, 39 passed, 0 failed, 0 skipped.

## 21. WPF Prompt028 Handoff

Build label is set to `prompt028`.

Window title target:

`OBM InstallationV0 Phase 1/2 - prompt028`

Operator should start the latest WPF build and click the explicit `Install Local Database Baseline` action only after confirming Phase 1 checkpoint and target lane.

## 22. First Missing Default/Table

No physical WPF runtime trial was executed after the new Phase 2 action, so no missing default/table has been observed yet.

Current classification: v001 ready for operator user test.

## 23. No Reference Mutation / No Secret Leakage / No Source Push

- Reference DB mutation: none.
- Target DB mutation: none in this Codex run.
- Passwords/tokens/connection strings printed: none.
- Source repo push: none.
- Coordination repo push: report only.

## 24. Coordination Commit SHA

Final pushed commit SHA is reported by Codex after commit/push. Embedding the final SHA inside this file would change the commit hash.
