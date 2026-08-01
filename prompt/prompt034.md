# Prompt 034 — Continue Phase 2 v002 after full permission parent verification

## Current operator observation

The operator has inspected `dbo."TblEmployeePermission"` and observed the complete seven-row permission set for the current tenant:

```text
Sub_Manager
VirtualAnyTechnician
Staff
Owner
AI_Assistant
Manager
Admin
```

This matches the permission labels required by the 20 selected reference employees.

Prompt034 must verify this state in the approved target database and continue the employee v002 flow safely. Do not assume the screenshot alone proves which database was open; verify the exact target first.

## Authoritative state

Read completely:

```text
report/report030.md
report/report031.md
report/report032.md
report/report033.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Approved target:

```text
obm_pos_dev_v0_pg
```

Reference database:

```text
enailsalon_phasee1_pos1_pg
```

Current logical upgrade version remains:

```text
phase2-reference-driven-trial-v002-employees
```

Do not create a new marker version merely because this is a continuation. No v002 marker has yet committed.

## First step — exact target-state reconciliation

Inspect `obm_pos_dev_v0_pg` read-only as role `hung` and collect sanitized evidence for:

```text
TblEmployeePermission count
exact distinct PermissionName set
TenantGuid consistency across permission rows
duplicate PermissionName rows
TblEmployee count
TblLocalOutbox count
v001 marker count
v002 marker count
TblPosRuntimeProfile count/state
TblPosRuntimeStateHistory count
```

Expected current permission state:

```text
TblEmployeePermission count = 7
required permission labels all present exactly once
all permission rows scoped to the Phase 1 TenantGuid
```

Expected employee/marker state before successful v002 execution:

```text
TblEmployee = 0
v001 marker = 1
v002 marker = 0
RuntimeState = Activated
```

If any employee rows or v002 marker already exist, classify them before continuing:

```text
compatible complete v002 state -> verify/replay only
partial employee state without v002 marker -> BLOCKED_PHASE2_V002_PARTIAL_EMPLOYEE_STATE
incompatible permission/employee state -> BLOCKED_PHASE2_V002_STATE_CONFLICT
```

Do not delete, truncate, or manually repair rows.

## Permission parent acceptance gate

The seven-row permission set is considered complete only when all conditions pass:

```text
exact required labels present
one row per PermissionName
all rows use the Phase 1 TenantGuid
all rows are active/compatible according to current entity rules
all EmployeePermissionGuid values are non-null and unique
```

Build one authoritative map from target rows:

```text
PermissionName -> actual target EmployeePermissionGuid
```

Never derive or replace a permission GUID when a compatible target row already exists.

## Outbox reconciliation for existing permission rows

Inspect deterministic baseline outbox identity for the seven permission rows.

For each permission parent:

```text
required deterministic permission outbox already exists -> adopt/no-op
missing required deterministic permission outbox -> add it in the same v002 transaction
conflicting duplicate outbox identity -> BLOCKED_PHASE2_V002_PERMISSION_OUTBOX_CONFLICT
```

Do not create duplicate permission outbox rows merely because the permission table currently contains seven rows.

The public report may show safe counts only, not raw payloads or GUID maps.

## Continue TblEmployee v002 seed

After permission acceptance succeeds, continue the same v002 transaction flow:

```text
1. Revalidate Phase 1 protected identity.
2. Verify target = obm_pos_dev_v0_pg / Development.
3. Verify V008 pre-v002 rollback anchor.
4. Acquire advisory transaction lock.
5. Verify v001 marker and absence/compatibility of v002 marker.
6. Read the 20 reference employees through a separate read-only reference transaction.
7. Read/adopt the seven actual permission parents from target.
8. Resolve every employee PermissionName to the actual target EmployeePermissionGuid.
9. Validate all 20 employee FK mappings before the first employee insert.
10. Insert/adopt TblEmployee rows.
11. Insert missing deterministic permission/employee TblLocalOutbox rows.
12. Verify TblPosRuntimeProfile remains Activated.
13. Append TblPosRuntimeStateHistory only on a real state transition.
14. Verify excluded runtime/business tables remain unchanged.
15. Write the unchanged v002 marker last.
16. Read back all FK/count/invariant proofs.
17. COMMIT once.
```

All target mutations must use:

```text
one NpgsqlConnection
one serializable transaction
one final COMMIT
```

## Employee safeguards retained

Keep all prior corrections:

```text
TblEmployee.LoginNumber <= varchar(20)
text fields use schema-aware deterministic shortening
employee GUIDs remain deterministic
employee FK comes only from the target permission map
private/contact/PIN/payroll/security fields remain reset or excluded
outbox payload excludes private employee data
```

Do not print employee names or private values in the public report.

## Runtime-state policy retained

```text
TblPosRuntimeProfile = current runtime state source of truth
TblPosRuntimeStateHistory = append-only transition audit
Phase2TrialCompletionMarker = immutable v002 completion proof
```

Current state is expected to be `Activated`, therefore expected runtime-history delta is:

```text
0
```

## Expected physical result

If the target currently has all seven permission parents and none of the 20 employees:

```text
TblEmployeePermission data delta = 0
TblEmployee delta = +20
TblLocalOutbox delta = actual missing deterministic events
  expected commonly: +20 employee events
  plus only permission events that are truly missing
TblPosRuntimeProfile delta = 0
TblPosRuntimeStateHistory delta = 0
v002 marker delta = +1
Marker last = true
Transaction committed = true
```

Do not force an outbox delta of 24 if the four permission events already exist. Report actual reconciled counts.

## Same-version replay

After first success, run or expose a second execution and prove:

```text
TblEmployeePermission delta = 0
TblEmployee delta = 0
TblLocalOutbox delta = 0
TblPosRuntimeProfile delta = 0
TblPosRuntimeStateHistory delta = 0
v002 marker delta = 0
```

## UI hydration

After successful v002 execution and restart, WPF must read database state and display:

```text
Phase 2 v002 Complete
```

The action should become verify/no-op or disabled according to current UI policy; it must not reinsert employees.

## Source-change rule and label

First inspect whether prompt033 source already handles the current seven-parent target correctly.

If no source change is required:

```text
keep active build label prompt033
```

If a source correction is required:

```text
set Build label: prompt034
Window title: OBM InstallationV0 Phase 1/2 - prompt034
```

Do not change the label merely because a new coordination report is produced.

## Physical execution rule

Use the real InstallationV0 executor/operator path, not ad-hoc employee SQL.

Codex may run a controlled local harness only if it invokes the same production executor and uses the approved target/rollback guards. Otherwise, leave the final WPF action to the operator and report exact readiness.

Never mutate the reference database.

## Build/tests

If source changes are made, run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Required focused proof:

```text
seven permission parents accepted exactly once
actual target permission GUID map used
no independently derived permission FK
all 20 employee FKs resolve before insert
permission outbox reconciliation is idempotent
employee outbox mapping is idempotent
one transaction and marker last
runtime history no-op while already Activated
same-version replay zero delta
startup hydration shows v002 Complete
```

## Report 034

Create and push:

```text
report/report034.md
```

Required sections:

1. Verdict.
2. Exact target database proof.
3. Current permission count and safe label set.
4. Tenant/duplicate/active compatibility proof.
5. Current employee/outbox/marker/runtime counts.
6. Whether permission rows were adopted, inserted previously, or already complete.
7. Permission outbox reconciliation counts.
8. Actual target permission-GUID mapping design.
9. Employee FK prevalidation proof.
10. Physical employee seed result or exact reason it remains operator-pending.
11. Employee/outbox/runtime/marker deltas.
12. One-transaction/marker-last proof.
13. Same-version replay result or pending step.
14. Restart/UI hydration result or pending step.
15. Source files changed, if any.
16. Build/test commands and counts, if run.
17. Active build label.
18. No reference mutation/no secret leakage/no source push.
19. Exact next operator step.
20. Coordination commit SHA.

## Valid verdicts

Physical v002 employee seed completed:

```text
PHASE2_V002_FULL_PERMISSION_EMPLOYEE_SEED_POS1_DB_PASS_READY_FOR_WPF_TEST
```

Source/state verified and operator action pending:

```text
PHASE2_V002_FULL_PERMISSION_EMPLOYEE_SEED_READY_FOR_USER_TEST
```

Partial/conflicting state:

```text
BLOCKED_PHASE2_V002_PARTIAL_EMPLOYEE_STATE
```

```text
BLOCKED_PHASE2_V002_STATE_CONFLICT
```

Implementation blocker:

```text
BLOCKED_PHASE2_V002_EMPLOYEE_SEED_CONTINUATION
```
