# Prompt 036 — Diagnose and fix the physical PostgreSQL `42601` syntax failure

## Physical evidence

Read completely:

```text
report/report032.md
report/report033.md
report/report034.md
report/report035.md
prompt/prompt035.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The operator physically retested build `prompt035` and received:

```text
PHASE2_V002_POSTGRES_EXCEPTION
SqlState=42601
Schema=unknown
Table=unknown
Column=unknown
Constraint=unknown
DataType=unknown
```

PostgreSQL `42601` means the server rejected a SQL statement at parse/syntax time. Because schema/table/column are unknown, do not treat this as a data conflict, FK conflict, or varchar issue.

Do not ask the operator to repair rows manually. The correction must be made in the InstallationV0 production executor.

## Preserve all accepted behavior

Keep all prior accepted corrections:

```text
full 7-row TblEmployeePermission reconciliation
TblEmployeePermission before TblEmployee
actual target permission GUID readback map
20 reference employees
prompt032 varchar-length safeguards
one target transaction
permission and employee TblLocalOutbox rows
runtime profile/history integration
v002 marker last
same v002 marker version
```

Marker version remains:

```text
phase2-reference-driven-trial-v002-employees
```

## First gate — prove rollback/no partial commit

Inspect `obm_pos_dev_v0_pg` read-only as role `hung`:

```text
TblEmployeePermission
TblEmployee
TblLocalOutbox
v001 marker
v002 marker
TblPosRuntimeProfile
TblPosRuntimeStateHistory
```

Expected state if the `42601` transaction rolled back completely:

```text
TblEmployeePermission = 3
TblEmployee = 0
TblLocalOutbox = 21
v001 marker = 1
v002 marker = 0
TblPosRuntimeProfile = 1 / Activated
TblPosRuntimeStateHistory = 1
```

If any v002 permission, employee, outbox, runtime transition, or marker row committed, stop:

```text
BLOCKED_PHASE2_V002_PARTIAL_COMMIT_DETECTED
```

Do not delete or repair rows manually.

## Exact statement-stage diagnosis

Audit every SQL command executed by:

```text
InstallationV0\Phase2\PostgreSqlPhase2ReferenceSeedExecutor.cs
```

At minimum classify these stages:

```text
TARGET_PREFLIGHT
ADVISORY_LOCK
V001_MARKER_VERIFY
PERMISSION_PARENT_SELECT
PERMISSION_PARENT_INSERT
PERMISSION_PARENT_UPDATE
PERMISSION_PARENT_READBACK
PERMISSION_OUTBOX_INSERT
EMPLOYEE_INSERT
EMPLOYEE_OUTBOX_INSERT
RUNTIME_PROFILE_SELECT
RUNTIME_PROFILE_UPDATE
RUNTIME_HISTORY_INSERT
EXCLUDED_TABLE_VERIFY
V002_MARKER_INSERT
FINAL_READBACK
```

For each stage, record safely:

```text
stage id
command construction method
single-row or multi-row command
parameter count and parameter types
whether dynamic identifiers/fragments are used
whether empty collections can produce invalid SQL
whether commas/parentheses/VALUES/SET/ON CONFLICT clauses are generated dynamically
```

Do not print SQL parameter values, employee names, PINs, contacts, payroll values, tokens, passwords, or connection strings.

## Safe PostgreSQL exception diagnostics

Extend safe exception reporting so a future parser error includes:

```text
SqlState
StageId
Position / InternalPosition when provided
Routine when provided
safe statement fingerprint/hash
schema/table/column/constraint/datatype when provided
```

Do not print the full SQL text if it embeds literals or identifiers derived from private data.

A static stage identifier and SHA-256 fingerprint of the normalized command text are sufficient for public/UI diagnostics.

## Determine the exact root cause

Do not guess. Identify the exact invalid command template and parser position.

Explicitly inspect for common generator failures:

```text
empty VALUES list
trailing comma
missing comma
unbalanced parentheses
empty UPDATE SET clause
invalid ON CONFLICT syntax
invalid RETURNING placement
multiple commands concatenated without separator
parameter placeholder malformed
quoted identifier malformed
reserved word used unquoted
semicolon inside a generated expression
invalid CTE composition
invalid array/JSON cast syntax
```

The report must state the exact safe root cause, such as:

```text
PERMISSION_PARENT_INSERT generated an empty/trailing VALUES fragment
```

without exposing private row values.

## Command validation requirement

Create focused validation for every production SQL template.

Preferred order:

1. Unit/contract tests inspect the generated command structure.
2. Use Npgsql prepared commands or a dedicated parser-validation path with the same parameter types.
3. When a physical validation is necessary, run the same production executor or exact command templates inside an explicit target transaction that always rolls back.

Any physical parser validation must use only:

```text
obm_pos_dev_v0_pg
role hung
V008 rollback anchor verified
BEGIN
... exact command validation ...
ROLLBACK
```

It must not leave permission, employee, outbox, runtime-history, or marker deltas.

Do not use the reference database for mutation or parser testing.

## SQL implementation rules

Prefer fixed parameterized SQL templates over string concatenation.

For multi-row work:

```text
use one command per row, or
use a proven parameterized batch builder with explicit separators and non-empty guards
```

Do not generate SQL by joining raw values into command text.

Required safety behavior:

```text
zero rows to insert -> skip command entirely
zero rows to update -> skip UPDATE command entirely
zero outbox rows -> skip outbox insert entirely
```

No stage may execute syntactically incomplete SQL.

## Permission-first and employee FK rules remain mandatory

Correct transaction order remains:

```text
BEGIN
  target/anchor/preflight
  advisory lock
  v001 marker verify

  read all reference permission defaults read-only
  adopt/update/insert TblEmployeePermission
  read back PermissionName -> actual target EmployeePermissionGuid
  prevalidate all employee FKs

  insert/adopt TblEmployee
  insert permission and employee outbox rows

  verify runtime profile/history
  verify excluded runtime/business tables
  insert v002 marker last
  final readback
COMMIT
```

Current expected successful first-run deltas remain approximately:

```text
TblEmployeePermission +4
TblEmployee +20
TblLocalOutbox +24
TblPosRuntimeProfile 0
TblPosRuntimeStateHistory 0
v002 marker +1
```

Use verified actual counts if they differ.

## Failure and replay behavior

Any exception must roll back all v002 work while preserving v001 and Phase 1.

After first successful run, same-version replay must produce:

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
Build label: prompt036
Window title: OBM InstallationV0 Phase 1/2 - prompt036
```

Focused tests must prove prompt035 and older labels are not active.

## Required tests/builds

Add tests for:

```text
all production SQL stage templates parse/prepare successfully
empty insert/update/outbox sets skip execution
permission insert/update syntax
permission readback syntax
employee insert syntax
outbox insert syntax
runtime profile/history syntax
marker insert syntax
safe stage/fingerprint diagnostics
permission-before-employee order
actual permission GUID mapping retained
varchar(20) correction retained
one transaction and marker last
rollback on parser error
same-version replay zero delta
prompt036 label
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Do not automatically run the final physical WPF seed. Leave the user-visible retest to the operator after source/build/tests pass.

## Versioned correction artifact

Preserve previous trial folders. Create a new correction folder, for example:

```text
InstallationV0\Phase2\Trials\reference-driven-v002-employees-r4\
```

Record the exact corrected SQL stage/template and test coverage without private values.

## Report 036

Create and push:

```text
report/report036.md
```

Required sections:

1. Verdict.
2. Physical `42601` evidence.
3. Post-failure rollback proof.
4. Full safe SQL stage inventory.
5. Exact failing stage and safe statement fingerprint.
6. Parser position/routine evidence when available.
7. Exact root cause.
8. Source correction.
9. Empty-set behavior.
10. Permission-before-employee/FK mapping proof.
11. Transaction/rollback/marker-last proof.
12. Runtime profile/history behavior.
13. Expected successful physical deltas.
14. Same-version replay policy.
15. Source files changed.
16. Build/test commands and counts.
17. Prompt036 label proof.
18. No reference mutation/no secret leakage/no source push.
19. Exact operator retest steps.
20. Coordination commit SHA.

## Valid verdicts

```text
PHASE2_V002_POSTGRES_SYNTAX_FIX_READY_FOR_USER_RETEST
```

```text
BLOCKED_PHASE2_V002_SQL_TEMPLATE_UNRESOLVED
```

```text
BLOCKED_PHASE2_V002_POSTGRES_SYNTAX_FIX
```
