# Prompt 037 — Lock the Phase 1 → Phase 2 identity spine and repair runtime-profile mismatch

## Operator-confirmed canonical identity spine

The following rule is authoritative and must be implemented exactly:

```text
Durable Phase 1 checkpoint / protected bootstrap identity
        ↓ materialize, never invent
TblTenant
        ↓
TblPosLocal
        ↓
TblPosRuntimeProfile
        ↓
Phase 2 completion marker context
```

Phase 2 must **never invent, substitute, infer, or independently derive** a Tenant/POS installation identity.

The durable Phase 1 proof is the only authority for:

```text
TenantGuid
TenantCode
TenantName
PosGuid
PosStationId
PosName
PosSlot
InstallationAttemptGuid / InstallationGuid as represented by the contract
PosDeviceGuid / device identity as represented by the contract
LocalInstallationGuid
SourceClientId
EnvironmentName
DatabaseName
```

The required invariant before a Phase 2 marker can be written is:

```text
Phase 1 checkpoint/bootstrap Tenant/POS identity
=
TblTenant current local identity
=
TblPosLocal current local POS identity
=
TblPosRuntimeProfile current runtime identity
=
Phase 2 marker Tenant/POS/installation context
```

If any identity differs, Phase 2 must fail closed and must not write a pass marker.

## Physical evidence

Read completely before changing source:

```text
report/report031.md
report/report035.md
report/report036.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The operator physically retested build `prompt036` and received:

```text
PHASE2_RUNTIME_PROFILE_IDENTITY_MISMATCH
```

Do not manually edit `TblTenant`, `TblPosLocal`, `TblPosRuntimeProfile`, `TblPosRuntimeStateHistory`, or marker rows in pgAdmin.

The correction must be part of InstallationV0 so a clean database can reproduce it.

## Current known target state

Approved target:

```text
obm_pos_dev_v0_pg
Environment = Development
```

Known pre-v002 state:

```text
TblEmployeePermission = 3
TblEmployee = 0
TblLocalOutbox = 21
v001 marker = 1
v002 marker = 0
TblPosRuntimeProfile = 1
RuntimeState = Activated
TblPosRuntimeStateHistory = 1
```

Preserve all prior corrections:

- PostgreSQL syntax fix from prompt036;
- all 7 permission defaults reconciled before employees;
- actual target permission GUID readback;
- 20 reference-driven employee rows;
- `LoginNumber` varchar(20) safeguard;
- one target transaction;
- deterministic outbox;
- runtime-state integration;
- v002 marker last.

## First requirement — prove rollback/no partial commit

Inspect the target read-only as role `hung` and record sanitized counts/state:

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

If any v002 permission, employee, outbox, runtime transition, runtime-profile rebind, or v002 marker committed from the failed attempt, stop with:

```text
BLOCKED_PHASE2_V002_PARTIAL_COMMIT_DETECTED
```

Do not clean up manually.

## Phase 1 durable identity proof

Read Phase 1 identity only through the existing authorized checkpoint/protected bootstrap path.

Required fields:

```text
TenantGuid
TenantCode
TenantName
PosGuid
PosStationId
PosName
PosSlot
InstallationAttemptGuid / InstallationGuid
PosDeviceGuid / device registration identity
LocalInstallationGuid
SourceClientId
EnvironmentName
DatabaseName
```

Revalidate with the protected API contract when the credential is still valid:

```text
GET protected hello
GET /bootstrap/me
```

Do not redeem another Pairing Code automatically.

Do not print tokens, Pairing Codes, protected credential content, or private identifiers unnecessarily.

## Materialization contract

### TblTenant

`TblTenant` must contain exactly one current compatible row for the Phase 1 tenant identity used by this installation.

Required equality:

```text
TblTenant.TenantGuid = Phase1.TenantGuid
TblTenant.TenantCode = Phase1.TenantCode
TblTenant tenant display/name field = Phase1.TenantName
```

Behavior:

```text
missing current row
→ insert from Phase 1 identity

compatible current row
→ adopt/verify

same stable tenant key with incompatible identity
→ fail PHASE2_TENANT_IDENTITY_CONFLICT
```

Do not delete unrelated legacy tenant rows in prompt037.

### TblPosLocal

`TblPosLocal` must materialize the authorized Phase 1 POS identity.

Required equality, according to the physical schema/model:

```text
TblPosLocal.TenantGuid = Phase1.TenantGuid
TblPosLocal.PosGuid = Phase1.PosGuid
TblPosLocal.PosStationId = Phase1.PosStationId
TblPosLocal.PosName = Phase1.PosName
TblPosLocal slot = Phase1.PosSlot
```

Behavior:

```text
missing current POS row
→ insert from Phase 1 identity

compatible current POS row
→ adopt/verify

same station/slot with incompatible PosGuid/TenantGuid
→ fail PHASE2_POS_IDENTITY_CONFLICT
```

Do not invent a new POS GUID or silently choose a different slot.

### TblPosRuntimeProfile

`TblPosRuntimeProfile` is the current runtime-state source of truth.

It must point to the same Phase 1/TblTenant/TblPosLocal identity:

```text
TenantGuid
TenantCode
PosGuid
PosDeviceGuid
InstallationGuid / DeviceRegistrationId as represented
SourceClientId
DatabaseName
EnvironmentName
```

Canonical post-install state:

```text
RuntimeState = Activated
```

`InstalledHealthy` remains a startup assessment result, not a runtime-state enum value.

Preserve the existing `RuntimeProfileGuid`/singleton identity when performing a safe rebind.

### Phase 2 marker

The v002 marker remains:

```text
phase2-reference-driven-trial-v002-employees
```

The marker must carry or be verifiably associated with the same:

```text
TenantGuid
PosGuid
Installation/LocalInstallation context as supported by the marker schema
```

The marker may be inserted only after all identity-spine equality checks pass and all seed/runtime invariants pass.

## Identity-spine cross-check

Create one explicit in-memory comparison result before seed mutation:

```text
Phase1Identity
LocalTenantIdentity
LocalPosIdentity
RuntimeProfileIdentity
ExistingMarkerIdentity
```

Compare field by field and report only safe field names/categories.

Required pre-marker check:

```text
Phase1.TenantGuid == TblTenant.TenantGuid
Phase1.TenantCode == TblTenant.TenantCode
Phase1.TenantName == TblTenant tenant name
Phase1.PosGuid == TblPosLocal.PosGuid
Phase1.PosStationId == TblPosLocal.PosStationId
Phase1.PosName == TblPosLocal.PosName
Phase1.PosSlot == TblPosLocal slot
Phase1 tenant/POS/device/install identity == TblPosRuntimeProfile identity
Phase1 tenant/POS identity == marker context
```

If any check fails after permitted reconciliation:

```text
PHASE2_IDENTITY_SPINE_MISMATCH
```

No v002 marker may be written.

## Mismatch classification

Classify the current physical mismatch into one exact category:

```text
A. Formatting/nullable normalization only
B. Missing Phase 1 TblTenant/TblPosLocal materialization
C. Runtime profile points to an older Development identity while correct Phase 1 local parent rows exist
D. TblTenant/TblPosLocal themselves conflict with Phase 1
E. Existing v001 marker context conflicts with current Phase 1
F. Business/runtime data makes rebind unsafe
```

Do not assume category C without proof.

## Legacy identity/dependent-data audit

Audit extra/legacy identities and whether dependent rows exist in:

```text
TblInvoice*
TblOutputInfo*
TblOutputInfoTam*
terminal/payment transaction tables
booking/appointment tables
queue/turn state/history
payroll history
customer/gift-card data
employee/service data
TblLocalOutbox ownership
```

Do not delete legacy rows in prompt037.

## Safe repair gate

Identity materialization/rebind is allowed only when all are proven:

```text
target database exactly obm_pos_dev_v0_pg
Environment exactly Development
V008 pre-v002 backup anchor valid
current durable Phase 1 identity revalidated
v001 marker belongs to or can be safely reconciled to the current Phase 1 identity
v002 marker absent
exactly one current TblPosRuntimeProfile row
RuntimeState is Activated or Installing
no operational/business rows are tied to a conflicting legacy identity
no production/reference/protected database involved
```

If unsafe:

```text
BLOCKED_PHASE2_IDENTITY_SPINE_REPAIR_UNSAFE
```

## Controlled repair behavior

Within the same transaction:

1. Materialize/adopt `TblTenant` from Phase 1.
2. Materialize/adopt `TblPosLocal` from Phase 1.
3. Preserve `RuntimeProfileGuid` and safely align `TblPosRuntimeProfile` to Phase 1/local parent identity.
4. Read back all three and verify the identity spine.
5. Only after identity equality succeeds, continue permission/employee seed.

Do not blindly overwrite identity.

Do not change `RuntimeState` when it is already `Activated`.

If `Installing -> Activated` is a real allowed transition, append one runtime-history row. An identity-only rebind with state unchanged must produce:

```text
TblPosRuntimeStateHistory delta = 0
```

## One atomic transaction

Use one target `NpgsqlConnection` and one serializable transaction:

```text
BEGIN
  verify target and V008 backup anchor
  acquire advisory transaction lock
  revalidate durable Phase 1 identity

  materialize/adopt TblTenant from Phase 1
  materialize/adopt TblPosLocal from Phase 1
  safely align TblPosRuntimeProfile
  read back and verify the full identity spine

  reconcile all 7 TblEmployeePermission defaults
  read back actual PermissionName -> EmployeePermissionGuid map
  insert/adopt 20 TblEmployee rows
  insert permission/employee TblLocalOutbox rows

  verify RuntimeState = Activated
  verify runtime-history policy
  verify excluded business/runtime tables unchanged

  verify marker context equals Phase 1/Tenant/POS/runtime identity
  write v002 marker last
  read back all invariants
COMMIT
```

Any error must rollback:

```text
TblTenant/TblPosLocal materialization changes
runtime-profile identity repair
permission rows/outbox
employee rows/outbox
runtime-history changes
v002 marker
```

Preserve the durable Phase 1 checkpoint and v001 completed data.

## Marker hard gate

Immediately before marker insert, execute an explicit identity-spine assertion.

Marker insert is forbidden unless:

```text
IdentitySpineVerified = true
RuntimeState = Activated
Permission parents complete = true
Employee FK validation = true
Outbox invariants = true
Excluded runtime/business deltas = 0
```

Tests must prove marker SQL is unreachable when identity equality fails.

## Expected physical delta

Subject to verified facts:

```text
TblTenant/TblPosLocal data delta = 0 when compatible rows already exist
TblPosRuntimeProfile row-count delta = 0
TblPosRuntimeProfile update delta = 0 or 1
TblPosRuntimeStateHistory delta = 0 for identity-only repair
TblEmployeePermission inserted = 4
TblEmployeePermission adopted = 3
TblEmployee inserted = 20
TblLocalOutbox delta = 24 plus proven safe permission updates
v002 marker delta = 1
```

## Same-version replay

After first success:

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

## UI behavior

Keep explicit operator action:

```text
Install Local Database Baseline
```

Do not auto-run.

After success display safe proof:

```text
Phase 1 identity revalidated
TblTenant identity verified
TblPosLocal identity verified
TblPosRuntimeProfile identity verified
Identity spine verified
RuntimeState Activated
Permission parents reconciled
Employees inserted/adopted
Outbox delta
Runtime history delta
Marker context verified
Marker last
Transaction committed
```

On failure show safe mismatched field names/categories, never secrets/raw identifiers.

## WPF label

Because source changes are expected:

```text
Build label: prompt037
Window title: OBM InstallationV0 Phase 1/2 - prompt037
```

If no source correction is needed, keep `prompt036` and explain why.

## Required tests

Add focused tests for:

```text
Phase 1 is the only identity authority
TblTenant materialization from Phase 1
TblPosLocal materialization from Phase 1
no independently invented TenantGuid/PosGuid
runtime profile equality with local parent rows
v001/marker context equality
normalization-only identity match
safe identity repair gate
unsafe business-data gate
preserve RuntimeProfileGuid
no fake runtime-history transition
identity readback before permission/employee insert
marker unreachable on identity mismatch
rollback restores old identity rows/profile on later failure
all prompt036 permission/employee behavior preserved
same-version zero delta
prompt037 label
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Do not run the final physical WPF seed automatically.

## Report 037

Create and push:

```text
report/report037.md
```

Required sections:

1. Verdict.
2. Physical runtime-profile identity mismatch evidence.
3. Post-failure rollback proof.
4. Durable Phase 1 identity inventory, sanitized.
5. TblTenant identity inventory and materialization decision.
6. TblPosLocal identity inventory and materialization decision.
7. TblPosRuntimeProfile identity inventory.
8. Existing marker identity context.
9. Exact mismatched field names/classification.
10. Legacy dependent-data audit.
11. Safe/unsafe repair decision.
12. Full identity-spine equality proof.
13. Runtime profile update mapping/history policy.
14. Transaction/rollback/marker hard-gate proof.
15. Permission/employee/outbox behavior preserved.
16. Expected physical deltas.
17. Same-version replay policy.
18. Source files changed.
19. Build/test counts.
20. Active label proof.
21. No reference mutation/no secret leakage/no source push.
22. Exact operator retest steps.
23. Coordination commit SHA.

## Valid verdicts

```text
PHASE2_V002_IDENTITY_SPINE_REPAIR_READY_FOR_USER_RETEST
```

```text
PHASE2_V002_IDENTITY_SPINE_ALREADY_VALID_READY_FOR_USER_RETEST
```

```text
BLOCKED_PHASE2_IDENTITY_SPINE_REPAIR_UNSAFE
```

```text
BLOCKED_PHASE2_IDENTITY_SPINE_FIX
```
