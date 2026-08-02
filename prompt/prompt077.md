# Prompt 077 — Fix Employee Weight Save pipeline and preserve Staff-only scope

## Operator evidence

The operator physically tested the current WPF source after prompt076.

The Employee Weight child loader is now working:

```text
Employee Weight Settings opens
-> current-tenant Staff rows are visible
-> Employee Weight / Turn Enabled / Queue Order / Notes are editable
```

The operator changed an approved employee-scoped value and clicked `Save`.

The window then reported:

```text
Save Failed (EMPLOYEE_WEIGHT_SAVE_FAILED)
Unable to save Employee Weight settings. Result code:
EMPLOYEE_WEIGHT_SAVE_FAILED
```

The grid remained open with the loaded Staff rows.

This task is only for the Employee Weight child save path. Do not test checkout/payment.

## Authoritative contracts already proven

### Row scope

Load and save only employees for the current tenant whose persisted `EmployeePermission` resolves to `Staff`.

Do not broaden the saved set to Owner/Admin/SubAdmin/non-Staff or other-tenant rows.

### Policy independence

Employee-scoped settings do not require:

- Included checkbox checked;
- Draft Policy;
- Active Policy;
- Policy Summary;
- API availability.

Saving the child must not create or modify a Turn Policy and must not persist the parent Included checkbox.

### Explicit operator mutation only

Opening and loading are read-only.

Database mutation occurs only after the operator explicitly clicks `Save` in the child window.

## Exact objective

Determine the real failure stage hidden behind the generic result code, implement the smallest proven correction, and guarantee:

```text
valid dirty Staff rows
-> validate all requested changes
-> one local DB transaction
-> update only changed employee-scoped fields
-> reuse existing outbox path only when contract requires it
-> commit once
-> reload through canonical Staff loader
-> show Save Ready / Saved result
```

Failure must produce no partial employee updates and no partial/duplicate outbox rows.

## Mandatory documentation gate

Read before editing:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report073.md
report/report074.md
report/report075.md
report/report076.md
```

Read local-only evidence:

```text
EmployeeWeightSettingsV001
EmployeeWeightStaffLoaderV001
```

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase A — trace the exact Save call chain

Map the real active path:

```text
operator edits DataGrid row
-> DataGrid commit edit / binding update
-> Save button command
-> dirty-row detection
-> row validation
-> tenant resolution
-> reload current Staff entities
-> apply approved field changes
-> BeginTransaction
-> SaveChanges
-> optional local outbox write
-> Commit
-> canonical reload
-> status/result update
```

For each stage record locally:

- repository-relative file/method;
- inputs;
- row count entering/leaving the stage;
- whether mutation has begun;
- exact failure type and safe message;
- rollback behavior.

Do not stop at `EMPLOYEE_WEIGHT_SAVE_FAILED`.

Capture the sanitized underlying exception classification, such as:

```text
VALIDATION_FAILED
NO_DIRTY_ROWS
GRID_EDIT_NOT_COMMITTED
TENANT_CONTEXT_NOT_RESOLVED
STAFF_ROW_NOT_FOUND
WRONG_TENANT_ROW
EMPLOYEE_PERMISSION_CHANGED
SCHEMA_MISMATCH
COLUMN_MAPPING_FAILURE
NULLABILITY_FAILURE
CONCURRENCY_FAILURE
UNIQUE_OR_CHECK_CONSTRAINT_FAILURE
DATABASE_PERMISSION_FAILURE
SAVECHANGES_FAILURE
OUTBOX_SERIALIZATION_FAILURE
OUTBOX_CONSTRAINT_FAILURE
TRANSACTION_COMMIT_FAILURE
RELOAD_AFTER_SAVE_FAILURE
OTHER_PROVEN_CAUSE
```

A reload failure after a committed save must not be reported as if the save rolled back. Distinguish `CommittedButReloadFailed` from `NotCommitted`.

## Phase B — force pending DataGrid edits into the model

Audit whether the operator can click `Save` while a checkbox/text/numeric cell still has keyboard focus.

Before collecting dirty rows, the Save command must use the application's existing WPF pattern to commit:

```text
cell edit
row edit
binding source update
```

Add tests proving that a just-edited checkbox or numeric cell is included in the Save request without requiring the operator to click another row first.

Do not use focus hacks or repeated UI timers.

## Phase C — prove field ownership and target schema

For these editable concepts, prove the actual active model and persisted representation:

```text
Employee Weight
Turn Enabled
Queue Order
Notes
```

For each field determine:

- actual table/column;
- CLR and PostgreSQL types;
- nullable/non-nullable behavior;
- default behavior when unset;
- validation range/length;
- whether existing rows in the target DB have compatible values;
- whether the current runtime DB role can update the field;
- whether the field is included in the existing sync/outbox DTO.

Inspect current target and reference schema only in read-only transactions.

Do not print employee names, raw identifiers, PINs, connection strings, or business rows in public artifacts.

If the target DB lacks a required column or constraint differs from the source model:

```text
classify SCHEMA_MISMATCH
prepare an idempotent operator-review schema repair artifact
DO NOT apply it automatically
```

Do not silently move values into unrelated legacy columns.

## Phase D — validation contract

Implement or correct explicit row validation before mutation.

### Employee Weight

Prove the valid numeric range and precision from active source/reference evidence. Do not invent a range.

### Turn Enabled

Persist the explicit boolean contract. It must not require login enabled, PIN presence, Draft Policy, or parent Included state.

### Queue Order

The physical grid currently shows the same default-looking Queue Order value on multiple Staff rows.

Do not assume uniqueness.

Audit whether Queue Order is:

- allowed to duplicate;
- required unique;
- required contiguous;
- only meaningful for Turn Enabled rows;
- a placeholder/default when not configured.

If duplicates are valid, do not reject them.

If uniqueness is truly required, surface a row-specific validation message before mutation; do not return only the generic save code.

### Notes

Prove max length/null handling. Blank notes must be handled according to schema without producing an exception.

### Whole-save atomicity

Any invalid dirty row must block the entire save before mutation begins.

Return structured validation information without exposing private employee data. Use row index or safe display label only where permitted in the local operator handoff.

## Phase E — dirty-row detection and no-op behavior

Compare editable current values to the loaded baseline.

Required behavior:

```text
no changes
-> no DB write
-> no outbox write
-> safe EMPLOYEE_WEIGHT_SAVE_NO_CHANGES result
```

```text
one changed Staff row
-> update only that row
```

```text
multiple changed Staff rows
-> one transaction
-> all or none
```

Do not mark every loaded employee dirty merely because nullable/default values were normalized for display.

After successful save, replace the baseline with persisted values so a second immediate Save is a no-op.

## Phase F — persistence and concurrency

Inside one transaction:

1. Resolve current tenant using the same canonical tenant resolver as load.
2. Resolve dirty rows by stable employee key.
3. Re-check each row still belongs to the current tenant and still has `EmployeePermission == Staff`.
4. Apply only editable employee-scoped fields.
5. Call SaveChanges once unless the existing outbox transaction pattern proves another ordering is required.
6. Write only required outbox entries for changed rows.
7. Commit once.

Handle concurrent changes explicitly:

- missing employee;
- permission changed from Staff;
- row moved tenant;
- concurrency token mismatch, if present.

Do not overwrite unrelated employee profile/login/payroll fields.

## Phase G — outbox audit

Prove whether these employee-scoped TurnEngine fields are part of existing employee sync.

If yes:

- reuse the canonical employee update payload/path;
- include only contractually required data;
- one intended outbox event per changed row;
- no outbox event for no-op;
- outbox write must participate in the same transaction.

If no active sync contract exists for these fields:

- do not invent a new event type in this bounded fix;
- report the gap privately;
- local save must still work if local-only behavior is canonical.

## Phase H — precise UI result behavior

Replace the generic-only failure path with a structured save result containing concepts equivalent to:

```text
TenantResolved
DirtyRows
ValidatedRows
UpdatedRows
OutboxRows
Committed
ReloadSucceeded
ResultCode
SafeMessage
```

Expected successful result:

```text
EMPLOYEE_WEIGHT_SAVE_READY
```

Expected no-op result:

```text
EMPLOYEE_WEIGHT_SAVE_NO_CHANGES
```

Validation failures must use a distinct result code and row-level safe message.

Schema mismatch, DB permission, SaveChanges, outbox, commit, and post-commit reload failures must remain distinguishable.

Do not expose raw SQL, credentials, identifiers, or private values in dialogs or GitHub reports.

## Phase I — save/reload correctness

After commit:

```text
reload using the same canonical Staff loader
-> show persisted values
-> clear dirty state
```

If reload fails after commit:

```text
show "Saved, but reload failed"
Committed=true
```

Do not encourage the operator to click Save repeatedly when the first save may already have committed.

## Tests

Add focused tests covering at least:

1. pending DataGrid checkbox edit is committed before dirty detection;
2. pending numeric edit is committed before dirty detection;
3. no-draft state can save employee-scoped values;
4. unchecked parent Included state can save employee-scoped values;
5. one dirty current-tenant Staff row saves;
6. multiple Staff rows save atomically;
7. no-op save creates zero mutation/outbox;
8. non-Staff/other-tenant row is rejected even if supplied to save;
9. permission changed concurrently is rejected safely;
10. nullable optional fields save safely;
11. Queue Order duplicate behavior matches proven contract;
12. validation blocks all mutation;
13. SaveChanges failure rolls back employee and outbox changes;
14. outbox failure rolls back employee changes when outbox is required;
15. post-commit reload failure reports committed state correctly;
16. API unavailable/401 does not block local Save;
17. second Save after successful reload is a no-op.

Run the WPF build and a narrow Employee Weight save/load/Turn Settings test filter only. Do not include unrelated migration suites.

## Current DB mutation boundary

Codex must not click the UI or mutate the current operator database automatically.

Read-only schema/count inspection is allowed.

Source corrections and automated tests may use isolated/disposable test transactions or existing test databases according to repository rules.

The operator will perform the physical Save after rebuild.

## Physical retest steps

Provide these steps in the private handoff:

1. Rebuild and launch WPF manually.
2. Open Employee Turn Settings.
3. Open Employee Weight Settings regardless of Included state.
4. Confirm current-tenant Staff rows load.
5. Change exactly one approved field on one Staff row.
6. While the edited cell still has focus, click Save.
7. Confirm a success result, not `EMPLOYEE_WEIGHT_SAVE_FAILED`.
8. Close and reopen; confirm persistence.
9. Click Save again without changes; confirm no-op result and no duplicate write.
10. Test one additional field only after the first change passes.
11. Do not test checkout/payment in this task.

## Evidence and reporting

Create a new versioned local evidence artifact, for example:

```text
EmployeeWeightSaveV001
```

Keep detailed call chains, schema findings, exception classification, DB counts, and source paths local only.

Create `report/report077.md` as an ultra-minimal public summary containing only:

- verdict;
- save pipeline corrected yes/no;
- atomic dirty-row save yes/no;
- no-op save yes/no;
- Staff-only scope preserved yes/no;
- build/test counts;
- current DB mutated no;
- checkout tested no;
- one aggregate local evidence SHA-256.

Do not include internal paths, source symbols, table/column names, schema/count metadata, employee information, exact values, DB names, or architecture details in GitHub.

Expected success verdict:

```text
OBM_POS_EMPLOYEE_WEIGHT_SAVE_READY_FOR_PHYSICAL_RETEST
```

If a current-target schema repair is required, use:

```text
OBM_POS_EMPLOYEE_WEIGHT_SAVE_SCHEMA_REPAIR_READY_FOR_OPERATOR_REVIEW
```

If blocked for another proven reason, use the narrowest exact verdict and explain it only in the private handoff.