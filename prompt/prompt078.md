# Prompt 078 — 100% evidence audit and repair of Employee Weight Save + sync boundary

## Physical operator evidence overrides report077

The operator rebuilt and physically retested the source after prompt077.

The Staff employee grid loads successfully. The operator changed one approved employee-scoped value and clicked `Save`. The real application still failed with:

```text
EMPLOYEE_WEIGHT_SAVECHANGES_FAILURE
StageId=SaveChanges
TenantResolved=True
DirtyRows=1
ValidatedRows=0
UpdatedRows=0
```

The physical result supersedes the prior `READY_FOR_PHYSICAL_RETEST` conclusion. Treat prompt077 as a failed physical retest, not as proof that the save path is correct.

This save defect has survived multiple prompts. Evidence escalation is now **100% direct proof**.

Do not make another speculative patch. Do not return only counts, classifications, or high-level prose.

## Operator requirements

1. Employee Weight Save must have a clear transaction boundary.
2. A valid edit must persist only the intended current-tenant Staff employee fields.
3. The same successful edit must enter the existing synchronization/outbox flow.
4. Entity update and outbox write must be atomic: both commit or both roll back.
5. No unrelated checkout, employee management, queue, payroll, or TurnEngine code may be damaged.
6. The final private handoff must include the relevant **actual C# code blocks**, exact call chain, exception evidence, schema mapping, and before/after diff.

## Canonical row scope

Save may affect only rows satisfying the proven loader contract:

```text
current tenant
AND persisted EmployeePermission resolves to Staff
```

Do not update Owner/Admin/SubAdmin/non-Staff, virtual rows, or other-tenant rows.

## Canonical policy independence

Employee-scoped Save must not require or mutate:

```text
Included checkbox
Draft Policy
Active Policy
Policy Summary
API availability
```

Saving child employee settings must not create a Turn policy and must not persist the parent Included checkbox.

## Mandatory documentation/evidence gate

Read completely before any source edit:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report073.md
report/report074.md
report/report075.md
report/report076.md
report/report077.md
```

Read all local-only evidence from prompts 073–077, especially the save-pipeline evidence artifact from prompt077.

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
EVIDENCE_ESCALATION=100_PERCENT_DIRECT_PROOF
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase 0 — freeze source and prove the current failure before editing

Before changing code, capture the exact current implementation and failure.

### A. Required complete C# blocks in the private handoff

Return the complete current method bodies, with repository-relative file and line range, for:

1. Employee Weight `Save` button/click command.
2. DataGrid cell/row commit method used before Save.
3. Dirty-row detection and original-value snapshot logic.
4. Row validation method(s).
5. The application/service method that performs Save.
6. Tenant resolution used by Save.
7. Staff revalidation/query used by Save.
8. EF entity update/mapping code for every editable field.
9. Transaction begin/commit/rollback wrapper.
10. `SaveChanges`/`SaveChangesAsync` calls.
11. Existing outbox enqueue/builder/payload code.
12. Reload-after-save path.
13. Generic exception-to-result-code mapping/catch block.

Do not provide isolated one-line excerpts. Include enough surrounding code to prove control flow and ownership.

### B. Exact runtime exception evidence

The generic result code is insufficient. Obtain and record privately:

```text
ExceptionType
InnerExceptionType
Message (sanitized only where it contains private data)
PostgreSQL SqlState if present
Table/Column/Constraint if present
EF entries and states involved
full project stack trace from UI Save to failing statement
transaction state at failure
SaveChanges call number (first/second/etc.)
```

Do not swallow the inner exception behind `EMPLOYEE_WEIGHT_SAVECHANGES_FAILURE`.

If the existing source cannot expose this safely, add temporary local diagnostics or use debugger exception inspection, then remove temporary raw diagnostics before finalizing. The final application may retain sanitized structured diagnostics but must not print secrets or private employee data.

### C. Explain the contradictory counters

The physical status says:

```text
DirtyRows=1
ValidatedRows=0
StageId=SaveChanges
```

Prove exactly why `ValidatedRows` is zero even though execution reached `SaveChanges`.

Classify this as one of:

```text
COUNTER_INSTRUMENTATION_WRONG
VALIDATION_BYPASSED
VALIDATION_RESULT_NOT_PROPAGATED
SAVE_STAGE_ASSIGNED_TOO_EARLY
OTHER_PROVEN_CAUSE
```

Correct the counters only after proving the real behavior. Do not let diagnostics imply validation occurred when it did not.

## Phase 1 — direct schema, EF mapping, and privilege proof

Inspect the current local target database read-only and the active EF model.

For each editable Employee Weight field shown in the child UI, prove:

| UI field | Entity property | DB column | DB type | Nullable | Default | Max length/precision | Constraint/trigger | Sync payload field |
|---|---|---|---|---:|---|---|---|---|
| Employee Weight | | | | | | | | |
| Turn Enabled | | | | | | | | |
| Queue Order | | | | | | | | |
| Notes | | | | | | | | |

Also prove:

```text
- the columns physically exist in the target schema;
- the EF model maps to the correct columns and types;
- no shadow/duplicate property is being updated;
- no unsupported column is included in the UPDATE;
- the runtime DB role has UPDATE privilege on the table/columns;
- relevant triggers/constraints do not reject the write;
- current row concurrency/version contract, if any;
- decimal precision and integer/string ranges;
- Notes maximum length;
- whether duplicate Queue Order values are allowed.
```

Use read-only catalog/privilege queries. Do not mutate the current target DB automatically.

Classify the proven root cause using the narrowest code:

```text
GRID_EDIT_NOT_COMMITTED
DIRTY_SNAPSHOT_WRONG
VALIDATION_BYPASSED_OR_WRONG
ENTITY_NOT_TRACKED
ENTITY_ALREADY_TRACKED_CONFLICT
COLUMN_MAPPING_MISMATCH
SCHEMA_COLUMN_MISSING
TYPE_OR_PRECISION_MISMATCH
NOT_NULL_OR_LENGTH_CONSTRAINT
RUNTIME_ROLE_UPDATE_DENIED
CONCURRENCY_CONFLICT
OUTBOX_PAYLOAD_FAILURE
OUTBOX_SCHEMA_FAILURE
TRANSACTION_BOUNDARY_WRONG
RELOAD_MISREPORTED_AS_SAVE_FAILURE
OTHER_PROVEN_CAUSE
```

## Phase 2 — prove the existing sync contract before changing it

The operator requires every successful Employee Weight edit to enter the established sync flow.

Trace the real path:

```text
TblEmployee local edit
-> TblLocalOutbox (or the current canonical local outbox)
-> employee DTO/payload
-> API/apply contract
-> remote/local peer apply handler
```

Return privately the actual C# blocks and mapping evidence for:

```text
- employee outbox event creation;
- entity name and operation code;
- payload serialization;
- Employee DTO fields;
- API/receiver apply mapping for the four editable fields;
- idempotency key/version behavior;
- no-op handling.
```

If all four fields already exist in the sync contract, reuse it.

If one or more fields are absent, do not falsely claim sync is complete. Either implement the smallest additive mapping through the existing employee DTO/apply path with focused tests, or return:

```text
BLOCKED_EMPLOYEE_WEIGHT_SYNC_CONTRACT_MISSING
```

Do not create a second outbox or a separate TurnEngine sync channel.

## Phase 3 — canonical save boundary

Implement one clearly owned save service/method. The intended shape is:

```text
UI commits current DataGrid cell and row
-> capture dirty rows from original snapshots
-> if no dirty rows: return NO_CHANGES, no DB/outbox mutation
-> validate all dirty rows before transaction
-> resolve current tenant
-> begin one local DB transaction
-> reload each intended employee by EmployeeGuid + TenantGuid
-> verify persisted EmployeePermission == Staff
-> apply only changed employee-scoped fields
-> add exactly one canonical employee outbox update per changed employee
-> SaveChanges inside the same transaction
-> commit once
-> reload using the canonical Staff loader
-> return committed result
```

Entity updates and outbox rows must commit atomically.

No partial save is allowed. Any validation, entity, outbox, SaveChanges, or commit failure must roll back all dirty employee changes and outbox rows.

Do not reuse the parent Turn Policy transaction or create/activate a policy.

### SaveChanges count

Prefer one `SaveChangesAsync` inside the transaction for entity changes plus outbox additions.

If two calls are genuinely required, prove why and show that both remain inside the same transaction. Do not let the first call persist outside the outbox boundary.

### Updated row count

After commit, prove:

```text
DirtyRows == intended changed rows
ValidatedRows == all validated dirty rows
UpdatedRows == employee rows actually changed
OutboxRows == exactly one per changed employee
Committed=True
ReloadSucceeded=True/False separately
```

A reload failure after a successful commit must return a `SAVED_RELOAD_FAILED` result with `Committed=True`; it must not be reported as Save failure.

## Phase 4 — controlled real-schema integration proof

Mock-only tests are insufficient after repeated physical failure.

Run at least one real Npgsql integration test against a disposable database that reproduces the current target schema and constraints. Use the same EF model and verify the runtime role's privileges separately with read-only privilege queries.

Do not write to the operator's current target DB automatically.

The integration test must:

1. Create or provision a disposable schema/database through an existing approved harness.
2. Insert or use sanitized test tenant/permission/Staff employee rows only.
3. Load the row through the same Staff loader.
4. Change exactly one editable field.
5. Execute the production save service.
6. Verify the employee row changed.
7. Verify exactly one outbox update exists and contains the changed field.
8. Verify transaction rollback when outbox insertion fails.
9. Verify rerun/no-op creates no second outbox row.
10. Verify non-Staff and wrong-tenant writes are rejected.
11. Verify current-cell edit is committed before dirty detection.

If no safe disposable real-schema harness can be established, do not claim ready for physical retest. Return a blocker with exact missing prerequisite.

## Phase 5 — required tests

Add/repair focused tests for:

```text
- one Employee Weight change saves and reloads;
- one Turn Enabled change saves and reloads;
- one Queue Order change saves and reloads;
- one Notes change saves and reloads;
- value changed while cell remains focused is committed;
- no-op save makes zero DB/outbox writes;
- multiple dirty Staff rows save atomically;
- one invalid row blocks all writes;
- non-Staff row rejected;
- wrong-tenant row rejected;
- SaveChanges failure rolls back employee and outbox;
- outbox failure rolls back employee update;
- reload failure is distinguished from commit failure;
- API unavailable/401 does not block local save;
- diagnostics counters are accurate;
- sync payload includes all four editable fields;
- receiver/apply mapping preserves the fields when applicable.
```

Do not broaden the filter to unrelated migration suites.

## Phase 6 — private handoff requirements (mandatory)

The private Codex handoff must contain all of the following. A high-level summary alone is unacceptable:

1. Exact root-cause classification.
2. Exact inner exception and stack trace, sanitized only for secrets/private row data.
3. Exact current schema/mapping/privilege matrix.
4. Complete **before** C# blocks for the save click handler and production save method.
5. Complete **after** C# blocks for those methods.
6. Relevant complete outbox builder/payload/mapping C# blocks.
7. Unified diff for every changed save/sync method.
8. Exact transaction boundary diagram.
9. Exact `SaveChanges` count and why.
10. Exact employee/outbox rollback proof.
11. Real Npgsql disposable integration test code and output.
12. Build/test commands and counts.
13. Exact physical retest steps.
14. Explicit statement that no unrelated code path changed.

Do not include passwords, connection strings, PINs, raw GUIDs, or employee names in the handoff.

## Physical retest after direct proof passes

1. Rebuild and launch WPF manually.
2. Open Employee Weight Settings.
3. Change one non-sensitive field on one Staff row.
4. Keep the cell focused and click Save.
5. Confirm committed structured status and no error dialog.
6. Close/reopen and verify persistence.
7. Inspect safe outbox counts/payload shape without exposing private values.
8. Save again without changes and confirm NO_CHANGES/no new outbox.
9. Test one additional editable field.
10. Do not test checkout/payment yet.

## Safety boundaries

- No automatic mutation of the current target DB.
- Disposable DB writes are allowed only through an approved isolated harness.
- No WPF automation/clicking.
- No checkout/payment test.
- No Turn Policy creation/activation.
- No employee names, raw IDs, private values, credentials, or internal code detail in GitHub public report.
- No OBM source commit/push; source edits remain local.

## Evidence and reporting

Create a versioned local evidence artifact, for example:

```text
EmployeeWeightSaveBoundaryV001
```

The public `report/report078.md` must remain ultra-minimal and may contain only:

```text
verdict
root-cause category at a generic level
save boundary proven yes/no
sync/outbox boundary proven yes/no
real-schema integration proof yes/no
build/test counts
current DB mutated no
checkout tested no
one aggregate evidence SHA-256
```

Do not push C# code, source paths, schema/table/column names, exception internals, or architecture details to GitHub. Those belong only in the private handoff and local evidence.

## Valid verdicts

```text
OBM_POS_EMPLOYEE_WEIGHT_SAVE_AND_SYNC_BOUNDARY_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_EMPLOYEE_WEIGHT_SYNC_CONTRACT_MISSING
```

```text
BLOCKED_EMPLOYEE_WEIGHT_REAL_SCHEMA_INTEGRATION_PROOF
```

```text
BLOCKED_EMPLOYEE_WEIGHT_SAVE_ROOT_CAUSE_UNPROVEN
```

```text
BLOCKED_EMPLOYEE_WEIGHT_SAVE_BUILD_OR_TEST
```
