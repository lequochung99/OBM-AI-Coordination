# Prompt 087 — Determine why Price Weight rule list is empty and establish the correct rule materialization path

## Operator physical evidence

After prompt086, the old SQL Server/provider exception is gone.

`Employee Turn Settings -> Price Weight -> Edit` now opens `Price / Amount Rule Settings` successfully and shows:

```text
PRICE_RULE_LOAD_EMPTY - No Price / Amount rules are configured yet.
```

The grid is empty.

This means the provider/load exception is closed, but it is not yet proven whether:

```text
A. the current target DB truly has zero Price / Amount rules;
B. rules exist but are filtered by the wrong tenant/policy/status predicate;
C. rules exist under a legacy/wrong scope;
D. rules were never migrated/materialized into the current clean-install flow;
E. the UI projection/binding is dropping returned rows.
```

Do not assume that an empty grid is a UI bug. Prove the database and query state first.

## Authoritative architecture

OBM-POS is PostgreSQL/Npgsql-only after prompt086.

Do not reintroduce SQL Server packages, provider switches, fallback DbContexts, or provider-specific branches.

The Price Weight editor must use the canonical PostgreSQL context factory.

## Business boundaries

```text
Checkout/payment = primary flow.
Price Weight = auxiliary TurnEngine configuration.
```

Therefore:

- do not test or modify checkout/payment;
- do not change service sale prices or invoice totals;
- no Draft/no Active Policy is a valid setup state;
- opening or loading the editor must not create a policy or rule row;
- the parent Included checkbox must not control access to the editor;
- explicit operator Save is the only normal mutation from this screen.

## Documentation/evidence gate

Read completely before editing:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report071.md
report/report072.md
report/report086.md
```

Read all existing local Price Weight / TurnEngine evidence and focused tests.

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
ARCHITECTURE_DECISION=POSTGRESQL_ONLY
EVIDENCE_MODE=TARGET_AND_REFERENCE_RULE_COUNT_PROOF
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase A — prove current target DB state read-only

Inspect the current target DB only inside:

```sql
BEGIN TRANSACTION READ ONLY;
-- inspection only
ROLLBACK;
```

Do not print raw GUIDs, rule names tied to private data, credentials, or connection strings.

Return privately aggregate proof for the actual rule table used by the active editor:

```text
Total rows
Rows for current tenant
Rows for current tenant + current policy/draft scope
Rows active
Rows inactive
Rows with null policy key
Rows under other tenants
Rows rejected by each query predicate
```

Also prove:

```text
current tenant resolved = yes/no
current Draft policy resolved = yes/no
current Active policy resolved = yes/no
query requires policy = yes/no
editor can load tenant-level rules without policy = yes/no
```

Capture the exact production LINQ/query method and every predicate.

Classify the target state using exactly one narrow code:

```text
TARGET_PRICE_RULES_EXIST_QUERY_FILTER_WRONG
TARGET_PRICE_RULES_EXIST_BINDING_OR_PROJECTION_WRONG
TARGET_PRICE_RULES_EXIST_WRONG_TENANT_OR_POLICY_SCOPE
TARGET_PRICE_RULES_EMPTY_EXPECTED
TARGET_PRICE_RULES_EMPTY_MATERIALIZATION_MISSING
TARGET_PRICE_RULE_SCHEMA_MISMATCH
OTHER_PROVEN_CAUSE
```

## Phase B — inspect the read-only reference DB

Inspect `enailsalon_phasee1_pos1_pg` read-only to determine whether the prior working POS contained Price / Amount rule rows and how they were scoped.

Return privately only sanitized/aggregate information:

```text
reference row count
active row count
scope type: tenant/policy/global/legacy
number of distinct amount bands
whether ranges are contiguous
whether ranges overlap
whether an open-ended final range exists
which factor/credit field is actually used by production lookup
```

Do not publish exact business rule values or raw rule names to GitHub.

Compare target vs reference structurally:

| Contract | Target | Reference | Decision |
|---|---|---|---|
| table exists | | | |
| schema matches | | | |
| rule rows exist | | | |
| tenant scope | | | |
| policy scope | | | |
| active/default state | | | |
| production lookup field | | | |

Do not automatically copy reference business values into the target DB.

## Phase C — prove the canonical Price Weight rule contract

Determine from active source/schema which fields are production versus legacy/future.

For every visible column:

```text
Rule Name
Min Amount
Max Amount
Factor1
Factor2
Turn Credit
Sort Order
Active
Notes
```

Return privately:

| UI column | Entity property | DB column | DB type | Nullable | Production use | Editable | Legacy/future |
|---|---|---|---|---:|---|---:|---|

Prove:

```text
- whether Min/Max are inclusive;
- whether Max can be null/open-ended;
- whether ranges may overlap;
- the deterministic winning rule when more than one range matches;
- whether inactive rows participate;
- whether Sort Order breaks ties or only controls display;
- whether Factor1, Factor2, or TurnCredit is the actual production Price Weight;
- whether rules are tenant-scoped or policy-versioned;
- whether rules may be prepared before a Draft exists.
```

The `Test Lookup` command must use the exact same shared lookup policy as production TurnEngine. No duplicate algorithm.

## Phase D — correct the proven load issue

### If target rows exist

Fix the smallest proven defect:

```text
wrong tenant/policy/status filter
wrong projection/null handling
wrong ItemsSource/collection update
wrong ordering
```

Required result:

```text
query rows > 0
projected rows == query rows
DisplayedRows == projected rows
no duplicates
```

### If target truly has zero rows

Treat the empty state as valid, but decide how the operator should obtain initial rules.

Audit `Reset Defaults` and the clean-install seed/materialization flow.

Select exactly one proven model:

#### Model A — NO_RULE_SEED_OPERATOR_OWNED

```text
clean install has zero rules
editor shows empty state
operator uses Add Row or Reset Defaults
Save persists the first rule set
```

Use when no product-wide default ranges are proven.

#### Model B — PRODUCT_DEFAULTS_IN_MEMORY

```text
clean install has zero persisted rules
Reset Defaults loads proven product defaults into the grid in memory
nothing is written until operator clicks Save
```

Use when product-wide defaults are proven and safe.

#### Model C — FUTURE_INSTALL_DEFAULT_RULE_SEED

```text
clean install materializes a proven product-default rule set
seed is tenant-safe, idempotent, and does not overwrite operator rules
```

Allowed only when the values are genuinely product defaults, not salon-specific reference values.

Do not select Model C merely because the reference DB has rows.

If the contract is not proven, use Model A and report the reference rules only as migration candidates.

## Phase E — Reset Defaults behavior

Audit the existing button.

Required behavior based on the selected model:

```text
Reset Defaults
-> creates/replaces editable grid rows in memory only
-> marks the editor dirty
-> does not write DB
-> does not create outbox
-> operator must click Save
```

It must not silently copy private reference values unless explicitly approved through a separate migration/import path.

If no defaults are proven, disable or relabel the button clearly rather than populating guessed rows.

## Phase F — Add/Delete/Save boundary

Preserve one canonical local transaction for explicit Save:

```text
commit focused DataGrid edit
-> capture inserts/updates/deletes
-> validate all rows/ranges
-> if no changes: NO_CHANGES, no DB/outbox
-> resolve tenant and valid scope
-> begin transaction
-> apply insert/update/delete
-> create canonical outbox events only if this entity has an existing sync contract
-> one SaveChangesAsync when possible
-> commit
-> reload
```

Validation must reject before transaction:

```text
negative amounts
Min > Max
illegal overlap according to the proven contract
duplicate sort/order keys when prohibited
invalid precision
invalid factor/credit values
missing required final open-ended range when contract requires it
```

No partial save.

If rule sync/apply is absent, do not invent a second sync channel. Report:

```text
BLOCKED_PRICE_RULE_SYNC_CONTRACT_MISSING
```

or keep local-only behavior only if the canonical architecture explicitly defines these rules as local-only.

## Phase G — real PostgreSQL proof

Use a disposable PostgreSQL harness with the production Npgsql model.

Prove at least:

```text
zero-rule load returns PRICE_RULE_LOAD_EMPTY without exception
existing tenant rules load all rows
other-tenant rules excluded
no Draft state still loads rules or valid empty state
nullable/open-ended Max loads safely
Reset Defaults is in-memory only
valid rule set saves atomically
invalid/overlap set rolls back
no-op save writes nothing
Test Lookup matches production lookup
```

If seed/default materialization is changed, also prove rerun idempotency and preservation of operator-owned rules.

Do not write to the operator's current DB automatically.

## UI result/status contract

Load success with rows:

```text
PRICE_RULE_LOAD_READY
TenantResolved=True
QueryRows=<n>
DisplayedRows=<n>
```

Valid empty state:

```text
PRICE_RULE_LOAD_EMPTY
TenantResolved=True
QueryRows=0
DisplayedRows=0
```

Wrong-scope/filter repair must report the corrected query result privately.

## Safety boundaries

- PostgreSQL/Npgsql only.
- No automatic mutation of current target DB.
- No automatic import from reference DB.
- No policy creation/activation.
- No checkout/payment test.
- No OBM source commit/push.
- No private rule values, raw IDs, credentials, connection strings, or source internals in public report.

## Private handoff requirements

Return directly to the operator:

1. Exact root-cause classification.
2. Current target aggregate rule counts and predicates.
3. Reference aggregate rule counts and structural comparison.
4. Exact rule ownership/scope contract.
5. Exact production lookup field and range semantics.
6. Selected initialization model A/B/C and why.
7. Source corrections made.
8. Save/outbox/apply decision.
9. Build/test/integration results.
10. Exact physical retest steps.

## Public report

Create and push only an ultra-minimal:

```text
report/report087.md
```

It may contain only:

```text
verdict
target rule state category
load/query corrected yes/no
initialization model A/B/C
Reset Defaults safe yes/no
save boundary proven yes/no
build/test counts
current DB automatically mutated no
checkout tested no
one aggregate evidence SHA-256
```

## Valid verdicts

```text
OBM_POS_PRICE_RULE_LIST_READY_FOR_PHYSICAL_RETEST
```

```text
OBM_POS_PRICE_RULE_EMPTY_STATE_AND_OPERATOR_SETUP_READY
```

```text
BLOCKED_PRICE_RULE_CONTRACT_AMBIGUOUS
```

```text
BLOCKED_PRICE_RULE_SYNC_CONTRACT_MISSING
```

```text
BLOCKED_PRICE_RULE_BUILD_OR_TEST
```
