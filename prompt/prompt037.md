# Prompt 037 — Reconcile Phase 1 identity with TblPosRuntimeProfile before v002 seed

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

The v002 permission/employee transaction was blocked before commit.

Do not manually edit `TblPosRuntimeProfile`, `TblPosRuntimeStateHistory`, `TblTenant`, or `TblPosLocal` in pgAdmin. The correction must be part of the InstallationV0 path so a clean database can reproduce it.

## Current known state

Approved target:

```text
obm_pos_dev_v0_pg
Environment = Development
```

Known pre-v002 state from prior reports:

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

Prompt036 already fixed SQL syntax. Preserve all prior corrections:

- full 7-row permission reconciliation;
- permission parents before employees;
- actual target permission GUID readback;
- 20 employee rows;
- `LoginNumber` varchar(20) safeguard;
- one target transaction;
- deterministic outbox;
- runtime-state integration;
- v002 marker last.

## First requirement — prove rollback/no partial commit

Inspect the target read-only as role `hung` and record sanitized counts/state:

```text
TblEmployeePermission
TblEmployee
TblLocalOutbox
v001 marker
v002 marker
TblPosRuntimeProfile
TblPosRuntimeStateHistory
```

If any v002 permission, employee, outbox, runtime transition, or marker row committed, stop with:

```text
BLOCKED_PHASE2_V002_PARTIAL_COMMIT_DETECTED
```

Do not clean up manually.

## Exact identity audit

Compare these identity sources field by field:

### A. Current Phase 1 authorized identity

Read only from the existing Phase 1 checkpoint / protected bootstrap identity path:

```text
TenantGuid
TenantCode
PosGuid
PosName
PosSlot
InstallationGuid / InstallationAttemptGuid as represented
PosDeviceGuid / device registration identity as represented
LocalInstallationGuid
SourceClientId
EnvironmentName
DatabaseName
```

Do not print tokens, Pairing Codes, secrets, or protected credential material.

### B. TblPosRuntimeProfile current row

Read safely:

```text
RuntimeProfileGuid / singleton key
TenantGuid
TenantCode
PosGuid
PosDeviceGuid
InstallationGuid
DeviceRegistrationId
SourceClientId
DatabaseName
EnvironmentName
ProfileVersion
SchemaVersion
BaselineVersion
AppVersion
RuntimeState
RecoveryReasonCode
```

### C. Current target identity rows

Inspect `TblTenant`, `TblPosLocal`, current v001 marker identity, and any canonical local POS identity/config files.

Report only sanitized identity classifications and mismatch field names; do not publish raw private identifiers unnecessarily.

## Mismatch classification

Classify the physical mismatch into exactly one category:

```text
A. Formatting/nullable normalization mismatch only
B. Runtime profile points to an older Development tenant/POS identity while current Phase 1 identity rows also exist
C. Runtime profile identity is truly conflicting with current Phase 1 identity and target business/runtime data
D. Phase 1 checkpoint identity is inconsistent with the v001 marker or target database identity
```

Do not assume category B without proving it.

## Target identity inventory and legacy-state audit

The target has previously shown more than one `TblTenant` / `TblPosLocal` row in some evidence. Audit exact current counts and relationships:

```text
TblTenant identities
TblPosLocal identities
TblPosRuntimeProfile identity
v001 marker identity
Phase 1 identity
```

For each extra/legacy identity, determine whether any dependent business/runtime data exists:

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

Do not delete legacy rows in prompt037. The purpose is to decide whether a controlled runtime-profile rebind is safe.

## Safe controlled rebind gate

A runtime-profile identity rebind is allowed only when all of the following are proven:

```text
target database exactly obm_pos_dev_v0_pg
Environment exactly Development
V008 pre-v002 backup anchor valid
v001 marker belongs to the current Phase 1 tenant/POS identity
v002 marker absent
exactly one TblPosRuntimeProfile row exists
current profile state is Activated or Installing
current Phase 1 TblTenant/TblPosLocal parents exist and are compatible
no operational/business rows are tied to the legacy runtime-profile identity
no production/reference/protected database is involved
```

If any gate fails, stop with:

```text
BLOCKED_PHASE2_RUNTIME_PROFILE_REBIND_UNSAFE
```

Do not overwrite identity blindly.

## Controlled rebind behavior

When the safe gate passes, preserve the existing runtime profile row identity/key and update only the canonical identity/configuration fields that are meant to follow the active installation:

```text
TenantGuid
TenantCode
PosGuid
PosDeviceGuid
InstallationGuid / device registration fields as defined by the current model
SourceClientId
DatabaseName
EnvironmentName
ProfileVersion / BaselineVersion / AppVersion only if canonical source requires update
RecoveryReasonCode cleared only if model semantics allow
UpdatedAt / equivalent timestamp
```

Do not guess field mappings. Use existing `PosRuntimeProfileRecord`, repository, startup assessment, and provisioning code as the contract.

Preserve:

```text
RuntimeProfileGuid / singleton identity
current RuntimeState unless a real canonical state transition is required
created timestamp/history identity
unrelated rollout/update fields
```

The target row after rebind must match current Phase 1 Tenant/POS/device/installation identity exactly according to the model.

## Runtime history policy

`TblPosRuntimeStateHistory` is append-only state-transition history.

Default rule:

```text
identity-only rebind while RuntimeState remains Activated
→ do not append a duplicate/same-state history row
```

Append history only if the authoritative runtime-state contract requires a real transition such as `Installing -> Activated`.

Do not manufacture a fake transition merely to create an audit row.

If the codebase has a separate identity-change audit path, use it; otherwise report the rebind through the v002 transaction result and marker evidence.

Expected current history delta after a safe identity-only rebind:

```text
0
```

## Transaction order

Use one target `NpgsqlConnection` and one serializable transaction:

```text
BEGIN
  verify target and V008 backup anchor
  acquire advisory transaction lock
  verify v001 marker/current Phase 1 identity
  audit current runtime profile identity
  safely rebind TblPosRuntimeProfile when allowed
  read back and verify runtime profile identity

  reconcile all 7 TblEmployeePermission defaults
  read back actual permission GUID map
  insert/adopt 20 TblEmployee rows
  insert permission/employee TblLocalOutbox rows

  verify RuntimeState = Activated
  verify runtime history policy
  verify excluded business/runtime tables unchanged
  write v002 marker last
  read back all invariants
COMMIT
```

Any error must rollback:

```text
runtime profile rebind
permission rows/outbox
employee rows/outbox
runtime history changes
v002 marker
```

Preserve v001 data/marker and Phase 1 checkpoint.

## Existing/legacy TblTenant and TblPosLocal rows

Do not delete old tenant/POS rows in prompt037.

After the runtime profile is aligned, report whether extra legacy rows remain and whether they should be handled in a later cleanup/versioned migration.

The current action should ensure all startup/source-of-truth paths select the current Phase 1 identity through `TblPosRuntimeProfile` and current marker/configuration.

## Expected physical delta if safe rebind is needed

Expected from the current known target, subject to verified facts:

```text
TblPosRuntimeProfile row count delta = 0
TblPosRuntimeProfile updated rows = 1
TblPosRuntimeStateHistory delta = 0
TblEmployeePermission inserted = 4
TblEmployeePermission adopted = 3
TblEmployee inserted = 20
TblLocalOutbox delta = 24 plus any proven permission updates
v002 marker delta = 1
```

If the runtime profile already matches after normalization, profile update delta should be 0.

## Same-version replay

After first successful v002 execution, replay must produce:

```text
TblPosRuntimeProfile delta = 0
TblPosRuntimeStateHistory delta = 0
TblEmployeePermission delta = 0
TblEmployee delta = 0
TblLocalOutbox delta = 0
v002 marker delta = 0
```

## UI behavior

Keep explicit action:

```text
Install Local Database Baseline
```

Do not auto-run on startup.

After success show safe proof:

```text
Runtime profile identity aligned
RuntimeState Activated
Permission parents reconciled
Employees inserted/adopted
Outbox delta
Runtime history delta
Marker last
Transaction committed
```

On mismatch failure show safe field names/categories, not raw secrets or private IDs.

## WPF label

Because source changes are expected, set:

```text
Build label: prompt037
Window title: OBM InstallationV0 Phase 1/2 - prompt037
```

If investigation proves no source correction is needed, keep `prompt036` and explain why. Do not change label for report-only work.

## Required tests

Add focused tests for:

```text
Phase 1 identity versus runtime profile field comparison
normalization-only match
legacy Development identity mismatch classification
safe rebind gate
unsafe business-data gate
preserve RuntimeProfileGuid
update only canonical identity fields
no fake runtime-history transition
readback identity verification before permission/employee insert
rollback restores old runtime profile on later failure
permission/employee behavior from prompt036 preserved
marker last
same-version zero delta
prompt037 label when source changes
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Do not run the final physical WPF seed automatically. Leave operator retest after source/build/test completion.

## Report 037

Create and push:

```text
report/report037.md
```

Required sections:

1. Verdict.
2. Physical runtime-profile identity mismatch evidence.
3. Post-failure rollback proof.
4. Phase 1 identity field inventory, sanitized.
5. Runtime profile identity field inventory, sanitized.
6. Exact mismatched field names and classification.
7. TblTenant/TblPosLocal/marker identity inventory.
8. Legacy identity dependent-data audit.
9. Safe/unsafe rebind decision.
10. Runtime profile field update mapping.
11. Runtime history policy/delta.
12. Transaction and rollback proof.
13. Permission/employee/outbox behavior preserved.
14. Expected physical deltas.
15. Same-version replay policy.
16. Remaining legacy identity cleanup recommendation.
17. Source files changed.
18. Build/test counts.
19. Active label proof.
20. No reference mutation/no secret leakage/no source push.
21. Exact operator retest steps.
22. Coordination commit SHA.

## Valid verdicts

```text
PHASE2_V002_RUNTIME_PROFILE_IDENTITY_REBIND_READY_FOR_USER_RETEST
```

```text
PHASE2_V002_RUNTIME_PROFILE_NORMALIZATION_FIX_READY_FOR_USER_RETEST
```

```text
BLOCKED_PHASE2_RUNTIME_PROFILE_REBIND_UNSAFE
```

```text
BLOCKED_PHASE2_RUNTIME_PROFILE_IDENTITY_FIX
```
