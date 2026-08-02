# Prompt 079 — Verify physical Employee Weight outbox delivery and close the remaining no-Draft Turn Settings load failure

## Operator physical evidence

The operator has now physically confirmed:

```text
Employee Weight Settings load: PASS
Employee Weight Save: PASS
Saved value persists after reload: PASS
```

Two bounded tasks remain:

1. Verify that the successful Employee Weight edit produced the correct `TblLocalOutbox` update for synchronization.
2. Correct the parent `Employee Turn Settings` open/load path, which still displays:

```text
Unable to load draft turn policy. Result code:
TURN_POLICY_DRAFT_LOAD_FAILED
```

Do not test checkout/payment in this prompt.

## Important evidence from prompt078

The final Employee Weight save is intended to:

```text
apply employee-scoped changes
-> queue one employee outbox update per changed Staff employee
-> execute one SaveChangesAsync
-> commit employee update and outbox atomically
```

However, the current outbox helper previously contained a potentially silent path equivalent to:

```csharp
if (factory == null)
    return 0;
```

Therefore a physically successful employee save does not by itself prove that a `TblLocalOutbox` row exists. This prompt must prove the real current database state and eliminate any silent outbox skip.

## Authoritative contracts

### Employee Weight save/sync

A successful approved Employee Weight edit must:

```text
update only the intended current-tenant Staff employee fields
AND create exactly one canonical employee update in TblLocalOutbox
AND commit both in the same transaction
```

If outbox infrastructure is unavailable, Save must fail and roll back. It must not silently commit the employee row with `OutboxRows=0`.

### Employee Turn Settings no-Draft state

```text
No Draft Policy
= normal Not Configured / No Draft state
```

Opening or reloading `Employee Turn Settings` must not:

```text
show TURN_POLICY_DRAFT_LOAD_FAILED
create a Draft Policy
activate a Policy
mutate the database
```

The first policy remains operator-owned through an explicit `Save Draft` action.

## Mandatory documentation/evidence gate

Read completely before editing:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report072.md
report/report076.md
report/report078.md
```

Read all detailed local evidence from:

```text
EmployeeTurnSettingsSeedV001
EmployeeWeightSaveBoundaryV001
```

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
EVIDENCE_ESCALATION=DIRECT_PHYSICAL_DB_PROOF
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Lane A — read-only physical verification of the successful outbox write

The operator already performed one successful Employee Weight save against the current local target DB.

Inspect the current DB only inside a read-only transaction:

```sql
BEGIN TRANSACTION READ ONLY;
-- inspection only
ROLLBACK;
```

### A1. Correlate the physical save safely

Use sanitized correlation evidence such as:

```text
latest relevant Staff employee UpdatedAt window
latest TblLocalOutbox employee-update window
entity/key equality booleans
transaction/sequence relationship
payload changed-field presence
```

Do not print employee names, raw GUIDs, PINs, exact private values, or connection metadata.

If more than one candidate save exists in the relevant time window and correlation cannot be proven uniquely, do not guess. Return privately:

```text
OUTBOX_CORRELATION_AMBIGUOUS
```

and state the exact safe operator action needed to generate a uniquely identifiable follow-up save.

### A2. Required outbox proof

For the physical save, prove:

```text
Employee row changed = true
Matching TblLocalOutbox rows = exactly 1
Operation = employee update
Entity key matches changed employee = true
Tenant matches = true
Payload contains the physically changed Employee Weight field = true
Payload carries the intended current value = equality true, without printing it
Occurred/created timestamp is consistent with the employee update = true
Transaction/sequence metadata is valid = true
Duplicate matching outbox rows = 0
```

Also report aggregate safe counts:

```text
CandidateEmployeeRows
CandidateOutboxRows
ExactMatches
DuplicateMatches
PayloadFieldPresent
```

### A3. Classify the current result

Use one exact classification:

```text
PHYSICAL_OUTBOX_EXACTLY_ONE_MATCH
PHYSICAL_OUTBOX_MISSING
PHYSICAL_OUTBOX_DUPLICATED
PHYSICAL_OUTBOX_PAYLOAD_FIELD_MISSING
OUTBOX_CORRELATION_AMBIGUOUS
```

### A4. Eliminate silent outbox skipping

Audit the actual production outbox helper and DI registration.

If any active code can still do this:

```text
outbox dependency missing
-> return zero
-> employee Save still commits
```

replace it with fail-closed behavior:

```text
outbox dependency missing
-> explicit EMPLOYEE_WEIGHT_OUTBOX_UNAVAILABLE result
-> no SaveChanges/Commit
-> employee and outbox both unchanged
```

Requirements:

- the required outbox dependency is constructor-injected or otherwise proven available;
- no service-locator fallback silently changes behavior;
- no second outbox implementation is created;
- no duplicate outbox row is written on retry/no-op;
- employee update and outbox remain in the same transaction and same DbContext;
- one successful changed employee = exactly one outbox row.

If the physical DB already has exactly one correct outbox row, preserve it; do not write another row in this prompt.

### A5. Sender/receiver contract check

For the actual changed field, verify privately:

```text
employee entity field
-> employee DTO
-> serialized TblLocalOutbox payload
-> receiver/apply mapping
```

This prompt needs direct proof only for the physically changed Employee Weight field. Record `Turn Enabled=false`, `Queue Order=0`, and clearing Notes as explicit future default-value serialization tests unless they are already directly proven without broadening this task.

## Lane B — close TURN_POLICY_DRAFT_LOAD_FAILED on opening Employee Turn Settings

### B1. Prove the exact current failing call site

Search exact text/result code:

```text
TURN_POLICY_DRAFT_LOAD_FAILED
Unable to load draft turn policy
Draft policy load failed
```

Trace the actual physical chain:

```text
navigation/open command
-> Employee Turn Settings constructor
-> Loaded/open event
-> draft read
-> active read
-> summary population
-> error dialog
```

Return privately:

```text
exact repository-relative file/method/line
exact result-producing method
whether it calls read-only load or get-or-create
exact exception/result behind TURN_POLICY_DRAFT_LOAD_FAILED
whether the failure occurs before or after the window becomes visible
```

Also prove the executing binary/source provenance so a stale build is not mistaken for a source defect:

```text
assembly path
assembly last-write timestamp
source/build label or equivalent
active launch profile
```

Do not expose absolute paths in the GitHub report.

### B2. Canonical read behavior

Create/use one canonical read method with a result shape equivalent to:

```text
TenantResolved
DraftExists
ActiveExists
Status
ResultCode
SafeMessage
```

Required outcomes:

#### No Draft and no Active

```text
Succeeded=True
Status=NotConfigured
DraftExists=False
ActiveExists=False
ResultCode=TURN_POLICY_NOT_CONFIGURED
No modal error
No DB mutation
```

#### Draft exists

```text
load Draft summary
allow Save Draft / Activate according to validation
```

#### Active-only, no Draft

```text
load Active summary
show No Draft separately
no error
```

#### Real DB/schema/query failure

```text
Succeeded=False
structured non-private error
modal allowed only for a real unexpected failure
```

### B3. Remove all open-time get-or-create behavior

Opening, loading, or pressing Reload must never create a policy row.

Only an explicit operator command may create the first Draft:

```text
operator edits setup
-> Save Draft
-> validate
-> create Draft and required child rules transactionally
```

Do not use a fake in-memory policy object that can later be mistaken for persisted state.

### B4. UI state after correction

When the current DB has no Draft Policy, the window must display concepts equivalent to:

```text
Policy: No Draft
Status: Not Configured
Enabled: No
Active: No
```

The setup sections and employee-scoped editors may still be opened according to their independent contracts.

`Activate Policy` must remain disabled until a valid persisted Draft exists.

`Reload` must use the same canonical read method and must not show an error in the no-policy state.

## Tests

### Outbox tests

Add/repair focused tests for:

```text
successful Employee Weight save -> exactly one outbox row
outbox dependency unavailable -> employee update rolls back / does not commit
no-op save -> zero new outbox rows
retry after committed save with no new change -> zero new outbox rows
outbox insert failure -> employee update rolls back
payload includes changed Employee Weight field
receiver applies changed Employee Weight field
```

Use a real disposable Npgsql schema test for the atomic employee + outbox case. Do not write to the operator's current DB.

### Turn Settings load tests

Add/repair focused tests for:

```text
no Draft/no Active -> Not Configured success, no modal
open/load makes zero policy mutations
Reload makes zero policy mutations
Draft exists -> summary loads
Active-only -> Active summary loads and No Draft is normal
real query failure -> structured failure
Employee Weight Edit remains independent
```

Do not broaden the filter to unrelated migration suites.

## Physical retest steps

After build/tests and private proof:

### Outbox

1. Do not make another employee edit initially.
2. Use read-only DB evidence to confirm the already-successful save has exactly one matching outbox row.
3. Only if correlation is ambiguous, perform one operator-approved uniquely identifiable Employee Weight edit and Save once.
4. Confirm safe status reports `OutboxRows=1`.
5. Save again without changes and confirm no new outbox row.

### Employee Turn Settings

1. Rebuild and launch WPF manually.
2. Open Employee Turn Settings with no Draft Policy.
3. Confirm no `TURN_POLICY_DRAFT_LOAD_FAILED` dialog.
4. Confirm `No Draft / Not Configured` status.
5. Click Reload and confirm the same state with no mutation.
6. Open Employee Weight Edit and confirm Staff rows still load.
7. Do not click Save Draft or Activate Policy in this task.
8. Do not test checkout/payment.

## Safety boundaries

- Current DB inspection is read-only.
- Do not automatically create/update/delete current policy or employee/outbox rows.
- Disposable DB writes are allowed only in isolated tests.
- No WPF automation/clicking.
- No checkout/payment test.
- No API token, credential, PIN, employee name, raw identifier, payload private value, or connection metadata in GitHub artifacts.
- No OBM source commit/push; source edits remain local.

## Private handoff requirements

Return privately:

1. Physical outbox classification and safe aggregate evidence.
2. Exact current outbox C# helper/DI behavior before and after.
3. Exact atomic transaction proof.
4. Actual sender/payload/receiver proof for Employee Weight.
5. Exact TURN_POLICY_DRAFT_LOAD_FAILED call site and root cause.
6. Complete before/after C# blocks for the Turn Settings open/load method.
7. Zero-mutation proof for no-policy open/reload.
8. Real disposable Npgsql test output.
9. Build/test counts.
10. Exact physical retest steps.

## Public report

Create and push:

```text
report/report079.md
```

The public report must be ultra-minimal and may contain only:

```text
verdict
physical outbox exact-match yes/no/ambiguous
automatic silent-outbox skip removed yes/no
no-Draft Turn Settings load corrected yes/no
zero policy mutation on open/reload yes/no
real-schema test yes/no
build/test counts
current DB mutated by Codex no
checkout tested no
one aggregate evidence SHA-256
```

Do not include source paths, class/method/table/column names, counts tied to private records, payload content, architecture details, or DB identifiers.

## Valid verdicts

```text
OBM_POS_EMPLOYEE_WEIGHT_OUTBOX_AND_NO_DRAFT_TURN_SETTINGS_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_PHYSICAL_OUTBOX_CORRELATION_AMBIGUOUS
```

```text
BLOCKED_EMPLOYEE_WEIGHT_OUTBOX_CONTRACT
```

```text
BLOCKED_TURN_SETTINGS_NO_DRAFT_LOAD_ROOT_CAUSE_UNPROVEN
```

```text
BLOCKED_PROMPT079_BUILD_OR_TEST
```
