# Prompt 072 — Audit Employee Turn Settings dependencies, correct draft loading, and prepare the future-install seed

## Operator direction — authoritative scope

The operator has physically opened the existing `Employee Turn Settings` window and wants this window to become the canonical place for configuring TurnEngine production weights.

The operator is deliberately postponing checkout retesting. This task must focus only on:

1. auditing this setup window and its complete local database dependencies;
2. identifying why the window currently shows `Draft policy load failed` / remains in a loading state;
3. listing every table required to load, save, activate, preview, and run the four production weights;
4. correcting the setup/load path where source changes are justified;
5. preparing an idempotent minimal seed for future clean installations;
6. leaving the current local database unchanged unless a later prompt explicitly authorizes mutation.

Do **not** run checkout/payment in this task.

## Physical UI evidence

The current window visibly defines the four operator-facing production dimensions as:

1. `Employee Weight` — optional.
2. `Service Category Weight` — always active.
3. `Price Weight` — optional.
4. `Customer / Booking Weight` — optional.

The window also shows this production model:

```text
Category Adjusted Amount = SUM(Actual Sale Amount × Service Category Weight)
Price Weight = matched from Category Adjusted Amount
TurnCredit = selected production weights
```

The UI explicitly states that Service Category Weight adjusts the amount before Price Weight lookup and is not the legacy service-score flag.

The lower section contains:

```text
Loaded Policy Summary
Result Summary
Legacy / Advanced
Preview Today Turn Count
Reload
Save Draft
Activate Policy
```

Current physical failure text:

```text
Draft policy load failed.
```

The operator wants exact source/database verification; do not treat UI wording alone as sufficient proof.

## Business boundary that must remain canonical

```text
Checkout/payment = primary flow.
TurnEngine = auxiliary flow.
```

Therefore:

- missing or invalid TurnEngine setup must never crash or block checkout;
- TurnEngine may be `Not Configured`, `Deferred`, or `Setup Required`;
- the four-weight setup is still important and must not silently use invented values;
- this prompt does not retest or modify checkout.

## Documentation gate

Before editing source/tests/docs/seed artifacts, read completely:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report070.md
report/report071.md
```

Also read the complete local-only evidence artifact:

```text
FourWeightPolicyV001
```

Record locally before the first edit:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=<actual current version>
CanonicalDocSha256=<actual current hash>
```

## Phase A — exact Employee Turn Settings UI/load audit

Locate the real window/control, ViewModel/code-behind, application service, repository/data access, model, and validation path.

Map the exact chain for:

```text
open Employee Turn Settings
-> resolve tenant/local station context
-> load draft policy
-> load active policy
-> load four weight groups
-> populate Loaded Policy Summary
-> populate Result Summary
-> enable Save Draft / Activate Policy / Preview
```

For every stage, record locally:

- repository-relative file and method;
- input keys;
- expected result when no policy exists;
- exception/error handling;
- UI state transition;
- whether a missing row is treated as normal `Not Configured` or as an error.

Prove the exact cause of the physical message:

```text
Draft policy load failed.
```

Classify it as one of:

```text
NO_DRAFT_POLICY_EXPECTED_BUT_TREATED_AS_ERROR
MISSING_POLICY_HEADER
MISSING_REQUIRED_CHILD_ROWS
SCHEMA_MISMATCH
TENANT_RESOLUTION_FAILURE
QUERY_PREDICATE_FAILURE
MAPPING_FAILURE
VALIDATION_FAILURE
OTHER_PROVEN_CAUSE
```

Do not hide the issue with a broad catch or fake empty object.

## Phase B — prove the canonical four-weight contract

Use active source plus read-only reference-data evidence to verify the four UI dimensions.

For each of the four dimensions, create a local operator table containing:

| Operator label | Active source model/property | Local DB table/column | Input data used | Formula position | Optional/always active | Valid type/range | Required child rows | Runtime effect |
|---|---|---|---|---|---|---|---|---|

Required dimensions to verify:

```text
Employee Weight
Service Category Weight
Price Weight
Customer / Booking Weight
```

Explicitly classify all other discovered factors as one of:

```text
legacy
advanced/future
derived input
runtime state
manual adjustment
dead/unreachable
```

Examples that must be classified rather than merged into the four production dimensions:

```text
legacy service factor
service-score flags
time weight
manual adjustment
VIP/skill/appointment factors
```

Do not guess that similarly named legacy fields are part of the current production contract.

## Phase C — complete database table inventory

Inspect the reference database and current target database in read-only transactions only:

```sql
BEGIN TRANSACTION READ ONLY;
-- inspection only
ROLLBACK;
```

Create a complete local-only inventory of every table participating in any of these actions:

```text
load draft
load active policy
save draft
activate policy
employee weight
service-category weight
price weight / amount bands
customer-booking weight
preview today's turn count
manage/review turn
runtime turn result/state
legacy/advanced sections
sync/outbox, if existing
```

For each table, report directly in the Codex operator response:

| Table | Role | Tenant scoped | Header/child/runtime | Required for clean setup | Required only after employees/categories exist | FK/dependency order | Seed candidate | Must never seed |
|---|---|---:|---|---:|---:|---|---:|---:|

Also report:

- primary/unique keys;
- active/draft/version/status constraints;
- parent-child delete/update behavior;
- whether exactly one active policy is enforced;
- whether draft and active policies may coexist;
- whether price rules require ordered non-overlapping ranges;
- whether employee/category child rows can exist before business entities exist;
- which tables contain configuration versus calculated/runtime history.

Do not print customer, employee, service, or business-row contents.

## Phase D — reference versus target gap analysis

Compare sanitized structure/count state between the read-only reference database and current local target.

Determine exactly why the target setup window cannot load.

Return an operator-only gap matrix directly in the Codex response:

| Dependency | Reference state | Target state | Required for UI load | Required for Save Draft | Required for Activate | Repair/seed decision |
|---|---|---|---:|---:|---:|---|

Exact private values may remain in local evidence, but the operator response must be detailed enough to identify every missing table/row class without exposing secrets or raw business data.

No target DB insert/update/delete is authorized.

## Phase E — correct the setup-window behavior

Implement only source changes supported by the audit.

Required behavior:

### No draft exists

```text
Loaded Policy Summary = Not Configured / No Draft
Status line = clear non-error message
Save Draft = available when inputs are valid
Activate Policy = disabled until a valid draft exists
Reload = safe and repeatable
```

A normal absence of a draft must not display `Draft policy load failed`.

### Actual load error

```text
Status = Load Failed
sanitized result code shown
Save/Activate disabled if state is unsafe
no raw exception or connection details shown
```

### Existing draft

```text
all four dimensions load deterministically
always-active Service Category Weight remains enforced
optional weights preserve enabled state
child rows load without duplicates
summary matches the editor state
```

### Existing active policy

```text
active policy summary is visible
editing does not silently modify active production values
Save Draft creates/updates draft only
Activate uses the proven version/activation transaction
```

Do not copy reference values into the current target automatically.

## Phase F — design and implement the future-install seed

Create a new versioned seed artifact/implementation for future clean installations. Do not overwrite previous seed versions.

The seed must be:

- tenant scoped;
- idempotent;
- one transaction;
- deterministic;
- safe to rerun;
- non-destructive to operator-edited policies;
- free of customer, employee, service, booking, invoice, payment, turn-history, and other runtime business data;
- compatible with an installation where employees and service categories do not yet exist.

### Seed decision must be evidence-driven

Determine which of these is canonical:

```text
A. Seed no policy row; UI treats absence as Not Configured.
B. Seed one disabled/draft policy shell with no invented weight values.
C. Seed one neutral/default policy only if active source and reference evidence prove a universal product default.
D. Another proven minimal configuration.
```

Do not choose C merely to avoid a null policy.

For each related table, classify:

```text
SEED_AT_INSTALL
CREATE_WHEN_OWNER_SAVES_DRAFT
CREATE_AFTER_EMPLOYEES_EXIST
CREATE_AFTER_SERVICE_CATEGORIES_EXIST
RUNTIME_ONLY_NEVER_SEED
LEGACY_NOT_SEEDED
```

Examples of required decisions:

- policy header/version/status row;
- employee-specific weight rows;
- service-category-specific rows;
- price-band rows;
- customer/booking rule rows;
- active-policy pointer/state;
- preview/runtime turn rows;
- outbox rows under the existing baseline-seed convention.

### Seed integration

Use the existing canonical installation/Phase 2 seed pipeline and versioning convention. Do not create a parallel seed framework.

Seed implementation must:

- skip existing operator configuration;
- never activate an incomplete policy;
- never create duplicate drafts or active policies;
- preserve tenant isolation;
- emit only the existing required outbox events if the canonical seed policy requires them;
- leave TurnEngine `Not Configured`/`Deferred` when dependent business data is absent.

Do not execute the seed against the current local target in this task.

## Phase G — tests

Add focused tests covering at minimum:

1. no draft is a normal `Not Configured` state;
2. actual load failure remains a safe explicit failure;
3. four production dimensions map correctly;
4. Service Category Weight is always active;
5. optional weights persist enabled/disabled state;
6. draft save does not mutate active policy;
7. activation produces exactly one active policy;
8. seed is idempotent;
9. seed does not overwrite operator values;
10. seed works before employees/categories exist;
11. dependent child rows are deferred until their entities exist;
12. runtime/history tables remain empty;
13. missing TurnEngine setup remains non-blocking to the primary checkout boundary at the contract/unit level only—do not run physical checkout.

Run the WPF build and a focused test subset that does not use a broad `~Migration` filter known to pull unrelated missing-file tests.

## Required local evidence

Create a new versioned local evidence folder, preserving all prior versions, containing at minimum:

```text
employee-turn-settings-load-flow.mmd
four-weight-contract.csv
turn-policy-table-inventory.csv
reference-target-gap-matrix.csv
seed-table-classification.csv
seed-dependency-order.mmd
safe-db-counts.json
build-test-results.txt
README.md
SHA256SUMS.txt
```

Local evidence may contain repository-relative technical details and sanitized schema metadata. Do not include secrets or raw business rows.

## Required direct Codex operator response

In the final Codex response, before mentioning the GitHub report, include:

1. the proven root cause of `Draft policy load failed`;
2. the exact four production weights;
3. the complete related-table list with each table's purpose;
4. the exact minimal future-install seed set;
5. tables intentionally excluded from seed;
6. whether source UI loading was corrected;
7. whether seed implementation is ready;
8. build/test counts;
9. confirmation that current DB was not mutated and checkout was not tested.

This direct response is private operator handoff and may use exact table/class names. Do not include secrets, credentials, GUIDs, or business rows.

## Public coordination report — ultra-minimal from the first write

Create:

```text
report/report072.md
```

The GitHub report may contain only:

- verdict;
- four-weight contract confirmed: yes/no;
- setup-window load corrected: yes/no;
- future-install seed ready: yes/no;
- build/test counts;
- current DB not mutated;
- checkout not tested;
- local evidence artifact version and one aggregate SHA-256.

The public report must not contain:

- paths;
- class/method/property/table/column names;
- database names;
- schema/count metadata;
- formulas or exact values;
- architecture call chains;
- GUIDs, identifiers, usernames, or connection information;
- evidence filenames.

Do not first create a detailed commit and redact later.

Commit/push only the ultra-minimal coordination report. Do not commit or push OBM source.

## Expected verdicts

Preferred completion:

```text
OBM_POS_EMPLOYEE_TURN_SETTINGS_AND_FUTURE_SEED_READY_FOR_PHYSICAL_SETUP
```

Valid blocked verdicts:

```text
BLOCKED_TURN_POLICY_SCHEMA_OR_CONTRACT_UNRESOLVED
BLOCKED_TURN_POLICY_SEED_DEPENDENCY_UNRESOLVED
BLOCKED_EMPLOYEE_TURN_SETTINGS_BUILD_OR_TEST
```
