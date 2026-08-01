# Prompt 076 — Fix Employee Weight child loader and load current-tenant Staff employees

## Operator evidence

The operator physically tested the current WPF source after prompt075.

The parent launch problem is closed: `Employee Weight Settings` now opens independently from the Included checkbox and policy summary.

The child window itself now reports:

```text
Load Failed (EMPLOYEE_WEIGHT_LOAD_FAILED)
Unable to load Employee Weight rows. Result code:
EMPLOYEE_WEIGHT_LOAD_FAILED
```

The grid remains empty.

This proves the failure is now inside the child employee-load path after the window opens.

## Authoritative employee inclusion rule

The operator explicitly requires:

```text
Load only employees for the current tenant whose EmployeePermission is Staff.
```

Interpret and implement this using the actual active source enum/storage contract. Do not guess whether the persisted representation is a string, integer, enum conversion, or another mapped value.

Required eligibility baseline:

```text
Current tenant
AND EmployeePermission == Staff
```

The following must NOT exclude a Staff employee from this setup grid merely by themselves:

- Included checkbox checked/unchecked;
- Draft Policy missing/present;
- Active Policy missing/present;
- Policy Summary success/failure;
- login enabled/disabled status;
- employee PIN/login-number presence;
- current Turn Enabled value;
- missing legacy factor;
- missing Employee Weight value;
- API availability or 401.

Audit soft-deleted, virtual/non-real, terminated, archived, or equivalent safety-state fields. Preserve an additional exclusion only when active source/schema/reference evidence proves that such rows are not real employee records. Report every additional predicate privately. Do not invent broad `working`, `login-enabled`, or management-visibility filters.

`Login Status` is a displayed status column, not the authoritative Staff eligibility predicate.

## Scope

Fix only the Employee Weight child load/query/projection/binding path and its exact Staff eligibility rule.

Do not:

- test checkout/payment;
- create or activate a Turn Policy;
- require a Draft or Active Policy;
- automatically mutate the current database;
- change the parent Included checkbox behavior;
- broaden into Price, Service Category, or Customer/Booking editors;
- commit or push OBM source.

## Mandatory documentation gate

Read before editing:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report072.md
report/report073.md
report/report074.md
report/report075.md
```

Also read the local-only evidence artifacts from prompts 072–075.

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase A — capture the exact failing stage and inner cause

Trace the actual child path:

```text
Employee Weight Settings opened
-> Loaded/open event
-> canonical Load command
-> current tenant resolution
-> DbContext creation
-> Staff employee query
-> optional setting-field projection
-> row ViewModel construction
-> ObservableCollection update
-> DataGrid ItemsSource
```

Do not stop at the public wrapper code `EMPLOYEE_WEIGHT_LOAD_FAILED`.

Capture locally and classify the first real failure as one of:

```text
TENANT_RESOLUTION_FAILED
DB_CONTEXT_OR_CONNECTION_FAILED
STAFF_PERMISSION_PREDICATE_MISMATCH
EMPLOYEE_PERMISSION_ENUM_MAPPING_FAILED
EMPLOYEE_PERMISSION_SCHEMA_MISMATCH
QUERY_TRANSLATION_FAILED
OPTIONAL_FIELD_PROJECTION_FAILED
LEGACY_FACTOR_PROJECTION_FAILED
EMPLOYEE_WEIGHT_FIELD_PROJECTION_FAILED
NULLABLE_VALUE_MAPPING_FAILED
ITEMSSOURCE_UPDATE_FAILED
OTHER_PROVEN_CAUSE
```

Record locally:

- failing stage ID;
- exception type;
- sanitized inner exception/message;
- repository-relative method;
- query predicate;
- whether failure occurs before or after SQL execution;
- rows returned before projection, if any;
- projected row count;
- final grid count.

Do not expose employee names, PINs, identifiers, connection strings, database names, or business rows in GitHub.

## Phase B — read-only database proof

Inspect the current target database and reference database using read-only transactions only:

```sql
BEGIN TRANSACTION READ ONLY;
-- inspection only
ROLLBACK;
```

Prove privately:

1. The real table/column and type representing `EmployeePermission`.
2. The real persisted representation corresponding to `Staff`.
3. Current-tenant employee count.
4. Current-tenant Staff count.
5. Counts after every additional proven safety-state predicate.
6. Whether nullable legacy factor, Employee Weight, Turn Enabled, Queue Order, or Notes values exist.
7. Whether the target schema has every column expected by the active model.
8. Whether the same canonical query succeeds against the reference schema.

Use counts only. Do not print raw employee rows or names.

## Phase C — canonical Staff loader

Create or retain exactly one canonical load method used by:

```text
window auto-load
Load button
post-save reload
```

Required query contract:

```text
resolve current local tenant
-> query employee rows for that tenant
-> require EmployeePermission == Staff using proven model mapping
-> apply only proven deleted/virtual safety exclusions
-> order deterministically
-> project display/edit fields
-> populate the grid
```

Do not use:

```text
Draft Policy ID
Active Policy ID
parent Included checkbox
policy summary
login-enabled predicate
legacy factor presence
Turn Enabled == true
Employee Weight non-null
```

as prerequisites for loading a Staff row.

### Defensive projection rule

Optional configuration/display values must not make the whole employee list fail merely because one value is null.

Examples include, only where the schema proves they are optional:

```text
legacy factor
employee weight
queue order
notes
login-display status
```

Use explicit nullable/view-state handling. Do not silently invent persisted values. A display fallback may be shown as blank, Not Set, or a proven UI default, but it must remain unsaved until the operator edits and clicks Save.

If a required schema column is physically absent, report `SCHEMA_MISMATCH` rather than hiding it with a fake value.

## Phase D — clear load result contract

Replace the generic-only failure surface with a structured child-load result containing concepts equivalent to:

```text
StageId
TenantResolved
StaffRowsFound
DisplayedRows
ResultCode
SafeMessage
```

Successful load:

```text
ResultCode=EMPLOYEE_WEIGHT_LOAD_READY
```

Legitimate zero Staff rows:

```text
ResultCode=EMPLOYEE_WEIGHT_NO_STAFF_EMPLOYEES
SafeMessage=No Staff employees are available for Employee Weight setup.
```

A legitimate zero-row state must not show an error modal.

A real failure may show one safe modal, but the private handoff must include the exact sanitized inner cause and stage.

Do not show raw IDs, tenant values, paths, SQL, or credentials in the UI.

## Phase E — UI behavior

Required physical behavior:

```text
Open Employee Weight Settings
-> Staff rows load automatically
-> grid is populated
```

The Load button must:

```text
call the same canonical method
clear and replace rows once
not duplicate rows
preserve no unsaved edits without following the existing dirty-change confirmation pattern
```

The grid must include only current-tenant Staff employees under the proven safety exclusions.

Non-Staff values such as Owner, Admin, SubAdmin, Manager, or other permissions must not appear even if they can log in.

Staff employees must not be excluded solely because Login Status is disabled or because Employee Weight is not yet configured.

## Phase F — preserve save/reset boundaries

Do not redesign the save/reset flow unless the load fix reveals a directly related defect.

Preserve:

```text
Reset From Employee Factor = in-memory only until Save
Save = validate all dirty rows, one transaction, dirty rows only
no-op Save = no mutation/outbox
API unavailable/401 = local setup still works
```

This prompt itself must perform no current-DB mutation and no automated UI clicking.

## Tests

Add focused tests covering at least:

1. current-tenant Staff employee is included;
2. other-tenant Staff employee is excluded;
3. current-tenant non-Staff employee is excluded;
4. login-disabled Staff employee is still included;
5. no Draft/Active Policy still loads Staff employees;
6. Included checkbox checked/unchecked does not change child query;
7. null optional legacy factor does not fail the entire load;
8. null optional current Employee Weight does not fail the entire load;
9. proven deleted/virtual exclusion behavior;
10. permission enum/storage mapping;
11. auto-load and Load button call the same canonical loader;
12. reload does not duplicate rows;
13. legitimate zero Staff rows returns clear empty state, not failure;
14. real query/projection failure returns stage-specific result;
15. API unavailable/401 does not block local load.

Run only focused Employee Weight/Staff loader tests and WPF build. Do not include broad migration filters.

## Physical retest steps

Return these in the private operator handoff:

1. Rebuild and launch WPF manually.
2. Open Employee Turn Settings.
3. Leave Employee Weight unchecked and click Edit.
4. Confirm the child opens and loads current-tenant Staff employees.
5. Confirm no non-Staff employee appears.
6. Confirm a Staff employee may appear even when Login Status is disabled.
7. Close the child.
8. Check Employee Weight without Save Draft and click Edit again.
9. Confirm the same Staff list loads.
10. Click Load and confirm no duplicate rows.
11. Do not Save employee values during the first load-only retest.
12. Do not test checkout/payment.

## Evidence and reporting

Create a versioned local evidence artifact, for example:

```text
EmployeeWeightStaffLoaderV001
```

Include sanitized local evidence for:

- failing-stage/inner-cause proof;
- permission mapping proof;
- read-only count matrix;
- query and projection flow;
- binding counts;
- focused tests/build.

Return a detailed private handoff containing:

- exact root cause;
- exact persisted Staff mapping;
- exact final query predicates;
- every additional safety exclusion;
- read-only counts without employee identities;
- source correction summary;
- build/test results;
- physical retest steps.

Create `report/report076.md` as an ultra-minimal public summary only. It may contain only:

- verdict;
- Staff-only child load corrected yes/no;
- no-policy load supported yes/no;
- optional-null projection safe yes/no;
- build/test counts;
- current DB mutated no;
- checkout tested no;
- one aggregate local evidence SHA-256.

Do not include source paths, symbols, schema/table/column names, permission representations, employee counts, database names, or architecture details in the public report.

Expected success verdict:

```text
OBM_POS_EMPLOYEE_WEIGHT_STAFF_LOADER_READY_FOR_PHYSICAL_RETEST
```

If blocked, use the narrowest proven verdict and provide the exact blocker privately.
