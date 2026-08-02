# Prompt 085 — Fix Price / Amount Rule Settings load failure and prove the PostgreSQL rule contract

## Operator physical evidence

The operator returned to `Employee Turn Settings`, selected `Price Weight`, and clicked `Edit`.

The `Price / Amount Rule Settings` window opens, but the grid does not load. The current dialog says:

```text
Unable to load price/amount rules: An exception has been raised that is likely due to a transient failure. Consider enabling transient error resiliency by adding 'EnableRetryOnFailure' to the 'UseSqlServer' call.
```

Visible window state:

```text
Price / Amount Rule Settings
Rule Name
Min Amount
Max Amount
Factor1
Factor2
Turn Credit
Sort Order
Active
Notes
Sample Amount
Test Lookup
Load
Save
Add Row
Delete Selected
Reset Defaults
Close
```

The parent page currently shows `Not Configured / No Draft`. The child editor still must be able to open and load its rule state before any Turn Policy is activated.

## Critical warning

Do not blindly add `EnableRetryOnFailure`.

Do not add or retain `UseSqlServer` in the active PostgreSQL POS path merely because the wrapper message mentions it.

OBM-POS uses PostgreSQL. The message may be a generic execution-strategy wrapper, a wrong-provider DbContext, a nested exception, or another mapping/schema error. Obtain the exact inner exception and active provider before editing.

## Authoritative business boundaries

```text
Checkout/payment = primary flow.
TurnEngine Price Weight = auxiliary configuration.
```

Therefore:

- this task must not test or modify checkout/payment;
- Price Weight rules must not change service prices or invoice totals;
- rules affect only TurnEngine lookup/credit after an applicable policy is activated;
- no Draft / no Active Policy is a normal setup state;
- opening/loading the child must not create or activate a policy;
- the parent `Included` checkbox must not be a prerequisite for opening or loading the Price editor.

The UI itself states that Price Weight is matched from `CategoryAdjustedAmount` and is not used until the Turn Policy is activated later. Verify this from active source; do not rely on UI text alone.

## Scope

1. Prove the exact load failure and active database provider.
2. Correct the Price / Amount Rule load path.
3. Prove the rule table/schema and tenant/policy ownership contract.
4. Ensure zero-rule/no-Draft states load safely without mutation.
5. Audit and, where required for consistency, correct Add/Edit/Delete/Reset/Test Lookup/Save behavior.
6. Preserve a clear transaction and sync/outbox boundary for explicit operator saves.
7. Do not mutate the operator's current DB automatically.
8. Do not test checkout/payment.

## Mandatory documentation gate

Read completely before editing:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report071.md
report/report072.md
report/report078.md
report/report079.md
```

Read any existing local TurnEngine/Price Weight evidence and tests.

Record locally before first edit:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
EVIDENCE_MODE=DIRECT_PROVIDER_EXCEPTION_AND_REAL_SCHEMA_PROOF
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase A — freeze and trace the exact current load path

Before source edits, capture the complete active chain:

```text
Employee Turn Settings
-> Price Weight Edit command
-> Price / Amount Rule Settings window construction
-> Loaded/open event
-> tenant/local station resolution
-> policy context resolution, if any
-> DbContext/factory creation
-> active provider selection
-> price/amount rule query
-> projection/view-model rows
-> ItemsSource/DataContext
-> grid item count
-> exception mapping/dialog
```

Return privately the complete current method bodies, repository-relative paths, and line ranges for:

1. Parent Price Weight Edit handler.
2. Child window constructor and Loaded/open handler.
3. Load button handler.
4. Canonical load method/service.
5. Tenant resolution.
6. Draft/Active policy resolution used by this editor.
7. DbContext factory/configuration path.
8. Price rule query and ordering.
9. Projection into grid rows.
10. Exception catch/result mapping.

Do not return only one-line excerpts.

## Phase B — obtain exact exception/provider evidence

The generic transient message is not a root cause.

Capture privately:

```text
ExceptionType
InnerExceptionType chain
sanitized messages
PostgreSQL SqlState, if present
server/provider name
ctx.Database.ProviderName
actual EF provider registration path
whether UseNpgsql or UseSqlServer is active
failing SQL/query stage
stack trace from Edit/Load to failing statement
table/column/constraint when applicable
connection-open state
transaction/execution-strategy state
```

Never print passwords, connection strings, raw tenant IDs, or business rows.

Classify the root cause using the narrowest proven category:

```text
WRONG_DB_PROVIDER_OR_FACTORY
GENERIC_EXECUTION_STRATEGY_WRAPPER
POSTGRES_CONNECTION_FAILURE
POSTGRES_TRANSIENT_FAILURE
TENANT_CONTEXT_NOT_RESOLVED
NO_DRAFT_EXPECTED_BUT_TREATED_AS_ERROR
POLICY_CONTEXT_JOIN_FAILURE
TABLE_OR_COLUMN_MISSING
EF_COLUMN_MAPPING_MISMATCH
TYPE_OR_NULLABILITY_MISMATCH
QUERY_TRANSLATION_FAILURE
ORDERING_OR_PROJECTION_FAILURE
RUNTIME_ROLE_SELECT_DENIED
OTHER_PROVEN_CAUSE
```

Do not enable retries until the failure is proven transient and the retry boundary is safe/idempotent.

## Phase C — prove the Price / Amount Rule data contract

Audit active source, EF mapping, target schema, and read-only reference evidence for the Price Weight rule entity, expected to involve `TblTurnAmountRule` or the proven current equivalent.

For each displayed field, return privately:

| UI field | Entity property | DB column | C# type | DB type | Nullable | Default | Precision/length | Constraint | Runtime role |
|---|---|---|---|---|---:|---|---|---|---|
| Rule Name | | | | | | | | | |
| Min Amount | | | | | | | | | |
| Max Amount | | | | | | | | | |
| Factor1 | | | | | | | | | |
| Factor2 | | | | | | | | | |
| Turn Credit | | | | | | | | | |
| Sort Order | | | | | | | | | |
| Active | | | | | | | | | |
| Notes | | | | | | | | | |

Prove:

1. Tenant-scoped vs policy-version-scoped ownership.
2. Whether a Draft Policy is required only for save/activation or not required at all.
3. Exact business key and uniqueness contract.
4. Range semantics: inclusive/exclusive Min/Max.
5. Whether Max may be null/open-ended.
6. Whether ranges may overlap.
7. Ordering and first-match/priority behavior.
8. Which factor is the current production Price Weight.
9. Whether Factor2/TurnCredit are active, legacy, derived, or future.
10. Whether inactive rows are loaded and how they are displayed.
11. Existing sync/outbox DTO and receiver/apply coverage.

Use read-only transactions for the current target and reference DB. Do not expose raw rule values publicly.

## Phase D — canonical load behavior

Implement one canonical load method used by both window open and `Load` button.

Required behavior:

```text
open child
-> resolve current local tenant
-> resolve rule ownership context without creating rows
-> use the canonical PostgreSQL DbContext/provider
-> load applicable rules
-> deterministic order
-> populate grid
-> show structured status
```

No Draft / no Active policy:

```text
window opens
load does not create a policy
load does not create rule rows
existing tenant-owned rules load when contract allows
otherwise show a clear empty state suitable for operator setup
```

Zero-rule state must not be an exception:

```text
PRICE_RULE_LOAD_EMPTY
No Price / Amount rules are configured yet.
```

Successful state must report safe counters such as:

```text
TenantResolved
ProviderIsPostgreSql
PolicyContextState
RowsFound
DisplayedRows
ResultCode
```

Do not display raw IDs or connection information.

## Phase E — Add/Delete/Reset/Test Lookup audit

Avoid fixing Load while leaving immediately adjacent actions broken.

### Add Row

- adds one unsaved in-memory row;
- does not mutate DB until explicit Save;
- uses safe defaults proven by source/schema;
- does not require policy activation.

### Delete Selected

- marks/removes the selected persisted rule according to the existing contract;
- confirms destructive action using the existing application pattern;
- does not partially save unrelated rows.

### Reset Defaults

Determine whether product defaults are canonical and safe.

- If proven defaults exist, reset in memory only until Save.
- If defaults are salon-specific/reference-only, do not copy them as universal product defaults.
- Do not mutate DB merely by clicking Reset Defaults.

### Test Lookup

For a sample amount:

```text
parse/validate amount
-> apply the exact production deterministic matching algorithm
-> show matched rule/factor/turn credit safely
-> no DB mutation
```

Zero match and overlapping-match conflicts must be explicit.

## Phase F — Save and sync/outbox boundary

Prove the active persistence contract before editing.

An explicit operator Save must use one clear boundary:

```text
commit current DataGrid cell/row
-> detect added/changed/deleted rows
-> validate all rows and range relationships
-> if no changes: NO_CHANGES, zero DB/outbox
-> resolve tenant and applicable rule ownership
-> begin one PostgreSQL transaction
-> apply all intended rule inserts/updates/deletes
-> add canonical outbox events using the existing rule entity contract
-> one SaveChangesAsync when possible
-> commit
-> reload through canonical loader
```

Validation must occur before mutation and cover at least:

```text
required rule name
nonnegative/valid amounts
Min <= Max when Max exists
numeric precision/range
valid factor/turn-credit values
sort order constraints
duplicate business keys
overlap contract
exactly one open-ended terminal band when required
```

If outbox infrastructure is unavailable, rollback. Do not silently commit local rules without sync when the entity is part of the established sync contract.

If Price rules are intentionally local-only and not synchronized, prove that from architecture and return it explicitly. Do not invent a new sync channel.

Inbound apply must be idempotent and must not create local outbox echo.

## Phase G — real PostgreSQL proof

Mock-only tests are insufficient for the provider-specific physical failure.

Use a disposable PostgreSQL database/harness with the production EF model and rule schema.

Prove at minimum:

```text
canonical load succeeds through Npgsql
zero-rule load returns empty state, not exception
no-Draft load does not create policy/rule rows
one persisted rule loads and projects correctly
nullable/open-ended fields project safely
wrong provider/factory regression is detected
one valid rule save commits with expected outbox behavior
invalid or overlapping rule set rolls back
no-op save writes nothing
Test Lookup matches the same deterministic production algorithm
inbound repeat is idempotent/no echo when sync applies
```

Do not write to the operator's current DB automatically.

## Tests/build

Run focused tests for Price/Amount Rule Settings, provider selection, tenant/no-Draft state, rule validation, lookup, save/outbox, and inbound apply where applicable.

Do not broaden the filter into unrelated migration suites.

Build the WPF project.

## Physical retest

After PASS:

1. Rebuild and launch WPF manually.
2. Open Employee Turn Settings with `No Draft / Not Configured` state.
3. Leave Price Weight unchecked and click Edit; child must open and load/empty safely.
4. Check Price Weight without saving Draft and click Edit; same result.
5. Confirm no transient/UseSqlServer dialog.
6. Confirm provider/status indicates the local PostgreSQL path without exposing connection details.
7. Click Load; no duplicate rows.
8. If zero rules, confirm clear empty-state text.
9. Add one safe test row in memory and validate Test Lookup.
10. Save only after the private handoff proves the exact rule contract and sync boundary.
11. Close/reopen and verify persistence when Save is approved.
12. Do not test checkout/payment.

## Safety boundaries

- No automatic mutation of the operator's current DB.
- No blind EnableRetryOnFailure change.
- No UseSqlServer addition to the PostgreSQL path.
- No automatic policy creation/activation on open/load.
- No checkout/payment test.
- No OBM source commit/push; source edits remain local.
- No credentials, connection strings, raw IDs, business rule values, or internal code in the public report.

## Private handoff requirements

Return directly to the operator:

1. Exact root cause and provider evidence.
2. Exact inner exception/stack trace, sanitized.
3. Complete before/after Load method bodies and unified diff.
4. Exact schema/mapping matrix.
5. Proven tenant/policy ownership contract.
6. Exact range/lookup algorithm.
7. Save/outbox transaction boundary and receiver/no-echo proof where applicable.
8. Real PostgreSQL integration-test code/output.
9. Build/test counts.
10. Physical retest steps.
11. Explicit list of unrelated paths left unchanged.

## Public report

Create and push only an ultra-minimal:

```text
report/report085.md
```

It may contain only:

```text
verdict
root-cause category
PostgreSQL provider path proven yes/no
no-Draft load supported yes/no
zero-rule empty state supported yes/no
rule contract proven yes/no
save/outbox boundary proven yes/no
real-schema proof yes/no
build/test counts
current DB automatically mutated no
checkout tested no
one aggregate evidence SHA-256
```

## Valid verdicts

```text
OBM_POS_PRICE_AMOUNT_RULE_SETTINGS_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_PRICE_RULE_PROVIDER_OR_EXCEPTION_UNPROVEN
```

```text
BLOCKED_PRICE_RULE_SCHEMA_OR_CONTRACT_AMBIGUOUS
```

```text
BLOCKED_PRICE_RULE_SYNC_CONTRACT_MISSING
```

```text
BLOCKED_PRICE_RULE_REAL_SCHEMA_PROOF
```

```text
BLOCKED_PRICE_RULE_BUILD_OR_TEST
```
