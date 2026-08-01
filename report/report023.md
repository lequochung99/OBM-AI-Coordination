# Report 023 — Prompt023 Phase 2 POS1 legacy reuse trial

## 1. Verdict

`PHASE2_LEGACY_REUSE_TRIAL_V001_READY_FOR_USER_TEST`

Safety stop classification: `BLOCKED_PHASE2_POS1_ROLLBACK_ANCHOR`.

Implementation and focused tests passed, but the physical POS1 database mutation was not executed because the mandatory rollback anchor did not contain a successful custom-format `pg_dump`, non-empty dump file, or checksum manifest.

## 2. Report019 scaffold assessment

Reused and adapted:

- Local `InstallationV0\Phase2` trial scaffold.
- Prompt label/build-info path.
- Phase 2 presentation surface.
- Deterministic plan-builder/test style.
- Transaction script-builder/in-memory verification approach.

Corrected from prompt019:

- Active label changed from `prompt019` to `prompt023`.
- Approved database changed to exact `obm_pos_dev_v0_pg`.
- Deny-list guard added for rejected targets.
- Previous `TblLocalOutbox delta = 0` assumption removed.
- Outbox is now part of the selected baseline atomic plan where the proven seed path emits outbox.

Not executed:

- No physical seed or second replay against PostgreSQL.
- No WPF Phase 2 operator action.

## 3. Exact approved target proof

Approved target encoded in source:

```text
obm_pos_dev_v0_pg
```

Hard rejects encoded:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
recovery_api_day16_pg
```

Guard behavior:

- exact database name required;
- environment must be `Development`;
- denied names fail even if otherwise supplied;
- non-approved names fail with `BLOCKED_PHASE2_POS1_TARGET_MISMATCH`.

## 4. Rollback anchor path/files/hashes

Chosen rollback anchor path:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV001\PreSeedBackup
```

Files present:

```text
pre-seed-table-counts.tsv
RESTORE-NOTES.md
sanitized-database-metadata.txt
```

Sanitized metadata result:

```text
PG_DUMP_FAILED
```

Missing required anchor files:

```text
custom-format pg_dump
SHA256SUMS.txt
```

Hash verification: not available because no successful dump/checksum manifest exists.

Result: rollback anchor failed closed before mutation.

## 5. Phase 1 revalidation/freeze proof

Physical Phase 2 execution was not reached, so no bootstrap credential was unprotected and no protected endpoint was called during this report.

Preserved Phase 1 artifacts:

```text
E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot\InstallationV0\Checkpoints\api-authorized.json
E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot\InstallationV0\Secrets\bootstrap-credential.dpapi
```

No Pairing Code was redeemed. No token or credential was printed.

## 6. Exact prior successful seed/save path found

Primary proven path located:

```text
E:\Project2026\4POS\NailSalonNet8\SeedDb\SeedDbProvider.cs
SeedDbProvider.RunLegacyDemoSeedAllAsync
```

Shared transaction path:

```text
E:\Project2026\4POS\NailSalonNet8\Services\MainServices.cs
MainServices.ExecuteInTransactionAsync
```

Outbox helper path:

```text
E:\Project2026\4POS\NailSalonNet8\Services\MainServices.cs
CreateLocalOutboxSingle / CreateLocalOutboxSingleAsync
```

Evidence classification: source-static proof. No production/reference database was read or mutated.

## 7. Reusable method matrix

| Method | Physical table | Tenant/POS behavior | Save/transaction behavior | Outbox behavior | Prompt023 decision |
|---|---|---|---|---|---|
| `RunLegacyDemoSeedAllAsync` | orchestrator | demo identity replaced by Phase 1 identity | caller-owned transaction through `ExecuteInTransactionAsync` | delegates to selected methods | adapt call list only |
| `ExecuteInTransactionAsync` | n/a | n/a | one transaction and commit boundary | n/a | reuse pattern |
| `SeedTenantAsync` | `TblTenant` | inject Phase 1 `TenantGuid` | `SaveChangesAsync` inside transaction | yes | adapt |
| `SeedPosAsync` | `TblPosLocal` | inject Phase 1 `TenantGuid`, `PosGuid`, POS name/slot | `SaveChangesAsync` inside transaction | no, local-only exception | adapt |
| `SeedSetupLoginMethodAsync` | `TblSetupLoginMethod` | tenant scoped | `SaveChangesAsync` inside transaction | yes | adapt |
| `SeedPaymentMethodsAsync` | `TblSetupPaymentMethod` | tenant scoped | `SaveChangesAsync` inside transaction | yes | adapt |
| `SeedEmployeePermissionAsync` | `TblEmployeePermission` | tenant scoped | `SaveChangesAsync` inside transaction | yes | adapt, filtered roles |
| `SeedSetupWeirdAsync` | `TblSetupWeird` | tenant scoped | `SaveChangesAsync` inside transaction | yes | adapt |
| `SeedSetupServicesMethodAsync` | `TblSetupServicesMethod` | tenant scoped | `SaveChangesAsync` inside transaction | yes | adapt |
| `SeedParameterSetingAsync` | `TblParameterSetting` | tenant scoped | `SaveChangesAsync` inside transaction | yes | adapt, minimal safe set |
| `SeedSetupPrinterAsync` | `TblSetupPrinter` | tenant scoped | `SaveChangesAsync` inside transaction | yes | adapt, complete logical printer set |

Excluded from v001:

```text
employees/PINs
service categories
services/products
customers
gift cards
invoice/output/runtime rows
terminal/payment runtime rows
queue/turn/payroll history
```

## 8. Selected baseline method/table call list

Selected v001 data rows:

| Order | Table | Rows |
|---:|---|---:|
| 1 | `TblTenant` | 1 |
| 2 | `TblPosLocal` | 1 |
| 3 | `TblSetupLoginMethod` | 3 |
| 4 | `TblSetupPaymentMethod` | 6 |
| 5 | `TblEmployeePermission` | 3 |
| 6 | `TblSetupWeird` | 1 |
| 7 | `TblSetupServicesMethod` | 1 |
| 8 | `TblParameterSetting` | 2 |
| 9 | `TblSetupPrinter` | 5 |
| 10 | `Phase2TrialCompletionMarker` | 1 |

Selected total data/marker rows: 24.

Planned `TblLocalOutbox` rows: 22.

## 9. Physical/logical dependency order

Order implemented in the plan:

```text
TblTenant
TblPosLocal
independent lookup/default rows
tenant/POS-scoped singleton/config rows
matching TblLocalOutbox rows
Phase2TrialCompletionMarker last
```

`Phase2SeedAuditV003` FK evidence did not force additional selected-table ordering beyond the logical parent identity dependency.

## 10. TenantGuid/PosGuid/PK/FK transformation behavior

Transformation rules encoded:

- `TenantGuid` comes from Phase 1 identity.
- `PosGuid` comes from Phase 1 identity where the entity has a POS identity column.
- stable row keys are deterministic per v001 plan.
- timestamps are execution-time values in the generated plan.
- foreign-key-like logical parent dependencies are ordered parent first.
- secret/private values are not copied from historical state.

## 11. Full `TblSetupPrinter` row list and preserved logical settings

Printer logical rows selected:

```text
CUSTOMER_TERMINAL
MERCHANT_TERMINAL
POS_CUSTOMER
POS_MERCHANT
POS_EMPLOYEE
```

The prompt023 plan preserves logical flags/default intent. These rows are logical print settings, not five physical printers.

## 12. Data-to-`TblLocalOutbox` mapping matrix

| Data table | Stable operation | Outbox required | Outbox entity | Operation | Source identity |
|---|---|---:|---|---|---|
| `TblTenant` | insert-or-verify | yes | table/entity name | `I` | Phase 1 POS source client |
| `TblPosLocal` | insert-or-verify | no | n/a | n/a | local-only station identity exception |
| `TblSetupLoginMethod` | insert-or-verify | yes | table/entity name | `I` | Phase 1 POS source client |
| `TblSetupPaymentMethod` | insert-or-verify | yes | table/entity name | `I` | Phase 1 POS source client |
| `TblEmployeePermission` | insert-or-verify | yes | table/entity name | `I` | Phase 1 POS source client |
| `TblSetupWeird` | insert-or-verify | yes | table/entity name | `I` | Phase 1 POS source client |
| `TblSetupServicesMethod` | insert-or-verify | yes | table/entity name | `I` | Phase 1 POS source client |
| `TblParameterSetting` | insert-or-verify | yes | table/entity name | `I` | Phase 1 POS source client |
| `TblSetupPrinter` | insert-or-verify | yes | table/entity name | `I` | Phase 1 POS source client |
| `Phase2TrialCompletionMarker` | marker-last | no | n/a | n/a | n/a |

Outbox rows are generated by the v001 plan builder and emitted in the generated transaction script beside the corresponding data rows.

## 13. One shared transaction proof across all SaveChanges

Source proof:

- `MainServices.ExecuteInTransactionAsync` is the proven shared transaction pattern.
- Prompt023 Phase 2 script builder emits one transaction boundary.
- Tests prove the in-memory executor rolls data, outbox, and marker together.

Physical proof: not executed because rollback anchor failed before mutation.

## 14. Marker-last and verification-before-commit proof

Source/test proof:

- completion marker stable key is `phase2-pos1-legacy-reuse-trial-v001:complete`;
- marker is ordered after selected baseline data and outbox rows;
- script builder and tests assert marker-last placement.

Physical proof: not executed.

## 15. Rollback/failure-injection evidence

Focused InstallationV0 tests include failure-injection coverage for rollback of:

```text
data rows
TblLocalOutbox rows
completion marker
```

Physical PostgreSQL rollback was not required because no mutation was attempted.

## 16. Same-version idempotent replay evidence

Source/test evidence:

- plan uses stable keys;
- insert-or-verify semantics are represented in the generated transaction script;
- in-memory replay tests prove no duplicate data/outbox/marker for the same version.

Physical second run: not executed because rollback anchor failed before first mutation.

## 17. Explicit no-seed runtime/user-data table proof

v001 excludes:

```text
TblEmployee and employee PIN/private rows
TblServiceCategory
TblService
TblProduct
TblCustomer*
TblGiftCard*
TblInvoice*
TblOutputInfo*
TblOutputInfoTam*
TblTerminalDejavoo
TblTerminalPaymentInfo
TblInvoiceBookingLink
booking/appointment rows
payment transaction/history tables
queue/turn/payroll runtime history
event delivery/log operational rows
```

No physical delta proof exists because no physical mutation occurred.

## 18. Exact source files changed

Prompt023 source files touched/adapted:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TrialConstants.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TargetSafetyGuard.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2LegacySeedMethod.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2OutboxPlanRow.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TrialPlan.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TrialPlanBuilder.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TrialVerificationResult.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\InMemoryPhase2TrialTransactionExecutor.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2PostgreSqlTransactionScriptBuilder.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2RollbackAnchorPrerequisite.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\CURRENT.md
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\legacy-reuse-v001\README.md
E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs
```

The OBM source worktree remains shared/dirty and was not committed or pushed.

## 19. Build/test commands and counts

Commands run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Results:

```text
InstallationV0 build: PASS, 0 warnings, 0 errors
NailSalonNet8 build: PASS, 176 warnings, 0 errors
Focused InstallationV0 tests: PASS, 37 passed, 0 failed, 0 skipped
```

Warnings were source-wide existing build warnings; no secret values are reproduced in this report.

## 20. Physical POS1 before/after counts

Not executed.

Reason:

```text
BLOCKED_PHASE2_POS1_ROLLBACK_ANCHOR
```

No before/after mutation counts were collected because prompt023 requires a verified rollback anchor before first mutation.

## 21. Second-run zero-delta proof

Not executed.

Reason: first physical run was blocked before mutation by missing rollback anchor dump/checksum.

## 22. WPF handoff and prompt023 label proof

Source/UI proof:

```text
OBM InstallationV0 Phase 1/2 - prompt023
Build label: prompt023
Phase 2 Local DB Baseline: Not Started
Target DB: obm_pos_dev_v0_pg (Development/Test)
Trial version: phase2-pos1-legacy-reuse-trial-v001
```

Tests assert active source no longer exposes prompt019 in the active title/header/build-info path.

WPF was not launched for Phase 2 physical handoff in this report.

## 23. First missing table/row/default observed

No physical WPF startup/runtime observation was made after v001 because the physical database seed did not run.

Next observation remains blocked until a valid rollback anchor exists.

## 24. No reference DB mutation/no secret leakage/source no-push proof

Confirmed:

- no reference database mutation;
- no production/protected database mutation;
- no physical mutation to `obm_pos_dev_v0_pg`;
- no Pairing Code redemption;
- no token/credential/connection string printed in this report;
- no OBM source commit;
- no OBM source push.

Coordination repo only receives this report.

## 25. Coordination commit SHA

This report is intended to be committed as:

```text
report/report023.md
```

The exact coordination commit SHA is reported by Codex after commit and push because a Git commit cannot embed its own final hash without changing that hash.

