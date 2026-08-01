# Prompt 032 — Diagnose and fix SQLSTATE 22001 during physical employee v002 seed

## Physical operator evidence

The operator ran the current `prompt031` WPF build and clicked the explicit Phase 2 action.

Observed UI result:

```text
Phase 2 blocked: 22001: value too long for type character varying(20)
```

Visible build label:

```text
prompt031
```

This is the first physical execution failure of:

```text
phase2-reference-driven-trial-v002-employees
```

Do not ask the operator to click the action again before the source correction is complete.

## Authoritative state

Read completely:

```text
report/report028.md
report/report030.md
report/report031.md
prompt/prompt030.md
prompt/prompt031.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Current target:

```text
obm_pos_dev_v0_pg
```

Valid pre-v002 rollback anchor:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV008\PreV002Backup
```

Expected pre-action state from report031:

```text
TblEmployee = 0
TblLocalOutbox = 21
v001 marker = 1
v002 marker = 0
TblPosRuntimeProfile = 1
RuntimeState = Activated
TblPosRuntimeStateHistory = 1
```

Because the v002 executor is required to use one transaction, the first task is to prove the failed `22001` attempt rolled back completely.

## Objective

Identify the exact table, exact column, exact target length, and safe value category causing SQLSTATE `22001`, then correct the employee v002 transform so it is schema-length-aware and can complete physically without leaking or corrupting employee data.

Do not guess that the failing field is `NickName`; treat that only as a hypothesis until proven.

Likely candidate classes include:

```text
TblEmployee display/name fields
permission/status/type text fields
TblLocalOutbox operation/entity/processor fields
runtime history reason fields
completion marker status fields
```

The actual offending table/column must be proven.

## Step 1 — Prove rollback before changing source

Using role `hung` and read-only queries, collect sanitized counts after the failed physical attempt:

```text
TblEmployee
TblLocalOutbox
Phase2TrialCompletionMarker v001 count
Phase2TrialCompletionMarker v002 count
TblPosRuntimeProfile count/current RuntimeState
TblPosRuntimeStateHistory count
```

Expected if rollback worked:

```text
TblEmployee = 0
TblLocalOutbox = 21
v001 marker = 1
v002 marker = 0
runtime profile/history unchanged
```

If any v002 data/outbox/marker/history row committed, stop with:

```text
BLOCKED_PHASE2_V002_PARTIAL_COMMIT_DETECTED
```

Do not clean rows manually. Report the exact sanitized delta and preserve the V008 backup.

## Step 2 — Capture the exact PostgreSQL failure location

Improve diagnostic handling temporarily or permanently so the executor captures safe fields from `PostgresException`:

```text
SqlState
SchemaName
TableName
ColumnName
ConstraintName when present
DataTypeName when present
safe failing row/table stable key
```

Never log or report the raw employee value, employee name, PIN, contact, payroll value, token, connection string, or full SQL parameter payload.

If PostgreSQL does not populate `ColumnName`, identify the exact column by schema-aware prevalidation before sending SQL.

## Step 3 — Audit actual target text limits

Read target metadata for all text columns touched by the v002 transaction:

```text
TblEmployee
TblLocalOutbox
TblPosRuntimeProfile
TblPosRuntimeStateHistory
Phase2TrialCompletionMarker
```

For every touched `character varying(n)` column, report:

```text
Table
Column
Maximum length
Value source category
Current transform rule
Risk
Correction
```

Do not print employee values.

Also compare target column lengths with the reference employee schema and current entity/model configuration. The target database schema is authoritative for insertion limits.

## Step 4 — Schema-length-aware transform policy

Implement one reusable length-validation/sanitization layer for v002 transformed text fields before SQL execution.

Classify each text field:

### A. Canonical enum/status/operation fields

Examples:

```text
RuntimeState
operation code
permission label
marker status
outbox processor/entity labels
```

Rules:

- use the exact canonical value expected by current code/schema;
- never truncate an enum/status into a different value;
- if the canonical constant exceeds the schema, fail closed with a dedicated developer result and report the schema mismatch.

### B. Employee display/edit fields

Examples may include employee nickname/display name.

Rules:

- normalize whitespace;
- preserve the local trial display value when it fits;
- when it exceeds the target maximum, produce a deterministic UI-safe shortened value;
- preserve uniqueness among all selected employees;
- use a deterministic suffix derived from the reference stable employee identity when truncation could collide;
- do not expose the original overlength value in logs or the public report;
- do not alter EmployeeType or permission classification.

Example algorithm shape for a `varchar(20)` display field:

```text
normalized value fits -> preserve
otherwise -> prefix fitting available length + deterministic short suffix
```

Do not use simple blind `Substring(0, 20)` if it can create duplicate employee stable/display keys.

### C. Private/security fields

Continue clearing/resetting/excluding them. Do not truncate and preserve private data.

### D. Diagnostic/reason text

Use concise canonical reason codes that fit the physical schema. Long human-readable explanation belongs in safe UI/report text, not a narrow DB code column.

## Step 5 — Stable key and idempotency

The correction must not change employee deterministic identity merely because a display value is shortened.

Employee deterministic GUID/stable identity must remain based on:

```text
v002 marker contract + Phase 1 Tenant/POS identity + reference stable employee identity
```

It must not be based solely on the transformed/truncated display name.

Outbox deterministic identity must remain stable across replay.

## Step 6 — Preserve runtime-state behavior

Do not change the runtime-state policy from report031:

```text
TblPosRuntimeProfile is current source of truth
TblPosRuntimeStateHistory is append-only only on actual transition
current Activated state -> verify only, history delta 0
v002 marker written last
```

The corrected transaction still owns:

```text
employees
employee outbox
runtime profile update if needed
runtime history append if needed
v002 marker
```

One exception must roll back all of them.

## Step 7 — Source versioning and WPF label

Because source changes are required, set:

```text
Build label: prompt032
Window title: OBM InstallationV0 Phase 1/2 - prompt032
```

Preserve prior trial artifacts. Add a correction artifact such as:

```text
InstallationV0\Phase2\Trials\reference-driven-v002-employees-r1\README.md
```

Keep the immutable completion marker version:

```text
phase2-reference-driven-trial-v002-employees
```

because no physical v002 marker was committed. Record implementation revision `r1` separately; do not create a false completed version.

## Step 8 — Tests

Add focused tests for:

```text
SQLSTATE 22001 diagnostic classification
exact touched varchar length inventory
employee display value fits unchanged
overlength display value shortened safely
multiple overlength employees remain unique
deterministic shortened value on replay
employee GUID unchanged by display shortening
canonical enum/status fields are not blindly truncated
private fields remain excluded
outbox payload respects target text limits
runtime history reason fits schema
one transaction rollback on length validation failure
v002 marker remains absent on failure
same-version replay zero delta
prompt032 label
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

## Step 9 — Physical retry

After build/tests pass and rollback is proven, run or hand back the `prompt032` WPF action for one physical retry.

Expected first successful v002 result:

```text
TblEmployee inserted/adopted = reference-selected employee count
employee outbox inserted = matching count for actual inserted/updated employees
RuntimeState = Activated
TblPosRuntimeStateHistory delta = 0 when already Activated
v002 marker delta = 1
marker last = true
transaction committed = true
```

Then restart WPF and prove UI hydrates:

```text
Phase 2 v002 Complete
```

Same-version replay must produce:

```text
TblEmployee delta = 0
TblLocalOutbox delta = 0
TblPosRuntimeProfile delta = 0
TblPosRuntimeStateHistory delta = 0
v002 marker delta = 0
```

## Safety

Do not mutate:

```text
enailsalon_phasee1_pos1_pg
any production/protected/reference DB
```

Do not print employee names or failing raw values in the public report.
Do not read/print pgpass contents or secrets.
Do not manually delete target rows to make the retry pass.
Do not change employee UI filtering logic.

## Report

Create and push:

```text
report/report032.md
```

Report must include:

1. Verdict.
2. Physical `22001` evidence classification.
3. Post-failure rollback count proof.
4. Exact offending table/column and target maximum length, without raw value.
5. Full touched varchar-length matrix.
6. Root cause.
7. Length-aware transform correction.
8. Collision-safe employee display shortening policy.
9. Stable GUID/idempotency proof.
10. Outbox/runtime-history/marker corrections if applicable.
11. Exact source files changed.
12. Build/test commands and counts.
13. Physical retry result or exact reason pending.
14. Runtime profile/history physical deltas.
15. v002 marker-last and replay proof.
16. Prompt032 label proof.
17. No private data/reference mutation/secret/source push proof.
18. Coordination commit SHA.

## Valid verdicts

Corrected implementation ready for operator retry:

```text
PHASE2_V002_VARCHAR_LENGTH_FIX_READY_FOR_USER_RETEST
```

Physical v002 and replay passed:

```text
PHASE2_V002_EMPLOYEES_RUNTIME_STATE_PHYSICAL_PASS
```

Partial commit detected:

```text
BLOCKED_PHASE2_V002_PARTIAL_COMMIT_DETECTED
```

Schema/implementation unresolved:

```text
BLOCKED_PHASE2_V002_VARCHAR_LENGTH_FIX
```
