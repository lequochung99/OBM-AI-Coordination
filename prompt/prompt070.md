# Prompt 070 — Investigate and correct transaction calculation NullReferenceException where policy `p` is null

## Physical operator evidence

The operator physically created/tested a transaction in OBM-POS and Visual Studio broke on:

```csharp
if (p.UseEmployeeFactor)
{
    result *= item.EmployeeFactor ?? 1;
}
```

Exception:

```text
System.NullReferenceException: Object reference not set to an instance of an object.
p was null.
```

The surrounding method also references policy flags such as:

```text
UseEmployeeFactor
UseServiceFactor
UseSkillFactor
UseVipFactor
Appointment...
```

This is authoritative physical evidence. The transaction path reached the calculation method with a valid `item` but a null policy/config object `p`.

## Critical rule

Do not apply a blind null suppression such as:

```csharp
p?.UseEmployeeFactor
p ??= new Policy()
p ?? new Policy { ... }
```

unless source/data/reference evidence proves that a missing policy is intentionally equivalent to a neutral/default policy.

This calculation may affect turn allocation, employee payout, service factor, appointment factor, or related financial/operational results. The correct missing-policy behavior must be proven before implementation.

## Mandatory documentation gate

Before editing source/tests/docs/config, read completely:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report066.md
report/report067.md
report/report069.md
```

Record before first source edit:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V002
CanonicalDocSha256=<actual current hash>
```

Canonical V002 does not require a version change for this bounded runtime defect unless investigation proves a new architecture or policy contract is needed.

## Exact objective

Determine why `p` is null in the real transaction path, establish the canonical policy-resolution contract, and ensure the transaction calculation cannot throw or silently calculate an incorrect amount/turn result.

Required outcome:

```text
transaction calculation receives a proven policy
OR
missing policy is handled through the proven canonical fallback/error behavior

no NullReferenceException
no partial transaction save
no silent financial/turn miscalculation
```

## Phase 1 — locate exact source and call chain

Search the exact code and symbols:

```text
if (p.UseEmployeeFactor)
UseEmployeeFactor
UseServiceFactor
UseSkillFactor
UseVipFactor
EmployeeFactor
ServiceFactor
SkillFactor
VipFactor
AppointmentFactor
baseTurn
```

Report:

```text
exact source file
class
method signature
line numbers
compile-time type of p
compile-time type of item
all callers
caller chain from UI transaction action to calculation
whether p is parameter/local/query result/cache/static property
where p is expected to be loaded
```

Map the physical flow, for example only after source proof:

```text
MainWindow/transaction UI
-> transaction item creation
-> employee/service selection
-> turn/payout calculation
-> policy lookup
-> factor calculation
-> persistence/UI result
```

Do not infer class/table names from the variable name `p`; prove them.

## Phase 2 — identify the exact null-producing branch

Trace every assignment to `p` and classify why it can become null:

```text
query returned no row
FirstOrDefault/SingleOrDefault returned null
wrong TenantGuid
wrong ServiceCategoryGuid/ServiceGuid/EmployeeGuid
policy table empty after Phase 2 seed
configuration not loaded at startup
cache not initialized/refreshed
inactive policy filtered out
legacy row expected but not migrated
new service/category has no matching policy
transaction item missing a key used by lookup
race/stale UI state
other proven cause
```

For each lookup predicate, record sanitized evidence:

```text
fields used
nullability
row-count result
active/status filters
fallback order
whether lookup is tenant-scoped
whether a global/default row is expected
```

## Phase 3 — read-only DB and reference audit

Inspect the current target database only with:

```text
BEGIN TRANSACTION READ ONLY;
...
ROLLBACK;
```

Target:

```text
obm_pos_dev_v0_pg
```

Audit only sanitized metadata relevant to the policy lookup:

```text
required table exists
relevant row counts
active/inactive counts
matching-row count for the failing lookup shape
whether one canonical default row exists
whether tenant/category/service/employee relationships resolve
whether Phase 2 baseline/seed omitted required rows
```

Do not print:

```text
customer data
employee names
service names
raw private GUIDs
prices/payout tied to named records
connection strings
credentials
```

Also inspect the proven reference/legacy source, read-only, when needed:

```text
enailsalon_phasee1_pos1_pg
```

Compare only safe structural/count/default-policy evidence to determine intended behavior.

Audit older/stable source code or recovery reports if a previous implementation exists. Do not assume the reference database is automatically canonical; use it as evidence.

## Phase 4 — classify the root cause

Choose one primary classification:

```text
REQUIRED_POLICY_SEED_MISSING
POLICY_LOOKUP_PREDICATE_WRONG
POLICY_CACHE_NOT_INITIALIZED
TRANSACTION_ITEM_KEY_MISSING
OPTIONAL_POLICY_NEUTRAL_FALLBACK_NOT_IMPLEMENTED
LEGACY_POLICY_NOT_MIGRATED
OTHER_PROVEN_CAUSE
```

Report the exact null-producing statement and the first point at which the system had enough information to prevent the null.

## Phase 5 — determine canonical missing-policy behavior

Use source/reference/business-flow evidence to select exactly one behavior.

### A. Policy is required

If every transaction must have a policy row:

```text
- validate/resolve policy before calculation;
- return a precise safe domain result before transaction persistence;
- show an operator-actionable message;
- do not calculate with guessed defaults;
- do not partially save invoice/output/turn/payment state;
- correct the seed/migration/lookup that caused the row to be missing.
```

Possible safe result shape:

```text
StageId=ResolveTransactionCalculationPolicy
ResultCode=TRANSACTION_CALCULATION_POLICY_MISSING
SafeMessage=Transaction calculation setup is missing. Open the relevant setup screen or contact support.
```

Use actual project result conventions; do not create a parallel framework.

### B. Policy is optional with a neutral fallback

Only if evidence proves missing policy historically means all factor switches are disabled and multiplier remains neutral:

```text
- use one explicit canonical neutral policy/fallback;
- name it clearly (for example NeutralCalculationPolicy);
- centralize it in the policy resolver, not inline at every dereference;
- log only a safe diagnostic/counter;
- calculation result remains baseTurn unless other proven rules apply;
- no silent per-call object creation with unknown defaults.
```

### C. Seed/config row omitted

If Phase 2 or clean-install seed omitted a mandatory default:

```text
- fix future seed materialization idempotently;
- add a versioned repair plan for existing databases;
- do not mutate obm_pos_dev_v0_pg in this prompt;
- return a verdict that identifies operator repair as a separate step.
```

## Phase 6 — transaction atomicity and error boundary

Audit the transaction path around this calculation:

```text
invoice/output row creation
employee/service assignment
turn state
payment state
outbox/event creation
SaveChanges/transaction boundaries
UI observable state
```

Prove that when calculation fails:

```text
no partial DB commit
no duplicate retry rows
no stale UI transaction item left as successfully completed
no incorrect outbox event
```

If the calculation occurs after partial writes, move validation/calculation before commit or wrap the whole operation in the existing transaction boundary. Do not invent a second transaction framework.

## Phase 7 — diagnostics

Replace raw NullReferenceException exposure with safe structured diagnostics at the policy resolver/calculation boundary:

```text
StageId
ResultCode
PolicyResolved
FallbackUsed
CalculationSucceeded
SafeMessage
```

Do not expose raw identifiers, customer/employee/service data, prices, payout values, or secrets in public reports/logs.

Preserve developer exception details in local debug logging only where the existing diagnostic policy permits it.

## Phase 8 — tests

Add focused tests covering the actual policy type and lookup contract:

```text
policy resolved -> employee/service/skill/VIP factors apply correctly
all switches false -> result equals baseTurn subject to existing rounding
required policy missing -> explicit domain failure, no NullReferenceException
optional policy missing -> proven neutral fallback only when contract supports it
wrong tenant/category/service/employee lookup -> no accidental cross-tenant policy
inactive policy behavior matches proven contract
missing transaction item key -> precise validation failure
transaction failure -> no partial persistence/outbox
retry after corrected setup -> one successful transaction, no duplicate rows
null item and null policy are distinguished
rounding behavior unchanged
```

Add a regression test matching the physical path as closely as possible without exposing business data.

## Phase 9 — Graphify/source evidence

Create the next available evidence folder:

```text
E:\Project2026\RecoveryReports\TransactionCalculation\NullPolicyV001
```

Never overwrite an existing version.

Preserve:

```text
README.md
SHA256SUMS.txt
physical-call-chain.mmd
policy-resolution-flow-before-after.mmd
policy-source-inventory.csv
safe-db-policy-counts.json
transaction-atomicity-proof.md
test-results.txt
```

No raw identifiers or business values.

## Build/test commands

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~Transaction|FullyQualifiedName~Turn|FullyQualifiedName~Factor|FullyQualifiedName~Calculation|FullyQualifiedName~Invoice|FullyQualifiedName~Checkout|FullyQualifiedName~Policy" -v minimal
```

If the broad filter pulls unrelated known missing ApiServer migration-file tests, report them separately and run a precise source-owned subset. Do not claim full-suite PASS for tests that were not run.

## Documentation updates

Canonical V002 remains unchanged unless the business policy contract itself changes.

Preserve current task/result under the next versioned history folder before updating current docs.

Update `CURRENT_RESULT.md` with:

```text
exact root cause
canonical missing-policy behavior
source correction
build/test evidence
physical retest pending
```

Update `CURRENT_TASK.md` to the exact physical transaction retest.

## Prohibited actions

Do not:

```text
mutate obm_pos_dev_v0_pg
run transaction/checkout automatically
launch/click WPF automatically
change service ordering behavior
change employee operational PIN policy
change API contracts/tokens
change DB roles/passwords
print business transaction values or raw identifiers
commit/push OBM source
drop ASP.NET Identity tables
```

Read-only DB inspection, source/test/docs changes, builds, tests, and evidence creation are allowed.

## Report 070

Create and push:

```text
report/report070.md
```

Required sections:

1. Verdict.
2. DOCS_READ_BEFORE_CODE_GATE proof.
3. Physical NullReferenceException evidence.
4. Exact file/method/call chain.
5. Exact type/source/assignment of `p`.
6. Null-producing branch and lookup predicate.
7. Read-only target/reference DB evidence.
8. Root-cause classification.
9. Proven canonical missing-policy behavior.
10. Minimal source correction.
11. Transaction atomicity/no-partial-write proof.
12. Structured diagnostics/result contract.
13. Exact files changed.
14. Build/test commands and counts.
15. Evidence folder and hashes.
16. No DB/process/source-push/secret mutation proof.
17. Exact operator physical transaction retest steps.
18. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_TRANSACTION_CALCULATION_NULL_POLICY_READY_FOR_PHYSICAL_RETEST
```

```text
OBM_POS_TRANSACTION_CALCULATION_POLICY_REPAIR_READY_FOR_OPERATOR_APPLY
```

```text
BLOCKED_TRANSACTION_CALCULATION_POLICY_CONTRACT_AMBIGUOUS
```

```text
BLOCKED_TRANSACTION_CALCULATION_BUILD_OR_TEST
```

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```
