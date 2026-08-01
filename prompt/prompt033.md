# Prompt 033 — Fix Phase 2 v002 employee-permission FK mapping

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

Do not ask the operator to insert permission rows manually and do not repair the target database outside the InstallationV0 seed path.

## Likely fault classes — verify, do not assume

The failure can be caused by one or both of these conditions:

1. The v001 target contains only the three permission rows `Owner`, `Admin`, and `Sub_Manager`, while the 20 selected reference employees require additional permission parents such as `Staff`, `Manager`, `AI_Assistant`, and `VirtualAnyTechnician`.
2. The employee transform derives an `EmployeePermissionGuid` independently instead of resolving the actual target parent row by stable key, so it may reference a GUID that does not match an existing compatible permission row.

Prompt033 must identify the exact condition and correct both classes safely.

## First requirement — prove rollback/no partial commit

Before source correction, inspect `obm_pos_dev_v0_pg` read-only as role `hung` and record sanitized counts/state:

```text
TblEmployee
TblEmployeePermission
TblLocalOutbox
v001 marker
v002 marker
TblPosRuntimeProfile
TblPosRuntimeStateHistory
```

Expected state if rollback succeeded:

```text
TblEmployee = 0
TblEmployeePermission = 3 unless prior manual/test rows exist
TblLocalOutbox = 21
v001 marker = 1
v002 marker = 0
TblPosRuntimeProfile = 1, RuntimeState Activated
TblPosRuntimeStateHistory = 1
```

If any v002 employee, employee outbox, runtime-history, or marker row committed, stop with:

```text
BLOCKED_PHASE2_V002_PARTIAL_COMMIT_DETECTED
```

Do not delete or repair rows manually.

## Exact FK audit

Inspect PostgreSQL metadata and EF/entity configuration for:

```text
dbo.TblEmployee
constraint FK_TblEmployee_Permission
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

Do not report employee names, PINs, contacts, payroll values, or raw employee rows.

## Required permission-parent set

Read the reference employees in `enailsalon_phasee1_pos1_pg` through a read-only transaction and collect only the distinct safe permission labels used by the selected 20 employees.

Expected reference distribution from report030 includes:

```text
Owner
Admin
Sub_Manager
Manager
Staff
AI_Assistant
VirtualAnyTechnician
```

Do not hard-code this list without verifying it against the current reference selection. The final parent set must be the distinct permission stable keys actually required by the selected employee rows.

## Parent-before-child correction

The v002 executor must perform this order inside the same target transaction:

```text
1. Resolve all required TblEmployeePermission parents by TenantGuid + PermissionName.
2. Adopt compatible existing parent rows and use their actual EmployeePermissionGuid.
3. Insert missing permission parent rows before TblEmployee.
4. Build one authoritative in-transaction map:
   PermissionName -> actual target EmployeePermissionGuid.
5. Validate every selected employee resolves to exactly one parent.
6. Insert/adopt TblEmployee using the resolved actual parent GUID.
7. Insert matching deterministic outbox rows.
8. Verify runtime profile/history.
9. Write the unchanged v002 marker last.
```

Hard rule:

```text
Never derive an employee FK independently when a compatible parent already exists.
```

The employee FK must come from the target parent row selected or inserted in the current transaction.

## Permission-row behavior

Stable key:

```text
TenantGuid + PermissionName
```

For each required permission:

```text
absent -> insert deterministic permission row
compatible existing -> adopt and use actual existing GUID
same stable key but incompatible role/type -> rollback PHASE2_PERMISSION_BASELINE_CONFLICT
duplicate target rows for one stable key -> rollback PHASE2_PERMISSION_DUPLICATE_PARENT
```

Do not delete extra permissions outside the selected v002 set.

Preserve only safe role/default fields. Do not attach employee PINs, private identities, or payroll information to permission rows.

## Permission outbox policy

Reuse the proven baseline save/outbox behavior:

- newly inserted/materially updated `TblEmployeePermission` rows receive deterministic `TblLocalOutbox` rows;
- compatible no-op adopted rows do not duplicate an existing baseline outbox event;
- if policy requires an event and the compatible adopted row lacks the deterministic baseline event, add it in the same transaction;
- payload contains only safe permission configuration.

All permission rows, permission outbox rows, employee rows, employee outbox rows, runtime-state work, and v002 marker must use one target transaction.

## Expected physical delta for the current known target

Use these values only as an acceptance expectation after verifying current counts:

```text
Required distinct permission parents: 7
Existing compatible permission parents: 3
Missing permission parents: 4
Employee rows to insert: 20
Expected first-run outbox delta: 24
  4 permission parent outbox rows
  20 employee outbox rows
RuntimeStateHistory delta: 0 because profile is already Activated
v002 marker delta: 1
```

If current verified target/reference counts differ, report and use the actual deterministic counts instead of forcing these numbers.

## Existing employee transform safeguards

Keep all prompt032 corrections:

- `LoginNumber` respects `varchar(20)`;
- display fields use schema-aware deterministic shortening;
- private/contact/security/payroll values remain reset or excluded;
- employee GUIDs remain deterministic and independent of shortened display text;
- PostgreSQL exceptions expose only safe schema/table/constraint metadata.

Do not change the v002 marker version:

```text
phase2-reference-driven-trial-v002-employees
```

Because no v002 marker has committed, this is a correction to the same immutable logical upgrade, not a new production version.

Store correction notes in a new preserved trial folder, for example:

```text
InstallationV0\Phase2\Trials\reference-driven-v002-employees-r2\
```

Do not overwrite prior trial folders.

## Runtime-state behavior

Keep prompt031 policy unchanged:

```text
TblPosRuntimeProfile = current runtime source of truth
TblPosRuntimeStateHistory = append only on a real state transition
Phase2TrialCompletionMarker = immutable seed completion proof
```

Current profile is `Activated`, so the corrected first execution should verify it and produce:

```text
TblPosRuntimeStateHistory delta = 0
```

## One transaction and failure behavior

Use one `NpgsqlConnection` and one serializable transaction:

```text
BEGIN
  verify target and V008 rollback anchor
  advisory transaction lock
  verify v001 marker
  read reference employees/permissions through separate read-only connection
  resolve/insert permission parents
  insert/adopt employees using actual parent GUIDs
  insert permission and employee outbox rows
  verify runtime profile/history
  verify excluded runtime/business tables unchanged
  write v002 marker last
  read back FK/invariants
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

Preserve v001 data/marker and Phase 1 checkpoint.

## UI and operator behavior

The explicit action remains:

```text
Install Local Database Baseline
```

Do not auto-run on startup.

After success show safe counts:

```text
Permission parents inserted/adopted
Employees inserted/adopted
Outbox delta
Runtime state/history delta
Marker last
Transaction committed
```

On FK/preflight failure show a safe result code, not raw employee data.

## Same-version replay

A second execution must prove:

```text
TblEmployeePermission delta = 0
TblEmployee delta = 0
TblLocalOutbox delta = 0
TblPosRuntimeProfile delta = 0
TblPosRuntimeStateHistory delta = 0
v002 marker delta = 0
```

## WPF label

Because source changes, set:

```text
Build label: prompt033
Window title: OBM InstallationV0 Phase 1/2 - prompt033
```

Focused tests must prove prompt032/prompt031 are absent from the active title/build-info path.

## Required tests

Add focused tests for:

```text
actual FK metadata/contract represented correctly
required permission set derived from selected employees
parent permissions inserted before employees
existing parent GUID is adopted and used by employee FK
missing permission parent inserted and mapped
all employee FK values resolve before insert
duplicate/missing/incompatible parent fails closed
permission outbox mapping
employee outbox mapping
one transaction and marker last
rollback after permission or employee failure
prompt032 varchar safeguards retained
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

Do not automatically perform the physical WPF retry. Leave it for the operator after build/tests pass.

## Report 033

Create and push:

```text
report/report033.md
```

Required sections:

1. Verdict.
2. Physical 23503 evidence.
3. Post-failure rollback proof.
4. Exact FK child/parent metadata.
5. Exact root cause: missing parents, wrong GUID mapping, or both.
6. Required distinct permission-parent set, sanitized.
7. Existing/adopted/inserted permission counts.
8. Parent-to-employee GUID mapping design.
9. Permission and employee outbox policy.
10. One-transaction/marker-last proof.
11. Runtime profile/history behavior.
12. Expected physical first-run deltas.
13. Same-version replay policy.
14. Source files changed.
15. Build/test commands and counts.
16. Prompt033 label proof.
17. No reference mutation/no secret leakage/no source push.
18. Exact operator retest steps.
19. Coordination commit SHA.

Do not print employee names, PINs, contacts, payroll values, raw GUID maps, or private row dumps.

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
