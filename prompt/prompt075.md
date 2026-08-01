# Prompt 075 — Make Employee Weight Edit independent from Included checkbox and policy summary

## Operator evidence

The operator physically tested the current WPF source after prompt074.

In `Employee Turn Settings`, clicking `Employee Weight -> Edit` still fails before the child window opens. The current dialog is:

```text
Unable to open Employee Weight Settings. Result code:
EMPLOYEE_WEIGHT_EDIT_POLICY_SUMMARY_FAILED
```

The operator also clarified the authoritative UI contract:

```text
The Edit button must not depend on the Employee Weight Included checkbox.
```

This means the editor must open whether Employee Weight is checked or unchecked, whether the checkbox change is saved or unsaved, and whether Draft/Active policy summary loading succeeds or fails.

## Canonical separation of responsibilities

### Parent checkbox

`Employee Weight Included` belongs to the parent Turn Policy draft/active configuration.

- Checking/unchecking it changes only parent UI dirty state until `Save Draft`.
- It controls whether Employee Weight participates in a future Draft/Active policy.
- It must not control access to employee-scoped setup.

### Child editor

`Employee Weight Settings` manages employee-scoped values:

- Employee Weight
- Turn Enabled
- Queue Order
- Notes

The child editor:

- resolves the current local tenant;
- loads eligible employees independently;
- does not require Draft Policy;
- does not require Active Policy;
- does not require parent Policy Summary;
- does not persist or toggle the parent Included checkbox;
- only persists approved employee-scoped edits when the operator clicks `Save` inside the child.

## Exact objective

Remove every stale policy-summary or checkbox prerequisite from the real Employee Weight Edit launch path.

Required final launch contract:

```text
Click Employee Weight Edit
-> open Employee Weight Settings directly through one canonical method
-> child resolves tenant and loads employees
```

Policy summary may be read by the parent for display elsewhere, but its absence or failure must not block this editor.

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
```

Also read local-only evidence:

```text
EmployeeTurnSettingsSeedV001
EmployeeWeightSettingsV001
EmployeeWeightEditNoDraftV001
```

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase A — trace the exact new failure

Locate the real path that emits:

```text
EMPLOYEE_WEIGHT_EDIT_POLICY_SUMMARY_FAILED
```

Map:

```text
Edit click
-> parent command/event
-> checkbox/inclusion checks
-> policy summary read
-> result classification
-> dialog creation
-> child constructor/open call
```

Prove exactly why policy summary failure still blocks the child after prompt074.

Classify the root cause narrowly, for example:

```text
PARENT_EDIT_STILL_REQUIRES_POLICY_SUMMARY
SHARED_PRECHECK_RETURNS_FAILURE
CHECKBOX_GATE_STILL_PRESENT
STALE_LAUNCH_HELPER_USED
DUPLICATE_EDIT_HANDLER
OTHER_PROVEN_CAUSE
```

## Phase B — delete the wrong prerequisite, not the diagnostics

Implement the smallest correct change.

### Required behavior matrix

| Included checkbox | Draft policy | Active policy | Policy summary read | Edit result |
|---|---|---|---|---|
| Unchecked | none | none | success/no policy | child opens |
| Checked but unsaved | none | none | success/no policy | child opens |
| Unchecked | exists | any | success | child opens |
| Checked | exists | any | success | child opens |
| Either | any | any | summary read fails | child still opens |

A policy-summary read failure may be logged or shown as a non-blocking parent status, but must not prevent the employee editor from opening.

### Only legitimate blockers

The child launch may fail only for a proven child-specific reason such as:

- local tenant cannot be resolved;
- child window construction fails;
- employee loader throws a real local data/schema error.

Use Employee Weight-specific result codes for these cases. Do not translate a parent policy-summary failure into an employee-editor failure.

## Phase C — one canonical open method

There must be one canonical method such as conceptually:

```text
OpenEmployeeWeightSettingsAsync
```

All active Employee Weight Edit entry points must call it.

That method must:

1. avoid checking the Included checkbox;
2. avoid requiring Draft or Active policy;
3. avoid requiring policy summary;
4. open only one child instance per operator click;
5. pass only child-relevant context;
6. allow the child to use its own canonical tenant resolver and employee loader;
7. not mutate DB merely by opening.

Remove or bypass duplicate/stale launch helpers that still contain policy gates.

## Phase D — preserve parent dirty-state semantics

Prove with tests that:

```text
Checkbox unchecked
-> Edit child
-> child closes
-> parent remains unchecked

Checkbox checked but unsaved
-> Edit child
-> child saves employee-scoped value
-> parent remains checked and dirty
-> no Draft policy is created until Save Draft
```

Opening or saving the child must never auto-save parent policy inclusion.

## Phase E — child behavior remains intact

Do not regress prompt073 behavior:

- eligible employee rows load automatically;
- no Draft Policy is supported;
- Load button reloads without duplicates;
- Save persists dirty employee-scoped rows only;
- Reset remains in-memory until Save;
- API availability/401 does not block local setup.

## Phase F — focused consistency audit

Audit active XAML/code-behind bindings for the Employee Weight `Edit` button:

- IsEnabled must not bind to Included checkbox state.
- Command CanExecute must not depend on Included.
- Visibility must not depend on Included.
- Click handler must be active in checked and unchecked states.

Do not broaden implementation to Price Weight or Customer/Booking Weight in this task. Record only whether those editors remain policy-scoped and leave them for later prompts.

## Tests

Add focused tests for at least:

1. Edit enabled and callable when checkbox is unchecked.
2. Edit enabled and callable when checkbox is checked.
3. Checked-but-unsaved parent state does not create Draft.
4. No Draft/no Active opens child.
5. Active-only/no Draft opens child.
6. Policy summary success opens child.
7. Policy summary missing/no policy opens child.
8. Policy summary service failure still opens child.
9. Child opens once per click.
10. Child save does not persist parent Included state.
11. Parent dirty state survives child open/close.
12. Existing child employee-load tests remain passing.
13. No DB mutation occurs on open.
14. API unavailable/401 does not block editor.

Run focused Turn Settings/Employee Weight tests only. Do not use broad migration filters.

Build the WPF project.

## Physical operator retest

Return these steps in the private handoff:

1. Rebuild and launch WPF manually.
2. Open Employee Turn Settings.
3. Leave Employee Weight unchecked.
4. Click Edit.
5. Confirm Employee Weight Settings opens and employees load.
6. Close the child without saving.
7. Check Employee Weight but do not click Save Draft.
8. Click Edit again.
9. Confirm the child opens and employees load.
10. Confirm no `EMPLOYEE_WEIGHT_EDIT_POLICY_SUMMARY_FAILED` dialog.
11. Confirm no Draft policy was created merely by opening the editor.
12. Optionally save one approved employee-scoped value and verify parent Included remains only an unsaved parent change.
13. Do not test checkout/payment.

## Mutation and safety boundaries

- No automated DB mutation.
- No automated WPF launch/clicking.
- No checkout/payment test.
- No OBM source commit/push.
- No secrets, employee names, PINs, identifiers, connection strings, database names, source paths, or architecture metadata in the public GitHub report.

## Evidence and reporting

Create a new versioned local evidence artifact, for example:

```text
EmployeeWeightEditIndependentV001
```

Return a detailed private operator handoff containing:

- exact root cause;
- final launch matrix proof;
- confirmation Edit is independent from Included;
- confirmation policy summary failure is non-blocking;
- build/test counts;
- physical retest steps.

Create `report/report075.md` as an ultra-minimal public summary containing only:

- verdict;
- Edit independent from Included yes/no;
- policy summary failure non-blocking yes/no;
- child employee load reachable yes/no;
- zero policy mutation on open yes/no;
- build/test counts;
- current DB mutated no;
- checkout tested no;
- one aggregate evidence SHA-256.

Expected success verdict:

```text
OBM_POS_EMPLOYEE_WEIGHT_EDIT_INDEPENDENT_READY_FOR_PHYSICAL_RETEST
```
