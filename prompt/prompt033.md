# Prompt 033 — Seed `TblEmployeePermission` before `TblEmployee` and fix the employee FK

## Operator-confirmed hard dependency

The operator confirms the canonical dependency:

```text
TblEmployeePermission
        ↓ parent must exist first
TblEmployee
```

This is no longer an open design question.

**Hard rule:** before inserting even one `TblEmployee` row, the executor must first seed or adopt every required `TblEmployeePermission` parent row and build an authoritative mapping from `PermissionName` to the actual target `EmployeePermissionGuid`.

Required execution order inside the same PostgreSQL transaction:

```text
1. Determine all permission labels required by the selected reference employees.
2. Seed/adopt TblEmployeePermission rows first.
3. Save/verify permission parents while remaining inside the same transaction.
4. Build PermissionName -> actual target EmployeePermissionGuid map.
5. Validate every employee has exactly one resolved parent.
6. Only then insert/adopt TblEmployee rows.
7. Insert required permission and employee TblLocalOutbox rows.
8. Verify runtime profile/history.
9. Write the v002 completion marker last.
10. Commit once.
```

Multiple `SaveChangesAsync()` calls are allowed, but they must share:

```text
one DbContext/connection
one PostgreSQL transaction
one final COMMIT
```

Do not calculate an employee permission FK independently from the employee row when the parent exists or is inserted in this transaction.

## Physical evidence

Read completely before changing source:

```text
report/report030.md
report/report031.md
report/report032.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The operator physically retested build `prompt032` and received:

```text
SQLSTATE=23503
Schema=dbo
Table=TblEmployee
Constraint=FK_TblEmployee_Permission
```

The v002 employee transaction was blocked before commit.

Do not ask the operator to insert permission rows manually. The correction must be part of the InstallationV0 seed flow so a clean database can reproduce the same result.

## First requirement — prove rollback/no partial commit

Inspect `obm_pos_dev_v0_pg` read-only as role `hung` and record sanitized state:

```text
TblEmployee
TblEmployeePermission
TblLocalOutbox
v001 marker
v002 marker
TblPosRuntimeProfile
TblPosRuntimeStateHistory
```

Expected rollback state:

```text
TblEmployee = 0
TblEmployeePermission = 3 unless previous approved rows exist
TblLocalOutbox = 21
v001 marker = 1
v002 marker = 0
TblPosRuntimeProfile = 1, RuntimeState Activated
TblPosRuntimeStateHistory = 1
```

If any v002 employee, employee outbox, runtime-history transition, or v002 marker committed, stop with:

```text
BLOCKED_PHASE2_V002_PARTIAL_COMMIT_DETECTED
```

Do not delete or repair rows manually.

## Exact FK audit

Inspect PostgreSQL metadata and EF/entity configuration for:

```text
dbo.TblEmployee
FK_TblEmployee_Permission
dbo.TblEmployeePermission
```

Report safely:

```text
child FK column
parent PK/unique column
Tenant scope columns
nullable/non-null
ON DELETE/ON UPDATE behavior
```

Do not report employee names, PINs, contacts, payroll values, or raw rows.

## Required permission parent set

Read selected employees from `enailsalon_phasee1_pos1_pg` through:

```text
BEGIN TRANSACTION READ ONLY
SELECT-only queries
ROLLBACK
```

Collect only distinct safe permission labels required by those employees.

Expected labels from the current reference distribution are:

```text
Owner
Admin
Sub_Manager
Manager
Staff
AI_Assistant
VirtualAnyTechnician
```

Verify the current reference selection rather than blindly forcing the list.

The final parent set is the distinct `PermissionName` set actually used by the selected employee rows.

## Permission-first implementation

Stable key for permission parents:

```text
TenantGuid + PermissionName
```

For each required permission:

```text
absent
→ insert deterministic TblEmployeePermission row

compatible existing
→ adopt it and use its actual EmployeePermissionGuid

same stable key but incompatible role/default
→ rollback PHASE2_PERMISSION_BASELINE_CONFLICT

duplicate target rows for one stable key
→ rollback PHASE2_PERMISSION_DUPLICATE_PARENT
```

After permission parents are inserted/adopted, execute an in-transaction readback and create exactly one authoritative map:

```text
PermissionName -> actual target EmployeePermissionGuid
```

Before inserting employees, validate:

```text
every selected employee permission label exists in the map
no employee resolves to zero parents
no employee resolves to multiple parents
all mapped parent GUIDs physically exist in TblEmployeePermission
```

Failure result:

```text
PHASE2_EMPLOYEE_PERMISSION_PARENT_MISSING
```

## Employee insertion rule

Only after permission verification succeeds may `TblEmployee` rows be inserted/adopted.

Each employee must use:

```text
EmployeePermissionGuid = actual GUID from the permission map
```

Never use:

```text
reference EmployeePermissionGuid
an independently derived permission GUID
a GUID calculated from employee identity
a parent GUID that was not read back from the target transaction
```

Employee behavior remains:

```text
absent stable employee key -> insert transformed row
compatible existing employee -> adopt/verify
incompatible stable identity/type/permission -> rollback PHASE2_EMPLOYEE_BASELINE_CONFLICT
extra target employees outside v002 -> preserve
```

## Dependency/order acceptance gate

Tests and source inspection must prove this exact order:

```text
TblTenant
→ TblEmployeePermission
→ TblEmployee
→ required TblLocalOutbox rows
→ runtime profile/history verification
→ v002 marker last
```

No SQL statement that inserts `TblEmployee` may appear before permission parent resolution and verification.

## Permission and employee outbox policy

Reuse the proven legacy save/outbox behavior.

For permission rows actually inserted or materially updated:

```text
insert matching deterministic TblLocalOutbox row
EntityType = TblEmployeePermission
Operation = I or U
TenantGuid = Phase 1 TenantGuid
SourceClientId = Phase 1 POS source identity
payload = safe permission configuration only
```

For employee rows actually inserted or materially updated:

```text
insert matching deterministic TblLocalOutbox row
EntityType = TblEmployee
Operation = I or U
payload excludes PIN/contact/payroll/private fields
```

Compatible no-op replay must not duplicate outbox rows.

All permission rows, employees, outbox rows, runtime-state work, and v002 marker must share one target transaction.

## Expected current physical delta

Use these as expectations only after verifying actual target/reference counts:

```text
Required permission parents: 7
Existing compatible parents: 3
Missing permission parents: 4
Employees to insert: 20

Expected TblEmployeePermission delta: +4
Expected TblEmployee delta: +20
Expected TblLocalOutbox delta: +24
  4 permission events
  20 employee events
Expected TblPosRuntimeStateHistory delta: 0
Expected v002 marker delta: +1
```

If verified counts differ, use actual deterministic counts and explain why.

## Preserve prompt032 text-length corrections

Keep all prompt032 safeguards:

- `TblEmployee.LoginNumber` respects `varchar(20)`;
- display text uses schema-aware deterministic shortening;
- employee GUIDs do not depend on shortened display text;
- private/contact/security/payroll fields remain reset or excluded;
- exceptions expose only safe SQLSTATE/schema/table/column/constraint metadata.

Do not change the marker version:

```text
phase2-reference-driven-trial-v002-employees
```

No v002 marker has committed, so this remains a correction to the same logical upgrade.

Store notes in a new preserved folder, for example:

```text
InstallationV0\Phase2\Trials\reference-driven-v002-employees-r2\
```

Do not overwrite prior trial versions.

## Runtime state behavior

Keep prompt031 policy:

```text
TblPosRuntimeProfile = current state source of truth
TblPosRuntimeStateHistory = append only on a real transition
Phase2TrialCompletionMarker = immutable upgrade completion proof
```

Current profile is already `Activated`, therefore the corrected physical run should verify it and produce:

```text
TblPosRuntimeStateHistory delta = 0
```

## One atomic transaction

Use one `NpgsqlConnection` and one serializable target transaction:

```text
BEGIN
  verify target = obm_pos_dev_v0_pg Development
  verify V008 rollback anchor
  acquire advisory transaction lock
  verify v001 marker
  read reference employees/permissions through separate read-only connection

  seed/adopt TblEmployeePermission parents
  SaveChanges/readback inside same transaction
  build and validate permission GUID map

  insert/adopt TblEmployee using actual mapped parent GUIDs
  insert permission and employee outbox rows

  verify runtime profile/history
  verify excluded runtime/business tables unchanged
  write v002 marker last
  read back FK and row-count invariants
COMMIT
```

Any error must rollback:

```text
permission parents
permission outbox
employees
employee outbox
runtime profile/history changes
v002 marker
```

Preserve v001 data/marker and the Phase 1 checkpoint.

## Same-version replay

Second execution must prove:

```text
TblEmployeePermission delta = 0
TblEmployee delta = 0
TblLocalOutbox delta = 0
TblPosRuntimeProfile delta = 0
TblPosRuntimeStateHistory delta = 0
v002 marker delta = 0
```

## UI behavior

Keep the explicit operator action:

```text
Install Local Database Baseline
```

Do not auto-run during startup.

After success show safe counts:

```text
Permission parents inserted/adopted
Employees inserted/adopted
Outbox delta
Runtime profile state
Runtime history delta
Marker last
Transaction committed
```

Do not show employee names, PINs, contacts, payroll data, or raw GUID maps.

## WPF label

Because source changes, set:

```text
Build label: prompt033
Window title: OBM InstallationV0 Phase 1/2 - prompt033
```

Tests must prove prompt032/prompt031 are absent from the active label path.

## Required tests

Add focused tests proving:

```text
FK metadata/contract represented correctly
required permission labels derived from selected employees
TblEmployeePermission inserts occur before TblEmployee inserts
existing permission parent actual GUID is adopted
missing permission parents are inserted before employees
permission map is read back from target transaction
all employee FK values resolve before insert
missing/duplicate/incompatible parents fail closed
permission and employee outbox mapping
one transaction and marker last
rollback after permission or employee failure
prompt032 varchar safeguards remain
runtime history no-op while already Activated
same-version replay zero delta
prompt033 label
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Do not perform the physical WPF retry automatically. Leave it for the operator after build/tests pass.

## Report 033

Create and push:

```text
report/report033.md
```

Required sections:

1. Verdict.
2. Physical `23503` evidence.
3. Post-failure rollback proof.
4. Exact FK child/parent metadata.
5. Exact root cause.
6. Verified required permission-parent set.
7. Existing/adopted/inserted permission counts.
8. Proof `TblEmployeePermission` is seeded before `TblEmployee`.
9. Actual parent-GUID mapping design.
10. Permission and employee outbox policy.
11. One-transaction/marker-last proof.
12. Runtime profile/history behavior.
13. Expected first-run deltas.
14. Same-version replay policy.
15. Source files changed.
16. Build/test commands and counts.
17. Prompt033 label proof.
18. No reference mutation/no secret leakage/no source push.
19. Exact operator retest steps.
20. Coordination commit SHA.

## Valid verdicts

```text
PHASE2_V002_EMPLOYEE_PERMISSION_FK_FIX_READY_FOR_USER_RETEST
```

```text
BLOCKED_PHASE2_V002_PERMISSION_PARENT_CONFLICT
```

```text
BLOCKED_PHASE2_V002_EMPLOYEE_PERMISSION_FK_FIX
```
