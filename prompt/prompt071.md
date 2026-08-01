# Prompt 071 — Audit the four TurnEngine weights, make TurnEngine non-blocking for checkout, and add setup UI

## Operator correction — authoritative business contract

The operator clarified the intended architecture:

```text
TurnEngine and its weight policy are important for fair employee turn allocation.
They are not part of the primary checkout/payment success path.
```

Therefore:

```text
Primary checkout/payment
-> must complete from valid local transaction data
-> must not be blocked by a missing/invalid TurnEngine weight policy

TurnEngine auxiliary calculation
-> must never calculate with an unknown/null policy
-> may complete, defer, or report Setup Required
-> must not throw a raw exception into checkout
-> must not silently invent financial/turn factors
```

The previous public report classified the policy as required. Interpret that precisely as:

```text
required for TurnEngine calculation
NOT required for primary checkout completion
```

This prompt supersedes any wording that made the TurnEngine policy a checkout prerequisite.

## Physical defect evidence

The operator reproduced a transaction and Visual Studio stopped on:

```csharp
if (p.UseEmployeeFactor)
{
    result *= item.EmployeeFactor ?? 1;
}
```

with:

```text
System.NullReferenceException
p was null
```

The surrounding code references multiple factor flags/values. The operator recalls four business weights, including concepts such as price weight and booking weight, but the exact current source/database contract must be proven rather than guessed.

## Mandatory documentation gate

Before editing source, tests, UI, configuration, or documentation, read completely:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report069.md
report/report070.md
```

Use the real local WPF root. Keep public artifacts redacted.

Record before the first edit:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V002
CanonicalDocSha256=<actual hash>
```

Canonical V002 installation/runtime architecture does not need a version change for this domain feature.

## Phase A — source and call-chain audit

Locate the exact method containing the null `p` dereference and map the full active transaction path:

```text
operator action
-> invoice/checkout command
-> transaction line creation
-> primary DB transaction boundary
-> TurnEngine invocation
-> policy resolution
-> factor calculation
-> turn-state write
-> checkout/payment commit
```

Report exact repository-relative files/methods and the order relative to:

```text
SaveChanges
BeginTransaction/Commit/Rollback
invoice writes
output/payment writes
turn-state writes
outbox writes
receipt completion
```

Answer these questions with source evidence:

1. Does TurnEngine run before, inside, or after the primary checkout transaction?
2. Can a TurnEngine failure currently roll back invoice/payment?
3. Can it leave partial turn rows or duplicate retry writes?
4. Does any factor alter sale/payment totals, or only turn allocation/scoring?
5. What exact type is `p` and what lookup returns it?

Do not make behavior changes before this audit is recorded locally.

## Phase B — identify the canonical four business weights

Search all active source/model/schema references related to:

```text
UseEmployeeFactor
UseServiceFactor
UseSkillFactor
UseVipFactor
AppointmentFactor
Booking
Price
TurnWeight
TurnFactor
EmployeeFactor
ServiceFactor
SkillFactor
VipFactor
```

The operator expects four business weights. Determine the exact active four dimensions used by the current TurnEngine.

Create a mapping table:

| Business label | Source flag/value | Input data source | Mathematical effect | Used by active path? | Legacy/unused? |
| --- | --- | --- | --- | --- | --- |

Reconcile source terminology with operator terminology such as:

```text
price weight
booking weight
and the remaining two weights
```

If source exposes more than four flags, classify each as:

```text
ACTIVE_CANONICAL_WEIGHT
DERIVED_INPUT_NOT_A_WEIGHT
LEGACY_UNUSED
OPTIONAL_MODIFIER
UNKNOWN
```

Do not collapse five unrelated factors into four without proof.

If the canonical four cannot be established, stop with:

```text
BLOCKED_TURN_ENGINE_WEIGHT_CONTRACT_AMBIGUOUS
```

## Phase C — read-only reference DB investigation

Investigate the reference database:

```text
enailsalon_phasee1_pos1_pg
```

and compare with the current local target DB, using configured protected credentials without printing secrets.

All DB investigation must run inside:

```sql
BEGIN TRANSACTION READ ONLY;
...
ROLLBACK;
```

Find every table/view/column involved in:

```text
TurnEngine policy
weight enable flags
weight numeric values
active/current policy selection
scope: tenant/shop/global/employee/service/category
valid date/version/status
```

For the reference DB, record locally:

```text
- exact table and column names;
- row counts;
- active-row selection predicate;
- whether exactly one active policy exists;
- numeric types/precision;
- nullability/defaults;
- safe value ranges inferred from constraints/source validation;
- exact four configured reference values and enable flags;
- related setup rows required for the lookup.
```

For the target DB, record safely:

```text
- table exists;
- matching policy row count;
- active policy row count;
- missing required setup dependencies;
- schema equality/differences versus reference.
```

Do not put raw policy values, private identifiers, DB names, absolute paths, or business data into the public GitHub report. Exact values may exist only in the local versioned evidence artifact for operator setup.

If the reference DB has no authoritative active policy or the source and DB disagree, stop with:

```text
BLOCKED_TURN_ENGINE_REFERENCE_DATA_MISSING
```

## Phase D — create TurnEngine domain contract before implementation edits

After source/reference investigation and before behavior/UI implementation, create the first domain-specific authority:

```text
<WPF_ROOT>/docs/TurnEngine/TURN_ENGINE_WEIGHT_CONTRACT.md
```

Required header:

```text
Version: V001
Status: Current TurnEngine Domain Authority
```

It must document:

```text
- exact four canonical business weights;
- source flag/value names;
- formula/order of application;
- allowed ranges and validation;
- policy scope and active-row selection;
- missing/invalid policy behavior;
- checkout is primary and non-blocking;
- TurnEngine is auxiliary and fail-closed for turn calculation only;
- no neutral/default substitution unless explicitly configured;
- setup UI ownership;
- transaction/atomicity boundary;
- retry/idempotency behavior.
```

Update `AGENTS.md` with a pointer requiring TurnEngine tasks to read this document first.

Required gate before implementation:

```text
TURN_ENGINE_DOCS_BEFORE_CODE=PASS
TurnEngineContractVersion=V001
TurnEngineContractSha256=<hash>
```

## Phase E — correct the checkout/TurnEngine boundary

Implement the smallest proven correction.

### Primary checkout rule

For valid sale/payment data:

```text
TurnEngine policy missing/invalid
-> checkout/payment continues
-> no raw exception
-> no wrong turn score/state is written
-> TurnEngine result = Deferred or SetupRequired
-> operator receives a non-blocking status/diagnostic
```

Do not show a modal that prevents payment completion.

Do not alter invoice totals, tax, tip, payment, receipt, or customer data because TurnEngine setup is missing.

### TurnEngine rule

```text
valid policy
-> calculate using the exact documented four-weight formula
-> persist turn result once

missing/invalid policy
-> do not calculate with guessed/default values
-> do not write a false successful turn result
-> return structured auxiliary result
```

Use a result contract such as the existing equivalent or a minimal explicit result:

```text
Completed
DeferredSetupRequired
Failed
```

with safe fields:

```text
StageId
ResultCode
PolicyResolved
TurnStateCommitted
SafeMessage
```

Do not create a new database table merely for this task unless an existing canonical deferred/recalculation table already exists and source evidence requires it.

### Transaction boundary

Prefer the proven safe architecture:

```text
primary checkout transaction commits independently
TurnEngine runs after primary commit or in an isolated auxiliary transaction
```

If current architecture cannot move it after commit safely, use the smallest proven isolation that guarantees:

```text
TurnEngine failure cannot roll back checkout
TurnEngine partial writes roll back
retry cannot duplicate turn state
```

Document the exact choice.

## Phase F — TurnEngine setup UI for immediate operator configuration

Audit existing settings screens first. Reuse an existing Turn/Queue/Advanced Settings area if present.

Do not create a separate application.

Add or repair a clearly named section/tab:

```text
TurnEngine Weights
```

The UI must expose the exact four audited weights, not guessed labels.

For each weight show:

```text
- business label;
- short description of its TurnEngine effect;
- Enabled toggle when the schema has one;
- numeric value input;
- valid range/format;
- current configured value;
- validation message.
```

Also show:

```text
Policy status: Configured / Not Configured / Invalid
Scope
Effective policy version/status when present
```

Commands:

```text
Save
Cancel/Reload
```

Save behavior:

```text
- resolve active local tenant/context;
- create or update the one canonical policy according to proven schema;
- validate all four values before writing;
- one local transaction;
- use existing audit/outbox path only if this setup entity already participates in sync;
- no duplicate active policy rows;
- no API required;
- no employee PIN/password gate beyond existing management access rules.
```

The prompt must not automatically save values. The operator will enter/confirm the four values physically after reviewing the reference evidence.

Do not connect normal WPF runtime to the reference DB. The reference DB is investigation-only.

## Phase G — source seed/install behavior

Audit whether a future clean installation should seed a TurnEngine policy.

Rules:

```text
- do not copy salon-specific reference values into every tenant automatically;
- if product-wide safe defaults are not proven, leave status Not Configured;
- checkout must still work;
- TurnEngine remains DeferredSetupRequired until configured;
- InstallationV0 may surface TurnEngine setup as post-install optional configuration, not an activation blocker.
```

If source already defines product-wide defaults, report and test them; do not invent new defaults.

## Phase H — tests

Add focused tests covering at least:

```text
exact four canonical weights and formula order
valid policy -> deterministic expected turn result
missing policy -> no NullReferenceException
missing policy -> checkout primary commit succeeds
missing policy -> no turn-state success write
missing policy -> structured DeferredSetupRequired result
invalid/out-of-range policy -> checkout succeeds, TurnEngine deferred/failed safely
TurnEngine partial write failure -> auxiliary rollback only
retry -> no duplicate turn-state rows
policy resolver returns exactly one active policy
multiple active policies -> safe invalid-policy result, no guessed selection
setup UI validates all four ranges
setup UI Save creates one canonical policy row
setup UI Save updates existing row without duplicate active rows
setup UI does not require API
reference DB is never used by runtime
no neutral/default policy object is silently constructed
```

Use disposable/in-memory/test DB infrastructure only. Do not mutate the operator target DB in automated tests.

Run:

```text
dotnet build <WPF_ROOT>/<WPF_PROJECT> -v minimal

dotnet test <WPF_TEST_ROOT>/<WPF_TEST_PROJECT> --filter "FullyQualifiedName~TurnEngine|FullyQualifiedName~TurnWeight|FullyQualifiedName~TurnFactor|FullyQualifiedName~TransactionCalculation|FullyQualifiedName~Checkout|FullyQualifiedName~TurnPolicy" -v minimal
```

Do not use broad `Migration` filters that pull unrelated missing SQL-artifact tests.

## Evidence and public-report policy

Create a versioned local evidence folder, for example:

```text
<RECOVERY_ROOT>/TurnEngine/FourWeightPolicyV001
```

Preserve locally:

```text
README.md
SHA256SUMS.txt
turnengine-call-chain.mmd
four-weight-source-map.csv
reference-db-policy-schema.md
reference-policy-values-private.json
checkout-turnengine-boundary-before-after.mmd
policy-resolution-matrix.csv
setup-ui-contract.md
test-results.txt
```

`reference-policy-values-private.json` must remain local only and must never be pushed.

Create a redacted public report from the start. Do not create a sensitive commit and redact later.

Public report must exclude:

```text
absolute paths
Windows usernames
real DB names
raw identifiers/GUIDs
exact private reference values
connection metadata
business rows
```

Use placeholders and counts only.

## Prohibited actions

Do not:

```text
- mutate the operator target DB automatically;
- mutate the reference DB;
- run checkout/payment automatically;
- launch or click WPF automatically;
- copy reference policy values into target automatically;
- invent default weights;
- make TurnEngine a checkout blocker;
- silently neutralize missing policy;
- change employee PINs;
- change DB roles/passwords;
- commit/push OBM source;
- print secrets or exact private policy values publicly.
```

Source/UI/tests/docs/local evidence changes are allowed.

## Documentation updates

Preserve current task/result under the next versioned history folder before updating.

Update current docs to state:

```text
- TurnEngine is auxiliary to checkout;
- four-weight contract V001 exists;
- setup UI is available;
- operator physical configuration is pending;
- checkout/TurnEngine physical retest is next;
- no automatic target DB mutation occurred.
```

## Report 071

Create and push a redacted public report:

```text
report/report071.md
```

Required sections:

1. Verdict.
2. Documentation gates and hashes using placeholders where needed.
3. Exact four canonical business weights, using safe generic labels if names are sensitive.
4. Source flag/value mapping and formula order.
5. Reference DB schema/row-count findings without exact private values.
6. Target missing/invalid policy classification.
7. Checkout versus TurnEngine transaction boundary before/after.
8. Missing-policy non-blocking behavior.
9. Setup UI location and fields.
10. Save/upsert/idempotency behavior.
11. Future clean-install behavior.
12. Exact repository-relative files changed.
13. Build/test counts.
14. Local evidence artifact version and public hashes.
15. No DB/process/secret/source-push mutation proof.
16. Exact operator physical setup/retest steps.
17. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_TURN_ENGINE_FOUR_WEIGHT_SETUP_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_TURN_ENGINE_WEIGHT_CONTRACT_AMBIGUOUS
```

```text
BLOCKED_TURN_ENGINE_REFERENCE_DATA_MISSING
```

```text
BLOCKED_TURN_ENGINE_BUILD_OR_TEST
```
