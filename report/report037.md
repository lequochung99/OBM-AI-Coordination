# Report 037 - Phase 1 to Phase 2 Identity Spine Lock

## 1. Verdict

PHASE2_V002_IDENTITY_SPINE_REPAIR_READY_FOR_USER_RETEST

Prompt037 source correction is implemented and verified by build/tests. The final physical WPF seed was not run.

## 2. Physical Runtime-Profile Identity Mismatch Evidence

The operator physically retested prompt036 and received:

`PHASE2_RUNTIME_PROFILE_IDENTITY_MISMATCH`

Read-only comparison against the current durable Phase 1 checkpoint showed:

- TblTenant exact Phase 1 match: yes
- TblPosLocal exact Phase 1 match: yes
- TblPosRuntimeProfile TenantGuid match: no
- TblPosRuntimeProfile TenantCode match: no
- TblPosRuntimeProfile PosGuid match: no
- TblPosRuntimeProfile slot match: yes
- TblPosRuntimeProfile InstallationGuid match: no
- TblPosRuntimeProfile DatabaseName match: yes
- TblPosRuntimeProfile EnvironmentName match: yes
- RuntimeState: Activated

Classification:

`C. Runtime profile points to an older Development identity while correct Phase 1 local parent rows exist`

No raw GUIDs, tokens, pairing codes, passfiles, passwords, or connection strings are recorded.

## 3. Post-Failure Rollback Proof

Read-only target verification used `BEGIN TRANSACTION READ ONLY` and `ROLLBACK`.

Current target counts:

- TblTenant: 2
- TblPosLocal: 2
- TblPosRuntimeProfile: 1
- TblPosRuntimeStateHistory: 1
- TblEmployeePermission: 3
- TblEmployee: 0
- TblLocalOutbox: 21
- v001 marker: 1
- v002 marker: 0
- RuntimeState: Activated

No prompt037 database mutation was performed.

## 4. Durable Phase 1 Identity Inventory, Sanitized

Checkpoint source:

`E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot\InstallationV0\Checkpoints\api-authorized.json`

Sanitized fields observed:

- checkpoint phase: ApiAuthorized
- TenantCode: OBMDEVV0
- PosName: POS1
- Slot: 1
- TenantGuid present: yes
- TenantName present: yes
- PosGuid present: yes
- InstallationAttemptGuid present: yes
- LocalInstallationGuid present: yes
- protectedCredentialReference present under current contract naming: yes
- EnvironmentName in checkpoint: not present
- DatabaseName in checkpoint: not present

The current Phase2 implementation derives DatabaseName from the approved target constant and EnvironmentName from the approved Development gate.

## 5. TblTenant Identity Inventory And Materialization Decision

Physical inventory:

- TblTenant rows: 2
- exact current Phase 1 tenant row exists: yes

Decision:

- Adopt and verify the compatible current Phase 1 row.
- Do not delete unrelated legacy tenant rows in prompt037.
- Missing current row would be inserted from Phase 1 identity.
- Same TenantCode with incompatible TenantGuid fails `PHASE2_TENANT_IDENTITY_CONFLICT`.

## 6. TblPosLocal Identity Inventory And Materialization Decision

Physical inventory:

- TblPosLocal rows: 2
- exact current Phase 1 POS row exists: yes

Decision:

- Adopt and verify the compatible current Phase 1 POS row.
- Do not delete unrelated legacy POS rows in prompt037.
- Missing current row would be inserted from Phase 1 identity.
- Same TenantGuid/slot with incompatible PosGuid fails `PHASE2_POS_IDENTITY_CONFLICT`.

## 7. TblPosRuntimeProfile Identity Inventory

Physical inventory:

- exactly one runtime profile row: yes
- RuntimeState: Activated
- DatabaseName: approved target
- EnvironmentName: Development
- Tenant/POS/install identity: stale relative to current Phase 1

Prompt037 repair behavior:

- preserves RuntimeProfileGuid
- safely aligns TenantGuid, TenantCode, PosGuid, PosSlotNumber, DeviceGuid, InstallationGuid, DeviceRegistrationId, SourceClientId, DatabaseName, and EnvironmentName
- does not change RuntimeState when already Activated
- identity-only rebind produces runtime-history delta 0

## 8. Existing Marker Identity Context

Existing baseline marker:

- v001 marker count: 1
- v002 marker count: 0
- marker table physical columns: BaselineVersion, AppliedAtUtc, ToolName

The v002 marker remains:

`phase2-reference-driven-trial-v002-employees`

The new marker hard gate verifies the in-memory marker TenantGuid/PosGuid context before marker insertion. The physical marker schema itself does not expose TenantGuid/PosGuid columns.

## 9. Exact Mismatched Field Names / Classification

Mismatched physical categories:

- TblPosRuntimeProfile.TenantGuid
- TblPosRuntimeProfile.TenantCode
- TblPosRuntimeProfile.PosGuid
- TblPosRuntimeProfile.InstallationGuid

Matching categories:

- TblTenant Phase 1 identity
- TblPosLocal Phase 1 identity
- TblPosRuntimeProfile.PosSlotNumber
- TblPosRuntimeProfile.DatabaseName
- TblPosRuntimeProfile.EnvironmentName
- TblPosRuntimeProfile.RuntimeState

Classification: category C.

## 10. Legacy Dependent-Data Audit

Read-only dependent-data audit showed zero rows in operational conflict categories, including invoices, output info, terminal/payment transactions, booking/appointment, queue/turn history, payroll, customers, gift cards, services, and actual employees.

Nonzero setup/reference data observed:

- TblSetupPaymentMethod: 6
- TblSetupServicesMethod: 1
- TblLocalOutbox: 21
- TblEmployeePermission: 3

These are v001/baseline setup artifacts, not operational business rows requiring a block.

## 11. Safe / Unsafe Repair Decision

Safe repair gate result: safe for operator retest.

The implementation requires:

- target database exactly `obm_pos_dev_v0_pg`
- Environment exactly Development
- V008 rollback anchor artifacts present
- current durable Phase 1 identity drives materialization
- v002 marker absent before first run
- exactly one runtime profile row
- RuntimeState Activated or Installing
- no conflicting operational business rows
- no production/reference/protected database target

If the gate fails, execution stops with:

`BLOCKED_PHASE2_IDENTITY_SPINE_REPAIR_UNSAFE`

## 12. Full Identity-Spine Equality Proof

Prompt037 added explicit identity spine verification:

Phase1Identity -> TblTenant -> TblPosLocal -> TblPosRuntimeProfile -> marker context

Verified fields:

- TenantGuid
- TenantCode
- TenantName
- PosGuid
- PosName
- slot
- runtime TenantCode
- runtime PosSlotNumber
- runtime device/install representation
- runtime SourceClientId
- runtime DatabaseName
- runtime EnvironmentName
- marker TenantGuid/PosGuid in-memory context

Any mismatch after permitted reconciliation fails closed with:

`PHASE2_IDENTITY_SPINE_MISMATCH`

## 13. Runtime Profile Update Mapping / History Policy

Runtime profile mapping:

- DeviceGuid = InstallationAttemptGuid under the current local representation
- InstallationGuid = LocalInstallationGuid
- DeviceRegistrationId = InstallationAttemptGuid as `N` format
- SourceClientId = `POS:{PosGuid:N}`
- DatabaseName = approved local database
- EnvironmentName = Development

History policy:

- Activated -> Activated identity-only repair: history delta 0
- Installing -> Activated: append one real transition row
- Disabled / RecoveryRequired: blocked
- unknown runtime transition: blocked

## 14. Transaction / Rollback / Marker Hard-Gate Proof

The executor still uses one target `NpgsqlConnection` and one serializable transaction.

Prompt037 transaction order:

1. target guard and rollback anchor
2. advisory transaction lock
3. v001 marker verification
4. materialize/adopt TblTenant
5. materialize/adopt TblPosLocal
6. safe runtime-profile identity repair
7. identity-spine readback verification
8. permission reconciliation
9. actual permission GUID map readback
10. employee insert/adopt
11. outbox insert/adopt
12. marker hard gate
13. v002 marker insert last
14. excluded-table invariant check
15. commit

Any exception rolls back Tenant/POS materialization changes, runtime-profile rebind, permission/employee rows, outbox rows, runtime-history rows, and the v002 marker.

## 15. Permission / Employee / Outbox Behavior Preserved

Preserved from prompt035/prompt036:

- PostgreSQL quoted identifier syntax fix
- all 7 permission defaults reconciled before employees
- actual permission GUID readback before employee insert
- 20 reference-driven employee rows
- `LoginNumber` varchar(20) safeguard
- deterministic outbox
- one target transaction
- v002 marker last

## 16. Expected Physical Deltas

Expected first successful prompt037 retest:

- TblTenant delta: 0 if compatible current row exists
- TblPosLocal delta: 0 if compatible current row exists
- TblPosRuntimeProfile row-count delta: 0
- TblPosRuntimeProfile update delta: 1 for current stale identity
- TblPosRuntimeStateHistory delta: 0 for identity-only repair while already Activated
- TblEmployeePermission inserted: 4
- TblEmployeePermission adopted: 3
- TblEmployee inserted: 20
- TblLocalOutbox delta: 24 plus any proven safe permission update events if defaults differ
- v002 marker delta: 1

## 17. Same-Version Replay Policy

After first success:

- TblTenant delta: 0
- TblPosLocal delta: 0
- TblPosRuntimeProfile delta: 0
- TblPosRuntimeStateHistory delta: 0
- TblEmployeePermission delta: 0
- TblEmployee delta: 0
- TblLocalOutbox delta: 0
- v002 marker delta: 0

## 18. Source Files Changed

Prompt037 source/test/artifact files:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\PostgreSqlPhase2ReferenceSeedExecutor.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\reference-driven-v002-employees-r5\README.md`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

## 19. Build / Test Counts

Commands run:

```powershell
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Results:

- InstallationV0 build: PASS, 0 warnings, 0 errors
- NailSalonNet8 build: PASS, 176 warnings, 0 errors
- InstallationV0 focused tests: PASS, 44 passed, 0 failed, 0 skipped

## 20. Active Label Proof

Active label:

`prompt037`

Window title pattern:

`OBM InstallationV0 Phase 1/2 - prompt037`

## 21. No Reference Mutation / No Secret Leakage / No Source Push

Confirmed:

- no final physical WPF seed was run
- no target database mutation was performed by Codex
- no reference database mutation was performed
- no Pairing Code, token, password, passfile content, connection string, or raw private identifier is included
- OBM source repo was not committed or pushed
- only this coordination report is committed/pushed

## 22. Exact Operator Retest Steps

1. Start the current prompt037 WPF build.
2. Use the existing Phase 1 authorized ProductRoot/checkpoint.
3. Click `Install Local Database Baseline` once.
4. Verify no `PHASE2_RUNTIME_PROFILE_IDENTITY_MISMATCH`.
5. Verify TblTenant/TblPosLocal identity adoption.
6. Verify TblPosRuntimeProfile preserves RuntimeProfileGuid and aligns to Phase 1 identity.
7. Verify RuntimeState remains Activated and runtime-history delta is 0.
8. Verify permissions become 7 total.
9. Verify employees become 20 total.
10. Verify outbox delta is 24 unless safe permission updates are reported.
11. Verify v002 marker is written last.
12. Re-run same version and verify zero deltas.

## 23. Coordination Commit SHA

Final pushed commit SHA is reported by Codex after commit/push. Embedding the final SHA inside this file would change the commit hash.

