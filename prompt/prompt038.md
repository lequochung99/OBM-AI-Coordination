# Prompt 038 — Fix `TblPosRuntimeProfile.SourceClientId` constraint mapping without weakening the identity spine

## Physical evidence

Read completely before changing source:

```text
report/report036.md
report/report037.md
prompt/prompt037.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The operator physically retested build `prompt037` and received:

```text
SQLSTATE=23514
Schema=dbo
Table=TblPosRuntimeProfile
Constraint=CK_TblPosRuntimeProfile_SourceClientId
```

The v002 transaction was blocked during runtime-profile identity repair.

Do not alter or drop the check constraint. Do not manually update the runtime profile in pgAdmin. The correction belongs in InstallationV0 so a clean installation reproduces the same canonical value.

## Preserve the identity spine

The authoritative invariant remains:

```text
Phase 1 checkpoint/bootstrap identity
=
TblTenant
=
TblPosLocal
=
TblPosRuntimeProfile
=
marker context
```

Phase 2 must not invent a new POS identity merely to satisfy the constraint.

The only permitted correction is to encode the Phase 1 `PosGuid` into `SourceClientId` using the canonical format required by the existing schema and runtime contract.

## First requirement — prove rollback/no partial commit

Inspect `obm_pos_dev_v0_pg` read-only as role `hung` and record sanitized counts/state:

```text
TblTenant
TblPosLocal
TblPosRuntimeProfile
TblPosRuntimeStateHistory
TblEmployeePermission
TblEmployee
TblLocalOutbox
v001 marker
v002 marker
```

Expected if rollback succeeded:

```text
TblPosRuntimeProfile still has the pre-prompt037 identity
TblEmployeePermission = 3
TblEmployee = 0
TblLocalOutbox = 21
v001 marker = 1
v002 marker = 0
TblPosRuntimeStateHistory = 1
```

If any prompt037 runtime-profile rebind, permission, employee, outbox, history, or marker mutation committed, stop with:

```text
BLOCKED_PHASE2_V002_PARTIAL_COMMIT_DETECTED
```

Do not repair manually.

## Exact constraint audit

Read PostgreSQL metadata for:

```text
dbo.TblPosRuntimeProfile
CK_TblPosRuntimeProfile_SourceClientId
```

Report safely:

```text
constraint expression
column type/max length
nullable/non-null
accepted prefix/case/UUID representation
```

Also inspect the corresponding EF/entity configuration, migrations, repository writers, startup readers, SignalR/subscriber identity code, outbox/source-client conventions, and tests.

Do not assume the required format only from the constraint name.

## Likely mismatch to verify

Report037 currently documents this mapping:

```text
SourceClientId = POS:{PosGuid:N}
```

Prior canonical runtime evidence used a hyphenated UUID shape similar to:

```text
POS:{PosGuid:D}
```

This strongly suggests the physical failure may be an `N` versus `D` UUID-format mismatch, but prompt038 must prove the exact contract before changing source.

## Canonical SourceClientId rule

After audit, create one shared canonical formatter/helper used by all InstallationV0 Phase 2 runtime-profile writes and identity comparisons.

The helper must:

```text
accept the authoritative Phase 1 PosGuid
produce the exact schema-valid SourceClientId
be deterministic
be culture-invariant
use canonical casing required by current code/schema
never accept an unrelated externally supplied POS identity
```

Do not duplicate string formatting in multiple SQL/repository/UI paths.

Examples are illustrative only; use the audited contract:

```csharp
FormatPosSourceClientId(PosGuid)
```

## Cross-layer consistency audit

Verify the same canonical value is expected by:

```text
TblPosRuntimeProfile.SourceClientId
TblPosLocal or local POS identity representation
Phase 1 checkpoint/bootstrap context where represented
SignalR registration/subscriber identity
TblLocalOutbox.SourceClientId
runtime startup assessment
update/rollout/runtime control code
```

Classify each usage:

```text
must equal canonical POS source client ID
stores a different identity by design
legacy/stale formatter that must be corrected
```

Do not silently change unrelated source-client formats used by non-POS clients.

## Runtime profile repair behavior

Keep prompt037 behavior:

- preserve `RuntimeProfileGuid`;
- materialize Tenant/POS identity from Phase 1;
- preserve `RuntimeState=Activated` when no real state transition occurs;
- runtime-history delta remains 0 for identity-only repair;
- read back and verify the complete identity spine before permissions/employees;
- write v002 marker last.

Correct only the canonical `SourceClientId` mapping and any directly related equality/validation code proven stale.

After repair, validate:

```text
runtime profile SourceClientId satisfies CK_TblPosRuntimeProfile_SourceClientId
runtime profile SourceClientId corresponds exactly to Phase 1 PosGuid
runtime profile SourceClientId equals canonical outbox/source identity where contract requires
```

## One transaction

Use one target connection and one serializable transaction:

```text
BEGIN
  verify target and V008 rollback anchor
  advisory lock
  verify v001 marker
  materialize/adopt TblTenant and TblPosLocal from Phase 1
  repair TblPosRuntimeProfile with canonical schema-valid SourceClientId
  read back/check constraint and full identity spine
  reconcile 7 TblEmployeePermission rows
  read back actual permission GUID map
  insert/adopt 20 TblEmployee rows
  insert required outbox rows using canonical SourceClientId
  verify runtime state/history and excluded tables
  marker hard gate
  write v002 marker last
COMMIT
```

Any error rolls back all runtime-profile, permission, employee, outbox, history, and marker work.

## Outbox consistency

Audit existing 21 baseline outbox rows only read-only; do not rewrite historical rows in prompt038 unless a proven contract violation makes current v002 execution unsafe.

New permission/employee outbox rows must use the canonical Phase 1 POS `SourceClientId` format.

If historical v001 outbox rows use a different but previously valid format, report it and defer a versioned migration decision. Do not mutate history silently.

## Same-version replay

After first successful run:

```text
TblTenant delta = 0
TblPosLocal delta = 0
TblPosRuntimeProfile delta = 0
TblPosRuntimeStateHistory delta = 0
TblEmployeePermission delta = 0
TblEmployee delta = 0
TblLocalOutbox delta = 0
v002 marker delta = 0
```

## WPF label

Because source changes are expected, set:

```text
Build label: prompt038
Window title: OBM InstallationV0 Phase 1/2 - prompt038
```

If audit proves the source already uses the correct formatter and the failure comes from another mapped field, keep the label rule tied to actual source changes and report the exact finding.

## Required tests

Add focused tests proving:

```text
physical check-constraint expression is represented by contract tests
canonical SourceClientId formatter accepts Phase 1 PosGuid
canonical output satisfies the check constraint
N-vs-D or other proven bad format is rejected
runtime-profile update uses the shared formatter
identity-spine comparison uses the same canonical formatter
new TblLocalOutbox rows use the same canonical value
no unrelated client SourceClientId formats are changed
runtime history remains zero for Activated identity-only repair
permission/employee behavior from prompt037 remains
marker last
rollback on later failure restores old runtime profile
same-version replay zero delta
prompt038 label
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Do not run the final physical WPF seed automatically. Leave it for the operator.

## Report 038

Create and push:

```text
report/report038.md
```

Required sections:

1. Verdict.
2. Physical 23514 evidence.
3. Post-failure rollback proof.
4. Exact check-constraint expression and column contract.
5. Exact root cause and rejected value shape classification.
6. Canonical SourceClientId formatter design.
7. Cross-layer SourceClientId usage audit.
8. Runtime-profile update/equality corrections.
9. Outbox SourceClientId policy.
10. Identity-spine proof preserved.
11. Transaction/rollback/marker-last proof.
12. Expected physical deltas.
13. Same-version replay policy.
14. Source files changed.
15. Build/test counts.
16. Prompt038 label proof.
17. No reference mutation/no secret leakage/no source push.
18. Exact operator retest steps.
19. Coordination commit SHA.

## Valid verdicts

```text
PHASE2_V002_SOURCECLIENTID_CONSTRAINT_FIX_READY_FOR_USER_RETEST
```

```text
BLOCKED_PHASE2_V002_SOURCECLIENTID_CONTRACT_CONFLICT
```

```text
BLOCKED_PHASE2_V002_SOURCECLIENTID_FIX
```
