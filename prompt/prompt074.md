# Prompt 074 — Remove stale Draft Policy prerequisite from Employee Weight Edit launch

## Operator evidence

The operator rebuilt and physically tested the current WPF source after prompt073.

In `Employee Turn Settings`, `Employee Weight` is checked as Included. Clicking `Edit` does **not** open `Employee Weight Settings`. Instead, the parent window shows:

```text
Unable to load draft turn policy. Result code:
TURN_POLICY_DRAFT_LOAD_FAILED
```

This proves the Employee Weight child-window grid correction from prompt073 is not yet reachable through the real parent launch path.

The failure occurs before the child window opens.

## Authoritative contracts already proven

From prompt072:

```text
No Draft Policy is a normal Not Configured / No Draft state.
Opening or reading setup must not create a policy row.
The first policy is operator-owned through Employee Turn Settings.
```

From prompt073:

```text
Employee Weight Settings employee rows are employee-scoped.
Displaying and saving employee weight rows does not require an existing Draft Policy.
The child window resolves the current local tenant and loads eligible employees independently.
```

Therefore the parent `Edit` command must not use Draft Policy existence as a prerequisite for opening Employee Weight Settings.

## Scope

Fix only the parent-to-child Employee Weight Edit launch path and its no-draft state handling.

Do not:

- test checkout/payment;
- activate a Turn policy;
- automatically create a Draft policy;
- mutate the current database automatically;
- broaden into Price, Service Category, or Customer/Booking editors except for a brief consistency audit;
- commit or push OBM source.

## Documentation gate

Read before editing:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report072.md
report/report073.md
```

Also read the complete local-only evidence artifacts:

```text
EmployeeTurnSettingsSeedV001
EmployeeWeightSettingsV001
```

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase A — trace the actual failing launch chain

Map the exact active path:

```text
Employee Turn Settings
-> Employee Weight Edit button/command
-> parent preflight
-> draft-policy lookup
-> result-code mapping
-> message box
-> Employee Weight Settings constructor/show
```

Record locally:

- repository-relative files and methods;
- whether the Edit handler calls load-draft, get-or-create-draft, or another wrapper;
- exact result returned when no draft exists;
- where `TURN_POLICY_DRAFT_LOAD_FAILED` is produced;
- whether the no-draft result is collapsed into a generic failure;
- whether child-window creation is skipped solely because no Draft exists;
- whether any database row is attempted or created before the error.

Classify the root cause using the narrowest proven value:

```text
PARENT_EDIT_REQUIRES_DRAFT_POLICY
NO_DRAFT_RESULT_MAPPED_TO_LOAD_FAILURE
STALE_GET_OR_CREATE_DRAFT_PREFLIGHT
PARENT_USES_WRONG_DRAFT_API
PARENT_TENANT_CONTEXT_FAILURE
ACTUAL_DRAFT_QUERY_FAILURE
OTHER_PROVEN_CAUSE
```

Do not classify the child employee grid again unless new evidence contradicts prompt073.

## Phase B — establish the correct launch contract

The Employee Weight Edit behavior must be:

```text
Click Edit
-> resolve current local tenant/context
-> optionally read Draft/Active summary without creating rows
-> no Draft is accepted as normal
-> open Employee Weight Settings
-> child window auto-loads eligible employee rows
```

### No Draft exists

Expected behavior:

```text
Employee Weight Settings opens.
Employee rows load.
No policy row is created.
The child may show a non-blocking status such as:
"No Draft policy. Employee settings can be prepared now; they affect TurnEngine only after a policy including Employee Weight is saved and activated."
```

Do not show an error modal merely because no Draft exists.

### Draft exists

Expected behavior:

```text
Employee Weight Settings opens.
Employee rows load.
Existing employee-scoped values display normally.
```

### Active policy exists but no Draft exists

Because employee weight rows are employee-scoped and not policy-versioned, the child window must still open unless active source proves a specific safety restriction. Report the exact behavior chosen and why.

### Genuine failure

Only block opening when there is a real prerequisite failure, such as:

- tenant/local context cannot be resolved;
- database is unavailable;
- schema/query/mapping fails;
- child-window construction fails.

The error must use a specific safe result code and message, not `TURN_POLICY_DRAFT_LOAD_FAILED` for a normal no-draft condition.

## Phase C — preserve parent unsaved state correctly

The parent UI may show `Employee Weight = Included` before a Draft Policy has been saved.

Audit whether that checkbox state is:

- only in-memory UI state;
- loaded from a Draft;
- loaded from an Active policy;
- inferred from another source.

Required behavior:

- opening Employee Weight Settings must not silently save or create a Draft;
- checking `Employee Weight` may remain an unsaved parent setting until `Save Draft` is explicitly clicked;
- editing employee-scoped rows must not falsely imply that the parent policy inclusion flag has been persisted;
- returning from the child must preserve the parent dirty state according to the existing UI pattern;
- child Save must save only proven employee-scoped fields, not the parent policy header/inclusion flag.

Add a clear non-blocking status where necessary so the operator understands:

```text
Employee values can be prepared now.
TurnEngine will use them only when Employee Weight is included in an active policy.
```

## Phase D — use one canonical child launch path

Avoid duplicate window-launch implementations.

Required structure:

```text
OpenEmployeeWeightSettingsAsync
-> validate local context
-> collect optional policy summary
-> construct child once
-> show child once
-> refresh parent summary if necessary without creating rows
```

The real Edit button and any keyboard/command route must call the same canonical launch method.

Prevent:

- duplicate modal windows;
- multiple automatic loads;
- parent reload recursively reopening the child;
- no-draft state being reinterpreted differently in separate handlers.

## Phase E — consistency audit of other weight Edit buttons

Briefly inspect the parent launch guards for:

```text
Price Weight Edit
Service Category Weight Edit, if present
Customer / Booking Weight Edit
```

Do not rebuild those editors in this task.

Report locally whether they repeat the same stale Draft prerequisite.

If a shared parent helper is the proven root cause, correct it minimally only when the change is safe for all editors and add tests for each affected route. Otherwise keep the source change limited to Employee Weight Edit and create follow-up findings for the others.

## Phase F — tests

Add focused tests covering at least:

1. Employee Weight Edit with no Draft opens the child window contract;
2. no Draft does not return `TURN_POLICY_DRAFT_LOAD_FAILED`;
3. no Draft causes zero policy-row creation/mutation;
4. child employee loader is invoked after launch;
5. existing Draft still opens normally;
6. Active-only/no-Draft behavior matches the proven contract;
7. parent unsaved Included state is not automatically persisted by opening Edit;
8. child Save does not save the parent policy inclusion flag;
9. unresolved tenant blocks with a specific tenant-context result;
10. genuine draft query failure is distinguished from no-draft state;
11. repeated Edit does not create duplicate child windows or duplicate load calls;
12. API unavailable/401 does not block the local editor.

Run a focused Turn Settings / Employee Weight parent-launch test filter and build WPF.

Do not use broad filters that include unrelated migration suites.

## Physical retest steps

Return these steps in the private operator handoff:

1. Rebuild and launch WPF manually.
2. Open Employee Turn Settings with the local database still having no Draft policy.
3. Check Employee Weight if needed, without clicking Save Draft.
4. Click Employee Weight `Edit`.
5. Confirm no `TURN_POLICY_DRAFT_LOAD_FAILED` dialog appears.
6. Confirm Employee Weight Settings opens.
7. Confirm eligible employee rows load automatically.
8. Close the child without saving and confirm no Draft policy was created.
9. Reopen the child and confirm it still loads.
10. Optionally edit and Save one approved employee-scoped value, then reopen and verify persistence.
11. Do not test checkout/payment in this task.

## Mutation and safety boundaries

- No automated INSERT/UPDATE/DELETE against the current DB.
- No automated WPF launch or click.
- No policy activation.
- No checkout/payment test.
- No OBM source commit/push.
- No employee names, identifiers, PINs, database names, paths, schema metadata, or architecture details in the public GitHub report.

## Evidence

Create a new versioned local evidence artifact, for example:

```text
EmployeeWeightEditLaunchV001
```

Include sanitized proof for:

- failing parent launch chain;
- no-draft result classification;
- corrected launch flow;
- zero policy mutation on open;
- child load invocation;
- focused tests/build.

## Reporting protocol

Return a detailed private operator handoff containing:

- exact root-cause classification;
- exact parent handler/helper corrected;
- no-draft launch behavior;
- active-only behavior;
- parent dirty-state behavior;
- other Edit-button consistency findings;
- build/test results;
- physical retest steps.

Create `report/report074.md` as an ultra-minimal public summary only. It may include only:

- verdict;
- no-draft Employee Weight Edit launch corrected yes/no;
- child employee load reachable yes/no;
- zero policy mutation on open yes/no;
- build/test counts;
- current DB mutated no;
- checkout tested no;
- one aggregate local evidence SHA-256.

Do not include source paths, symbols, table/column names, counts, database names, or internal call-chain details.

Expected success verdict:

```text
OBM_POS_EMPLOYEE_WEIGHT_EDIT_NO_DRAFT_READY_FOR_PHYSICAL_RETEST
```

If blocked, use the narrowest proven verdict in the private handoff.
