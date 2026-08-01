# Prompt 073 — Rebuild Employee Weight Settings load/edit/save flow

## Operator direction

The operator physically opened `Employee Turn Settings`, enabled `Employee Weight`, and clicked `Edit`.

The `Employee Weight Settings` window opens, but the employee grid is empty. The operator confirms this screen previously loaded employees and wants this section corrected before continuing with the other TurnEngine weights.

Visible columns/actions in the current window include concepts equivalent to:

- Employee Name
- Login Status
- Current Legacy Factor
- Employee Weight
- Turn Enabled
- Queue Order
- Notes
- Load
- Save
- Reset From Employee Factor
- Close

This task is only for the Employee Weight configuration workflow. Do not test checkout/payment.

## Canonical business behavior

1. Opening Employee Weight Settings must load the current tenant's eligible employee rows even when no Turn policy draft exists.
2. Draft-policy absence is a normal `Not Configured / No Draft` state and must not cause an empty employee list.
3. Employee data is the row source. Policy state only controls whether Employee Weight is included in TurnEngine; it must not be required merely to display employees.
4. Source/runtime behavior must be proven before changing persistence semantics.
5. No automated database mutation is allowed. Database changes may occur only later through explicit operator UI actions such as `Save`.

## Documentation gate

Read before editing:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report071.md
report/report072.md
```

Also read the complete local evidence artifacts:

```text
FourWeightPolicyV001
EmployeeTurnSettingsSeedV001
```

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase A — trace the exact empty-grid cause

Locate the full active chain:

```text
Employee Turn Settings
-> Employee Weight Edit command
-> Employee Weight Settings window construction
-> Loaded/open event
-> tenant/local context resolution
-> employee query
-> eligibility/status filters
-> projection/view-model rows
-> ItemsSource/DataContext
-> grid rendered count
```

Prove the exact root cause and classify it as one of:

```text
LOAD_NOT_CALLED_ON_OPEN
LOAD_REQUIRES_DRAFT_POLICY
TENANT_CONTEXT_NOT_RESOLVED
WRONG_DATABASE_CONTEXT
EMPLOYEE_FILTER_EXCLUDES_ALL
EMPLOYEE_TYPE_FILTER_WRONG
LOGIN_STATUS_FILTER_WRONG
MAPPING_FAILURE
ITEMSSOURCE_BINDING_FAILURE
SCHEMA_MISMATCH
OTHER_PROVEN_CAUSE
```

Record locally, without exposing employee names or identifiers:

- target employee total count;
- eligible employee count;
- rows returned by query;
- rows projected to the UI collection;
- final grid item count;
- every predicate used by the employee query.

Inspect the target database and the read-only reference database only inside read-only transactions. Do not print raw employee rows, names, PINs, GUIDs, or credentials.

## Phase B — prove the Employee Weight data contract

Audit active source/model/schema and determine the exact owner of each displayed field:

| UI concept | Source model/property | DB table/column | Editable | Runtime use | Legacy/current |
|---|---|---|---:|---|---|
| Employee Name | | | no | display only | |
| Login Status | | | usually no | eligibility/status | |
| Current Legacy Factor | | | no or proven behavior | reference/reset source | legacy |
| Employee Weight | | | yes | production TurnEngine | current |
| Turn Enabled | | | yes | employee participation | current |
| Queue Order | | | yes if active | queue/turn ordering | current or runtime |
| Notes | | | yes if persisted | operator metadata | |

Do not assume column names from the UI. Prove them from active source and schema.

Also prove:

1. Whether Employee Weight is stored directly on the employee row or in a policy child table.
2. Whether values are policy-versioned or global per employee.
3. Whether saving employee weight requires an existing draft policy.
4. Whether `Reset From Employee Factor` is only an in-memory copy or immediately persists.
5. Whether inactive/non-login/non-staff employees should be shown, hidden, or read-only.
6. Which employee types are legitimate TurnEngine technicians.
7. Whether queue order must be unique, contiguous, optional, or independent from employee weight.

## Phase C — corrected load behavior

Implement the smallest proven correction.

Required behavior:

```text
Open window
-> resolve current local tenant
-> load eligible employees automatically
-> populate grid
-> show structured status/count line
```

The `Load` button must explicitly reload using the same canonical load method.

No draft policy:

```text
employee rows still load
policy-related state displays No Draft / Not Configured where relevant
no row is created merely by opening or loading
```

Empty legitimate tenant:

```text
show a clear empty-state message such as
"No eligible employees are available for Employee Weight setup."
```

Do not leave a blank grid without explanation.

Add a safe status/result contract containing concepts equivalent to:

```text
TenantResolved
EmployeeRowsFound
EligibleRows
DisplayedRows
DraftPolicyResolved
ResultCode
SafeMessage
```

Do not display raw tenant IDs or private identifiers.

## Phase D — edit and save behavior

Use the proven persistence contract from Phase B.

Required UI behavior:

- rows are editable only where the active contract permits;
- unsaved changes remain in memory until `Save`;
- `Close` with dirty changes follows the application's existing confirmation pattern;
- validation errors are row-specific and do not partially save;
- `Save` uses one transaction;
- only changed employee rows are updated;
- existing local outbox/sync path is reused if these employee fields are already synchronized;
- no duplicate outbox writes for no-op saves;
- reload after successful save uses the canonical load method;
- API availability/401 must not block local setup.

Do not invent a new policy-child table or generic settings framework.

### Reset From Employee Factor

Audit the existing intent and preserve it safely.

Expected behavior unless source proves otherwise:

```text
operator clicks Reset From Employee Factor
-> copy the current legacy factor into the editable Employee Weight field
-> do not save automatically
-> mark affected rows dirty
-> require explicit Save
```

If the legacy value is missing/invalid, provide a row-level safe message and do not invent a value.

## Phase E — eligibility rules

Codify and test the proven eligibility contract.

At minimum investigate:

- active vs inactive employee;
- Staff/technician vs Owner/Admin/SubAdmin/non-staff;
- login-enabled vs login-disabled;
- deleted/hidden/terminated status if present;
- tenant scope;
- participation/TurnEnabled state.

Do not reuse management visibility rules blindly if TurnEngine uses a different employee eligibility contract.

The final report must state the exact operator-visible behavior, for example:

```text
active technicians shown and editable
inactive technicians shown read-only when requested
non-staff excluded
```

Only state what source/reference evidence proves.

## Phase F — no-seed rule

Preserve the future-install decision from prompt072:

```text
Do not seed employee-specific TurnEngine rows.
Do not seed fake employees.
Do not create Employee Weight values before employees exist.
```

A clean installation with zero employees must open this window successfully and show the empty-state message.

## Tests

Add focused tests covering at least:

1. window load invokes the canonical employee loader;
2. no draft policy still returns employee rows;
3. correct tenant filtering;
4. eligibility filters;
5. empty employee tenant gives a clear empty state;
6. UI projection/binding collection count;
7. Load button reload;
8. valid edit/save transaction;
9. validation prevents partial saves;
10. no-op save creates no mutation/outbox;
11. Reset From Employee Factor changes UI state but does not auto-save;
12. local setup works when API is unavailable/401.

Run focused Employee Weight/Turn Settings tests and build the WPF project.

Do not broaden the test filter to unrelated migration suites.

## Physical retest steps for operator

Provide these exact steps in the local operator handoff:

1. Rebuild and launch WPF manually.
2. Open Employee Turn Settings.
3. Enable Employee Weight if needed.
4. Click Edit.
5. Verify employee rows load automatically.
6. Verify the status line reports nonzero displayed rows when eligible employees exist.
7. Click Load and verify the same rows reload without duplicates.
8. Change one Employee Weight value without saving; close/reopen behavior must follow the dirty-change rule.
9. Use Reset From Employee Factor and verify it does not persist until Save.
10. Save one approved change.
11. Reload and verify persistence.
12. Confirm no API connection is required.
13. Do not run checkout/payment in this task.

## Mutation and safety boundaries

- No automated INSERT/UPDATE/DELETE against the current DB.
- No automated clicking or WPF launch.
- No checkout/payment test.
- No employee names, PINs, raw identifiers, connection strings, or business rows in GitHub artifacts.
- No OBM source commit/push.
- Source changes remain local.

## Evidence

Create a new versioned local evidence folder, for example:

```text
EmployeeWeightSettingsV001
```

Include sanitized evidence for:

- load call chain;
- employee eligibility/filter inventory;
- DB count proof;
- binding flow;
- save/reset transaction behavior;
- focused tests/build.

## Reporting protocol

Return a detailed operator handoff directly in the private Codex conversation, including:

- exact root-cause classification;
- exact eligibility contract;
- exact field ownership/persistence model;
- whether a draft policy is required for display and save;
- source correction summary;
- build/test results;
- physical retest steps.

Create `report/report073.md` as an ultra-minimal public summary only. It may include only:

- verdict;
- employee grid load corrected yes/no;
- no-draft display supported yes/no;
- save/reset flow corrected yes/no;
- build/test counts;
- current DB mutated no;
- checkout tested no;
- one aggregate local evidence SHA-256.

Do not include source paths, symbols, schema/table/column names, employee counts, employee metadata, database names, or architecture details in the GitHub report.

Expected success verdict:

```text
OBM_POS_EMPLOYEE_WEIGHT_SETTINGS_READY_FOR_PHYSICAL_RETEST
```

If blocked, use the narrowest proven verdict and state the blocker in the private operator handoff.