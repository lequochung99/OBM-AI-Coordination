# Report 038 - SourceClientId Constraint Mapping Fix

## 1. Verdict

PHASE2_V002_SOURCECLIENTID_CONSTRAINT_FIX_READY_FOR_USER_RETEST

Prompt038 source correction is implemented and verified by build/tests. The final physical WPF seed was not run.

## 2. Physical 23514 Evidence

The operator physically retested prompt037 and received:

```text
SQLSTATE=23514
Schema=dbo
Table=TblPosRuntimeProfile
Constraint=CK_TblPosRuntimeProfile_SourceClientId
```

This blocked the runtime-profile identity repair inside the v002 transaction.

## 3. Post-Failure Rollback Proof

Read-only target verification used `BEGIN TRANSACTION READ ONLY` and `ROLLBACK`.

Current state:

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

No prompt037 or prompt038 partial runtime-profile rebind, permission, employee, outbox, history, or v002 marker mutation is present.

## 4. Exact Check-Constraint Expression And Column Contract

Physical metadata:

```text
CK_TblPosRuntimeProfile_SourceClientId
CHECK ("SourceClientId"::text = ('POS:'::text || "PosGuid"::text))
```

Column contract:

- table: dbo.TblPosRuntimeProfile
- column: SourceClientId
- type: character varying
- max length: 80
- nullable: NO

Accepted shape:

- prefix: `POS:`
- UUID representation: PostgreSQL `uuid::text`
- format: lowercase hyphenated Guid `D`
- example shape: `POS:xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`

## 5. Exact Root Cause And Rejected Value Shape

Prompt037 mapped:

```text
SourceClientId = POS:{PosGuid:N}
```

The constraint requires:

```text
SourceClientId = POS:{PosGuid:D}
```

Root cause:

- `N` format removes hyphens.
- PostgreSQL `uuid::text` emits hyphenated lowercase text.
- Therefore `POS:{PosGuid:N}` violates `CK_TblPosRuntimeProfile_SourceClientId`.

Rejected shape classification:

`N-vs-D UUID format mismatch`

## 6. Canonical SourceClientId Formatter Design

Added shared InstallationV0 Phase2 helper:

`Phase2SourceClientIdFormatter.FormatPosSourceClientId(Guid posGuid)`

Rules:

- accepts only the authoritative Phase 1 `PosGuid`
- rejects `Guid.Empty`
- returns `POS:` plus `posGuid.ToString("D", CultureInfo.InvariantCulture)`
- deterministic
- culture-invariant
- does not accept an externally supplied POS identity string

Validation helper:

`Phase2SourceClientIdFormatter.SatisfiesRuntimeProfileConstraint(Guid posGuid, string? sourceClientId)`

## 7. Cross-Layer SourceClientId Usage Audit

Audited usages:

- TblPosRuntimeProfile.SourceClientId: must equal canonical POS SourceClientId.
- TblLocalOutbox.SourceClientId for new v002 rows: must equal canonical POS SourceClientId.
- InstallationV0 Phase2 trial plan rows: corrected to canonical formatter.
- Runtime profile policy: already uses `POS:{posGuid:D}`.
- runtime startup/recovery/control checks: expect runtime profile SourceClientId to match POS Guid.
- Station identity builder: uses default Guid string interpolation, which is hyphenated `D` by design.
- SignalR/subscriber and CompanyInfo consumers: consume the current runtime/station SourceClientId by design.
- non-POS clients such as API tests, customer app, booking console, and tools: different identity by design and not changed.

Existing historical target data:

- existing 21 TblLocalOutbox rows use `POS_N` shape.
- current TblPosRuntimeProfile row uses `POS_D_LOWER` shape.

Historical v001 outbox rows are reported and deferred. They were not rewritten in prompt038.

## 8. Runtime-Profile Update / Equality Corrections

Prompt038 changed InstallationV0 Phase2 runtime-profile repair/equality to use:

`Phase2SourceClientIdFormatter.FormatPosSourceClientId(identity.PosGuid)`

This applies to:

- runtime-profile `UPDATE dbo."TblPosRuntimeProfile" SET "SourceClientId" = @sourceClientId`
- runtime-profile identity comparison
- new v002 TblLocalOutbox rows
- Phase2 trial-plan outbox rows

The identity spine from prompt037 remains intact.

## 9. Outbox SourceClientId Policy

New v002 permission/employee outbox rows now use canonical `POS:{PosGuid:D}`.

Existing v001 outbox rows remain unchanged:

- count: 21
- shape: `POS_N`

No historical outbox migration was performed because prompt038 is scoped to the current v002 transaction and runtime-profile constraint fix.

## 10. Identity-Spine Proof Preserved

Preserved invariant:

```text
Phase 1 checkpoint/bootstrap identity
= TblTenant
= TblPosLocal
= TblPosRuntimeProfile
= marker context
```

Phase 2 still:

- materializes/adopts TblTenant from Phase 1
- materializes/adopts TblPosLocal from Phase 1
- preserves RuntimeProfileGuid
- repairs TblPosRuntimeProfile identity safely
- reads back and verifies identity spine before permission/employee seed
- hard-gates marker insert on identity spine equality

## 11. Transaction / Rollback / Marker-Last Proof

The executor still uses one target `NpgsqlConnection` and one serializable transaction.

Prompt038 transaction order remains:

1. target guard and rollback anchor
2. advisory lock
3. v001 marker verification
4. TblTenant adopt/materialize
5. TblPosLocal adopt/materialize
6. TblPosRuntimeProfile repair with canonical SourceClientId
7. identity-spine readback
8. permission reconciliation
9. actual permission GUID map readback
10. employee insert/adopt
11. required outbox rows with canonical SourceClientId
12. runtime/excluded table verification
13. marker hard gate
14. v002 marker last
15. commit

Any error rolls back all runtime-profile, permission, employee, outbox, history, and marker work.

## 12. Expected Physical Deltas

Expected first successful prompt038 retest:

- TblTenant delta: 0 if compatible row already exists
- TblPosLocal delta: 0 if compatible row already exists
- TblPosRuntimeProfile row-count delta: 0
- TblPosRuntimeProfile update delta: 1 for current stale identity and canonical SourceClientId
- TblPosRuntimeStateHistory delta: 0 for identity-only repair while already Activated
- TblEmployeePermission inserted: 4
- TblEmployeePermission adopted: 3
- TblEmployee inserted: 20
- TblLocalOutbox delta: 24 plus any proven safe permission update events if defaults differ
- v002 marker delta: 1

## 13. Same-Version Replay Policy

After first success:

- TblTenant delta: 0
- TblPosLocal delta: 0
- TblPosRuntimeProfile delta: 0
- TblPosRuntimeStateHistory delta: 0
- TblEmployeePermission delta: 0
- TblEmployee delta: 0
- TblLocalOutbox delta: 0
- v002 marker delta: 0

## 14. Source Files Changed

Prompt038 source/test/artifact files:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2SourceClientIdFormatter.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TrialPlanBuilder.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\PostgreSqlPhase2ReferenceSeedExecutor.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\reference-driven-v002-employees-r6\README.md`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

## 15. Build / Test Counts

Commands run:

```powershell
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Results:

- InstallationV0 build: PASS, 0 warnings, 0 errors
- NailSalonNet8 build: PASS, 176 warnings, 0 errors
- InstallationV0 focused tests: PASS, 45 passed, 0 failed, 0 skipped

## 16. Prompt038 Label Proof

Active label:

`prompt038`

Window title pattern:

`OBM InstallationV0 Phase 1/2 - prompt038`

## 17. No Reference Mutation / No Secret Leakage / No Source Push

Confirmed:

- no final physical WPF seed was run
- no target database mutation was performed by Codex
- no reference database mutation was performed
- no check constraint was altered or dropped
- no Pairing Code, token, password, passfile content, connection string, or raw private identifier is included
- OBM source repo was not committed or pushed
- only this coordination report is committed/pushed

## 18. Exact Operator Retest Steps

1. Start the current prompt038 WPF build.
2. Use the existing Phase 1 authorized ProductRoot/checkpoint.
3. Click `Install Local Database Baseline` once.
4. Verify no SQLSTATE 23514 for `CK_TblPosRuntimeProfile_SourceClientId`.
5. Verify TblPosRuntimeProfile.SourceClientId satisfies `POS:` + `PosGuid::text`.
6. Verify TblPosRuntimeProfile preserves RuntimeProfileGuid.
7. Verify RuntimeState remains Activated and runtime-history delta is 0.
8. Verify permissions become 7 total.
9. Verify employees become 20 total.
10. Verify new v002 outbox rows use canonical `POS:{PosGuid:D}`.
11. Verify v002 marker is written last.
12. Re-run same version and verify zero deltas.

## 19. Coordination Commit SHA

Final pushed commit SHA is reported by Codex after commit/push. Embedding the final SHA inside this file would change the commit hash.

