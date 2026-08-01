# Report 033 — Phase 2 v002 Employee Permission FK Correction

Generated: 2026-08-01

## 1. Verdict

`PHASE2_V002_EMPLOYEE_PERMISSION_FK_FIX_READY_FOR_USER_RETEST`

Source correction is implemented and verified by build/tests. The physical WPF Phase 2 v002 retry was not run by Codex.

## 2. Physical 23503 Evidence

Operator-reported physical retest of build `prompt032` failed with safe PostgreSQL metadata:

- SQLSTATE: `23503`
- schema: `dbo`
- table: `TblEmployee`
- constraint: `FK_TblEmployee_Permission`

Classification:

`PHASE2_V002_EMPLOYEE_PERMISSION_PARENT_MISSING`

No employee names, PINs, contacts, payroll values, secrets, connection strings, tokens, or raw rows are reported.

## 3. Post-Failure Rollback Proof

Read-only target proof was performed with `BEGIN TRANSACTION READ ONLY` and `ROLLBACK`.

| Target proof | Count/state |
| --- | ---: |
| `TblEmployee` | 0 |
| `TblEmployeePermission` | 3 |
| `TblLocalOutbox` | 21 |
| v001 marker | 1 |
| v002 marker | 0 |
| `TblPosRuntimeProfile` | 1 |
| runtime profile state | `Activated` |
| `TblPosRuntimeStateHistory` | 1 |

No v002 employee rows, v002 marker, employee outbox rows, or runtime-history transition committed.

## 4. Exact FK Child/Parent Metadata

PostgreSQL metadata:

| Item | Value |
| --- | --- |
| child table | `dbo.TblEmployee` |
| constraint | `FK_TblEmployee_Permission` |
| child FK column | `EmployeePermissionGuid` |
| child FK nullable | `NO` |
| parent table | `dbo.TblEmployeePermission` |
| parent column | `EmployeePermissionGuid` |
| parent key | primary key `TblEmployeePermission_pkey(EmployeePermissionGuid)` |
| parent unique stable key | `UQ_TblEmployeePermission_Name(PermissionName)` |
| `TblEmployee.TenantGuid` nullable | `NO` |
| `TblEmployeePermission.TenantGuid` nullable | `NO` |
| ON DELETE | `NO ACTION` |
| ON UPDATE | `NO ACTION` |

## 5. Exact Root Cause

Prompt032 preserved the employee transform but still assigned:

`TblEmployee.EmployeePermissionGuid`

from an independently derived deterministic GUID based on `PermissionName`.

That GUID was not guaranteed to be the actual `TblEmployeePermission.EmployeePermissionGuid` present in the target transaction. The employee insert therefore referenced a parent GUID that did not exist physically and hit `FK_TblEmployee_Permission`.

## 6. Verified Required Permission-Parent Set

Reference database was read through a read-only transaction and rolled back.

Distinct selected employee permission labels:

| PermissionName | Employee count |
| --- | ---: |
| `Admin` | 2 |
| `AI_Assistant` | 1 |
| `Manager` | 1 |
| `Owner` | 2 |
| `Staff` | 9 |
| `Sub_Manager` | 1 |
| `VirtualAnyTechnician` | 4 |

Final required parent set count: 7.

## 7. Existing/Adopted/Inserted Permission Counts

Target read-only proof showed existing compatible parent labels:

- `Admin`
- `Owner`
- `Sub_Manager`

Expected first corrected run:

- required permission parents: 7
- existing/adopted parents: 3
- missing/inserted parents: 4

The four missing parents are derived from the verified reference employee labels, not hard-coded as a blind replacement list.

## 8. Proof `TblEmployeePermission` Is Seeded Before `TblEmployee`

Source now separates:

- `permissionRows`
- `employeeRows`
- `regularRows`

Execution order inside the target transaction:

1. target guard, rollback anchor, advisory lock, v001 marker verification;
2. seed/adopt `TblEmployeePermission`;
3. read back actual permission parents;
4. resolve each employee FK from the readback map;
5. verify parent GUIDs physically exist;
6. insert/adopt `TblEmployee`;
7. insert required permission/employee outbox rows;
8. verify runtime profile/history;
9. write v002 marker last;
10. commit once.

Tests assert the source ordering and that the old independent permission GUID assignment is absent.

## 9. Actual Parent-GUID Mapping Design

Stable parent key:

`TenantGuid + PermissionName`

Implementation:

- missing permission labels create deterministic parent rows;
- compatible existing rows are adopted;
- readback query returns `PermissionName`, actual `EmployeePermissionGuid`, `TenantGuid`, and `IsActive`;
- duplicate parent labels fail closed with `PHASE2_PERMISSION_DUPLICATE_PARENT`;
- tenant or active-state conflict fails closed with `PHASE2_PERMISSION_BASELINE_CONFLICT`;
- missing parent map entries fail closed with `PHASE2_EMPLOYEE_PERMISSION_PARENT_MISSING`;
- each employee FK is assigned from the actual readback map.

## 10. Permission And Employee Outbox Policy

Permission and employee outbox rows are inserted only for rows actually inserted by this run.

Policy:

- permission events use `EntityType = TblEmployeePermission`;
- employee events use `EntityType = TblEmployee`;
- payload remains sanitized and excludes raw private employee data;
- compatible no-op replay does not duplicate outbox rows;
- all permission, employee, outbox, runtime, and marker work share the same transaction.

## 11. One-Transaction/Marker-Last Proof

The executor still uses:

- one target `NpgsqlConnection`;
- one serializable target transaction;
- one advisory transaction lock;
- one final `CommitAsync`;
- rollback on any ordinary or PostgreSQL exception.

The marker remains:

`phase2-reference-driven-trial-v002-employees`

It is still selected and written after permission parent resolution, employees, outbox, and runtime verification.

## 12. Runtime Profile/History Behavior

Prompt031 runtime policy is preserved:

- `TblPosRuntimeProfile` is the current runtime state source of truth;
- `TblPosRuntimeStateHistory` is append-only on a real transition;
- current physical profile is already `Activated`;
- expected corrected first-run history delta is 0.

## 13. Expected First-Run Deltas

Based on verified target/reference state:

| Table/component | Expected delta |
| --- | ---: |
| `TblEmployeePermission` | +4 |
| `TblEmployee` | +20 |
| `TblLocalOutbox` | +24 |
| `TblPosRuntimeProfile` | 0 |
| `TblPosRuntimeStateHistory` | 0 |
| v002 marker | +1 |

The outbox delta is expected to be 4 permission events plus 20 employee events.

## 14. Same-Version Replay Policy

Second execution of the same v002 marker/version should produce:

- `TblEmployeePermission` delta: 0
- `TblEmployee` delta: 0
- `TblLocalOutbox` delta: 0
- `TblPosRuntimeProfile` delta: 0
- `TblPosRuntimeStateHistory` delta: 0
- v002 marker delta: 0

Physical replay proof is pending operator retest.

## 15. Source Files Changed

Prompt033-related source/test/artifact files:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\PostgreSqlPhase2ReferenceSeedExecutor.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\reference-driven-v002-employees-r2\README.md`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

## 16. Build/Test Commands And Counts

Commands:

- `dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj`: PASS, 0 warnings, 0 errors.
- `dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj`: PASS, 176 warnings, 0 errors.
- `dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"`: PASS, 43 passed, 0 failed, 0 skipped.

Warnings in the full WPF/test build are existing project warnings and were not introduced as blocking errors for this prompt.

## 17. Prompt033 Label Proof

Active build label:

`prompt033`

Active title pattern remains:

`OBM InstallationV0 Phase 1/2 - {InstallationV0BuildInfo.CoordinationPromptLabel}`

Tests assert `prompt032` and older labels are absent from the active build-info path.

## 18. No Reference Mutation / No Secret Leakage / No Source Push

Confirmed:

- reference database mutation: none;
- target manual repair/cleanup: none;
- physical WPF retry: not run;
- no employee names, PINs, contact values, payroll values, passwords, tokens, connection strings, or raw GUID maps printed;
- OBM source repo was not committed or pushed;
- only this coordination report is committed and pushed.

## 19. Exact Operator Retest Steps

Operator retest should:

1. start corrected WPF build showing `prompt033`;
2. use the explicit `Install Local Database Baseline` action;
3. verify no `23503` FK failure;
4. verify `TblEmployeePermission` parent delta +4;
5. verify `TblEmployee` delta +20;
6. verify `TblLocalOutbox` delta +24;
7. verify runtime history delta 0 while state remains `Activated`;
8. verify v002 marker written last;
9. run same action again and verify zero-delta replay.

## 20. Coordination Commit SHA

Final pushed commit SHA is reported by Codex after commit/push. Embedding the final SHA inside this file would change the commit hash.
