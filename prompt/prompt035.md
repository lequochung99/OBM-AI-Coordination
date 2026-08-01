# Prompt 035 — Reconcile full permission defaults, then seed employees

## Authoritative current state

Read completely:

```text
report/report033.md
report/report034.md
prompt/prompt033.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Report034 proved the exact approved target state:

```text
database = obm_pos_dev_v0_pg
TblEmployeePermission = 3
labels = Admin, Owner, Sub_Manager
TblEmployee = 0
TblLocalOutbox = 21
v001 marker = 1
v002 marker = 0
TblPosRuntimeProfile = 1 / Activated
TblPosRuntimeStateHistory = 1
```

The operator screenshot showing seven permission rows was not authoritative proof for the target connection. Treat `obm_pos_dev_v0_pg` read-only evidence as authoritative.

## Operator decision

The installation seed must use the **complete canonical permission-default set** from the reference database before inserting employees.

Reference database:

```text
enailsalon_phasee1_pos1_pg
```

Expected current permission labels:

```text
Admin
AI_Assistant
Manager
Owner
Staff
Sub_Manager
VirtualAnyTechnician
```

Do not require all seven rows to already exist in the target. The purpose of the InstallationV0 transaction is to reconcile/insert them.

The target having only three permission rows is an expected pre-v002 state, not a blocker by itself.

## Correct reconciliation rule

Read all canonical `TblEmployeePermission` rows from the reference DB through:

```text
BEGIN TRANSACTION READ ONLY
SELECT-only queries
ROLLBACK
```

For every reference permission row, classify each column as:

```text
stable key
preserve canonical safe value
replace TenantGuid
preserve existing target primary GUID when adopting
create deterministic GUID when inserting missing row
derive timestamp
exclude private/environment value
```

Permission rows are default configuration, not employee-private data. Preserve all safe functional fields required by the app, including role labels, descriptions, active/visibility/configuration flags, and other permission defaults proven by the schema/code.

Do not blindly compare only `IsActive`. Audit the actual field semantics and current WPF readers first.

## Existing three rows

For target rows with matching stable key:

```text
TenantGuid + PermissionName
```

Use this behavior:

```text
same stable key and safe fields already canonical
→ adopt existing row and actual EmployeePermissionGuid

same stable key but safe default fields differ from canonical reference
→ update the existing row in place to canonical safe defaults
→ preserve its existing EmployeePermissionGuid
→ create deterministic TblLocalOutbox U event

same stable key but irreconcilable tenant/identity corruption
→ rollback PHASE2_PERMISSION_BASELINE_CONFLICT

duplicate stable key rows
→ rollback PHASE2_PERMISSION_DUPLICATE_PARENT
```

Do not classify a normal v001-to-v002 default upgrade as a conflict merely because a safe flag or description differs.

## Missing four rows

Insert every missing canonical permission row before any employee row.

For missing parents:

```text
TenantGuid <- Phase 1 TenantGuid
EmployeePermissionGuid <- deterministic target GUID
safe functional/default values <- canonical reference row
created/updated timestamps <- execution time where applicable
```

Create matching deterministic `TblLocalOutbox` insert events.

## Mandatory parent-before-child order

Inside one target transaction:

```text
BEGIN
  verify exact target and Development environment
  verify V008 rollback anchor
  acquire tenant/POS/v002 advisory transaction lock
  verify v001 marker and no v002 marker

  read reference permission defaults and employees via separate read-only connection

  adopt/update/insert the complete TblEmployeePermission set
  SaveChanges/read back inside the same transaction
  build PermissionName -> actual target EmployeePermissionGuid map
  verify exactly one parent for every selected employee

  insert/adopt all selected TblEmployee rows using actual mapped parent GUIDs
  insert permission and employee TblLocalOutbox rows

  verify TblPosRuntimeProfile remains Activated
  append TblPosRuntimeStateHistory only for a real transition
  verify excluded runtime/business tables unchanged
  write v002 marker last
  read back FK/count/invariant proof
COMMIT
```

Multiple `SaveChangesAsync()` calls are allowed only within the same connection/transaction. One final commit.

## Employee rules retained

Keep all prompt030–033 safeguards:

- all 20 valid reference employee starter rows are selected;
- TenantGuid is replaced;
- EmployeeGuid is deterministic;
- EmployeePermissionGuid comes only from the actual target permission map;
- `LoginNumber` respects `varchar(20)`;
- names/display fields use deterministic schema-aware shortening when required;
- PIN/contact/security/payroll/private values are reset or excluded;
- employee outbox payload is sanitized;
- staff/non-staff/virtual EmployeeType classification is preserved.

## Expected deltas

Derive actual counts from read-only preflight and canonical comparison.

Likely current first-run shape:

```text
TblEmployeePermission inserted: 4
TblEmployeePermission updated: 0..3 depending on canonical field comparison
TblEmployee inserted: 20
TblLocalOutbox delta:
  4 permission I events
  plus one U event for each existing permission materially corrected
  plus 20 employee I events
TblPosRuntimeStateHistory delta: 0
v002 marker delta: 1
```

Do not force a fixed outbox count when safe canonical reconciliation proves updates are needed.

## Conflict diagnostic

If reconciliation still returns `PHASE2_PERMISSION_BASELINE_CONFLICT`, report only safe metadata:

```text
PermissionName
conflict field name
reference safe classification
target safe classification
whether update is allowed by this prompt
```

Do not print employee names, GUID maps, PINs, contacts, payroll data, secrets, or raw rows.

## Physical execution policy

After source/build/tests pass, Codex may run a controlled non-UI physical harness against exactly `obm_pos_dev_v0_pg` only when it invokes the same production executor and validates the V008 backup anchor.

Otherwise leave the corrected WPF build for operator retry.

Never mutate:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
any other database
```

## UI hydration

After success and restart, InstallationV0 must read the marker/runtime tables and show:

```text
Phase 2 v002 Complete
RuntimeState: Activated
```

Do not reset to `Not Started` or `Upgrade Available` when the v002 marker and invariants exist.

## Replay

Second execution must produce:

```text
TblEmployeePermission delta = 0
TblEmployee delta = 0
TblLocalOutbox delta = 0
TblPosRuntimeProfile delta = 0
TblPosRuntimeStateHistory delta = 0
v002 marker delta = 0
```

## WPF label

If source changes, set:

```text
Build label: prompt035
Window title: OBM InstallationV0 Phase 1/2 - prompt035
```

If source already satisfies the new reconciliation behavior and only physical execution was missing, retain `prompt033` and prove why no source change is needed.

## Required tests

Add/update focused tests for:

```text
target with only 3 permissions is accepted as pre-v002 state
all 7 reference permission defaults are selected
existing permission actual GUID is preserved
safe canonical field differences are updated, not blocked
missing permissions inserted before employees
permission I/U outbox events are deterministic
employee FK uses actual target parent GUID
all employees resolve exactly one parent
one transaction and marker last
rollback on irreconcilable/duplicate parent
prompt032 varchar fix retained
runtime history no-op while Activated
restart hydration
same-version zero delta
active build label
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

## Report 035

Create and push:

```text
report/report035.md
```

Include:

1. Verdict.
2. Confirmation exact target has 3 pre-v002 permissions.
3. Explanation why this is expected, not automatically a conflict.
4. Reference permission schema/default inventory, sanitized.
5. Existing permission canonical comparison.
6. Inserted/updated/adopted permission counts.
7. Parent-before-employee proof.
8. Employee and permission outbox policy/deltas.
9. Transaction/rollback/marker-last proof.
10. Runtime profile/history proof.
11. Physical execution result or exact operator handoff.
12. Replay result/policy.
13. Restart hydration proof.
14. Source files changed.
15. Build/test counts.
16. Active label proof.
17. No reference mutation/no secret leakage/no source push.
18. Coordination commit SHA.

## Valid verdicts

```text
PHASE2_V002_FULL_PERMISSION_AND_EMPLOYEE_SEED_READY_FOR_USER_RETEST
```

```text
PHASE2_V002_FULL_PERMISSION_AND_EMPLOYEE_SEED_POS1_DB_PASS_READY_FOR_WPF_TEST
```

```text
BLOCKED_PHASE2_V002_IRRECONCILABLE_PERMISSION_CONFLICT
```

```text
BLOCKED_PHASE2_V002_PERMISSION_EMPLOYEE_IMPLEMENTATION
```
