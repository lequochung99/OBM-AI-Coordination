# Report 035 — Phase 2 v002 Full Permission Default Reconciliation

Generated: 2026-08-01

## 1. Verdict

`PHASE2_V002_FULL_PERMISSION_AND_EMPLOYEE_SEED_READY_FOR_USER_RETEST`

Source correction is implemented and verified by build/tests. Physical employee seed was not run by Codex; the corrected WPF/operator path is ready for retest.

## 2. Exact Target Has 3 Pre-v002 Permissions

Read-only target proof:

- database: `obm_pos_dev_v0_pg`
- role: `hung`
- mode: `BEGIN TRANSACTION READ ONLY`
- terminal action: `ROLLBACK`

Current target state:

| Proof | Value |
| --- | ---: |
| `TblEmployeePermission` | 3 |
| `TblEmployee` | 0 |
| `TblLocalOutbox` | 21 |
| v001 marker | 1 |
| v002 marker | 0 |
| `TblPosRuntimeProfile` | 1 |
| runtime profile state | `Activated` |
| `TblPosRuntimeStateHistory` | 1 |

## 3. Why This Is Expected

Prompt035 supersedes report034's blocker interpretation.

The target having only these three v001 permission defaults is a valid pre-v002 state:

- `Admin`
- `Owner`
- `Sub_Manager`

The InstallationV0 v002 transaction must reconcile the complete canonical permission-default set from the reference database before inserting employees.

## 4. Reference Permission Schema/Default Inventory

Reference permission schema was inspected read-only:

| Column | Type | Nullable |
| --- | --- | --- |
| `EmployeePermissionGuid` | uuid | NO |
| `TenantGuid` | uuid | NO |
| `PermissionName` | varchar(50) | NO |
| `Description` | varchar(255) | YES |
| `IsActive` | boolean | NO |
| `CreatedAt` | timestamp | NO |
| `UpdatedAt` | timestamp | YES |

Canonical reference labels:

| PermissionName | Safe default fields |
| --- | --- |
| `Admin` | description/default flag |
| `AI_Assistant` | description/default flag |
| `Manager` | description/default flag |
| `Owner` | description/default flag |
| `Staff` | description/default flag |
| `Sub_Manager` | description/default flag |
| `VirtualAnyTechnician` | description/default flag |

No raw GUID maps, secrets, or employee private values are reported.

## 5. Existing Permission Canonical Comparison

The existing target rows for `Admin`, `Owner`, and `Sub_Manager` match the reference safe defaults for the inspected fields:

- `Description`
- `IsActive`

Therefore the expected prompt035 first run should adopt these three rows and preserve their actual target `EmployeePermissionGuid` values.

If future safe default fields differ, source now updates them in place instead of treating ordinary v001-to-v002 default reconciliation as a conflict.

## 6. Inserted/Updated/Adopted Permission Counts

Expected first corrected run from current state:

| Permission category | Expected count |
| --- | ---: |
| adopted existing rows | 3 |
| updated existing rows | 0 |
| inserted missing rows | 4 |
| total canonical parents after run | 7 |

Updated count can become nonzero only if safe canonical defaults differ at runtime.

## 7. Parent-Before-Employee Proof

Source now:

- reads all `TblEmployeePermission` defaults from the reference DB by setting the manifest rule to `Set()`;
- reconciles permission parents before employee insert;
- preserves actual target parent GUID for adopted rows;
- inserts missing parents before employees;
- reads back the `PermissionName -> actual EmployeePermissionGuid` map;
- resolves every employee FK from that map before the first `TblEmployee` insert.

## 8. Employee And Permission Outbox Policy/Deltas

Source now emits deterministic outbox work as:

- permission insert: operation `I`;
- permission safe-default update: operation `U`;
- employee insert: operation `I`;
- compatible no-op adoption: no new outbox.

Expected first-run outbox delta:

- 4 permission `I` events;
- 0 permission `U` events from current inspected safe defaults;
- 20 employee `I` events;
- expected total: 24.

If runtime comparison finds safe default differences, one `U` event is added per corrected existing permission row.

## 9. Transaction/Rollback/Marker-Last Proof

The executor continues to use:

- one target `NpgsqlConnection`;
- one serializable target transaction;
- one tenant/POS/v002 advisory transaction lock;
- rollback on any ordinary or PostgreSQL exception;
- unchanged marker version `phase2-reference-driven-trial-v002-employees`;
- v002 marker after permissions, employees, outbox, and runtime verification.

## 10. Runtime Profile/History Proof

Current runtime state remains:

- `TblPosRuntimeProfile` count: 1
- `RuntimeState`: `Activated`
- `TblPosRuntimeStateHistory` count: 1

Expected corrected first-run runtime deltas:

- profile delta: 0
- history delta: 0

## 11. Physical Execution Result

Not run.

Reason:

Codex did not run WPF or a physical harness. The prompt allows a harness only if it invokes the same production executor and validates the V008 backup anchor; this run stopped after source/build/test verification and leaves the physical action to the operator.

## 12. Replay Result/Policy

Replay was not run because first physical v002 execution has not yet occurred.

Expected same-version replay after first success:

- `TblEmployeePermission` delta: 0
- `TblEmployee` delta: 0
- `TblLocalOutbox` delta: 0
- `TblPosRuntimeProfile` delta: 0
- `TblPosRuntimeStateHistory` delta: 0
- v002 marker delta: 0

## 13. Restart Hydration Proof

Physical restart/UI hydration was not run.

Expected after successful retest:

- InstallationV0 reads the v002 marker/runtime tables;
- displays `Phase 2 v002 Complete`;
- shows `RuntimeState: Activated`;
- does not reset the action to `Not Started` when invariants exist.

## 14. Source Files Changed

Prompt035-related source/test/artifact files:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2ReferenceDrivenManifest.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\PostgreSqlPhase2ReferenceSeedExecutor.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\reference-driven-v002-employees-r3\README.md`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

## 15. Build/Test Counts

Commands:

- `dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj`: PASS, 0 warnings, 0 errors.
- `dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj`: PASS, 176 warnings, 0 errors.
- `dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"`: PASS, 43 passed, 0 failed, 0 skipped.

## 16. Active Label Proof

Active build label:

`prompt035`

The label changed because source changed for prompt035.

## 17. No Reference Mutation / No Secret Leakage / No Source Push

Confirmed:

- reference database mutation: none;
- target database mutation: none;
- WPF physical retry: not run;
- no ad-hoc employee SQL seed was run;
- no employee names, PINs, contact values, payroll values, passwords, tokens, connection strings, or raw GUID maps printed;
- OBM source repo was not committed or pushed;
- only this coordination report is committed and pushed.

## 18. Coordination Commit SHA

Final pushed commit SHA is reported by Codex after commit/push. Embedding the final SHA inside this file would change the commit hash.
