# Prompt 091 — Read the physical Price Rule Save diagnostic, prove the exact PostgreSQL failure, and repair the Save boundary

## Physical operator evidence — authoritative

The operator rebuilt and ran OBM-POS WPF directly on Windows against the normal local PostgreSQL runtime.

Physical path:

```text
Employee Turn Settings
-> Price Weight
-> Edit
-> Add Row
-> enter one valid rule
-> Save exactly once
```

The UI returned:

```text
PRICE_RULE_SAVE_DIAGNOSTIC_FAILED
StageId=SaveChanges
NewRows=1
ValidatedRows=1
OutboxRowsStaged=0
Committed=False
ReloadSucceeded=False
ExceptionType=Microsoft.EntityFrameworkCore.DbUpdateException
```

The diagnostic artifact is already present locally:

```text
E:\Project2026\RecoveryReports\PriceRuleDirectWindowsDiagnosticV001\price-rule-save-diagnostic-latest.txt
```

This proves:

```text
Add Row works
DataGrid edit reached the Save path
one new row was detected
validation passed
failure occurred during SaveChanges
transaction did not commit
outbox was not staged before failure
```

Do not investigate Docker. Docker is not part of this runtime.
Do not return to Load/ListView investigation.
Do not treat the generic DbUpdateException as the root cause.

## Objective

1. Read the local diagnostic artifact completely.
2. Extract the full sanitized exception chain and exact PostgreSQL/schema failure.
3. Reproduce the failure through the direct Windows/local PostgreSQL path or an equivalent local PostgreSQL test using the same schema and mapping.
4. Fix only the proven defect.
5. Complete one atomic Price Rule + TblLocalOutbox Save boundary.
6. Complete WPF inbound receiver/apply and no-echo behavior if the canonical sync contract can be proven.
7. Preserve PostgreSQL/Npgsql-only architecture.
8. Do not test checkout/payment.
9. Do not automatically mutate the operator's current DB. A later operator-triggered physical Save is the only permitted mutation of that DB.
10. Do not commit or push OBM source.

## Documentation gate

Read completely before editing:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report086.md
report/report087.md
report/report088.md
prompt/prompt090.md
```

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
RUNTIME_MODE=DIRECT_WINDOWS_WPF_LOCAL_POSTGRESQL
DOCKER_REQUIRED=NO
EVIDENCE_MODE=PHYSICAL_DBUPDATEEXCEPTION_ROOT_CAUSE
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase A — read and classify the exact physical exception

Read:

```text
E:\Project2026\RecoveryReports\PriceRuleDirectWindowsDiagnosticV001\price-rule-save-diagnostic-latest.txt
```

Return privately, sanitized:

```text
StageId
ResultCode
ProviderName
TenantResolved
PolicyScopeResolved
NewRows
DirtyRows
DeletedRows
ValidatedRows
InvalidRows
RuleEntityStates
OutboxRowsStaged
SaveChangesCallNumber
TransactionStarted
Committed
ReloadSucceeded
ExceptionType
all InnerException types in order
all sanitized exception messages in order
PostgreSQL SqlState
SafeTable
SafeColumn
SafeConstraint
EF entries included in DbUpdateException.Entries
```

If the current diagnostic omitted the inner PostgreSQL exception, improve diagnostics locally and reproduce once against the same direct Windows/local PostgreSQL runtime. Do not ask for Docker.

Classify the root cause using exactly one primary code:

```text
PRICE_RULE_TIMESTAMP_KIND_MISMATCH
PRICE_RULE_REQUIRED_COLUMN_NULL
PRICE_RULE_FOREIGN_KEY_MISSING
PRICE_RULE_POLICY_SCOPE_REQUIRED
PRICE_RULE_TENANT_SCOPE_MISMATCH
PRICE_RULE_DECIMAL_PRECISION_OR_RANGE
PRICE_RULE_COLUMN_MAPPING_MISMATCH
PRICE_RULE_DUPLICATE_OR_UNIQUE_CONSTRAINT
PRICE_RULE_ENTITY_TRACKING_CONFLICT
PRICE_RULE_KEY_GENERATION_FAILURE
PRICE_RULE_OTHER_POSTGRESQL_CONSTRAINT
```

Do not choose a code without direct inner-exception/schema proof.

## High-probability checks — prove, do not assume

Because a similar PostgreSQL issue previously occurred in Employee Weight, explicitly inspect every timestamp assigned by Price Rule Save:

```text
DateTime.UtcNow
DateTime.Now
CreatedAt
UpdatedAt
```

Compare C# `DateTime.Kind` with the actual PostgreSQL column type:

```text
timestamp without time zone
timestamp with time zone
```

If a local timestamp helper such as `PostgreSqlLocalTimestamp.UtcNow()` is the established project convention, use it only when the actual column is `timestamp without time zone` and the exception proves this mismatch.

Also prove whether `TblTurnAmountRule` requires a policy/version foreign key. The editor is intentionally allowed in `No Draft / No Active Policy` state. If the schema requires a non-null policy key, do not silently create a Draft Policy. Return:

```text
BLOCKED_PRICE_RULE_POLICY_SCOPE_REQUIRED
```

with a domain-safe remediation proposal.

## Phase B — prove complete model/schema mapping

Return privately the actual mapping for the Price Rule entity:

```text
entity property
DB column
C# type
PostgreSQL type
nullable
precision/scale or max length
key generation
FK and unique constraints
CreatedAt/UpdatedAt semantics
```

Include the complete current insert mapping and the exact values/source categories assigned to all required columns, but redact private business values and identifiers.

## Phase C — repair the proven Save defect

Provide complete BEFORE and AFTER method bodies, paths, and line ranges for:

```text
PriceWeightSettingWindow Save handler
DataGrid cell/row commit logic
new/dirty/deleted detection
validation
TurnAmountRuleSettingService Save method
entity insert/update/delete mapping
transaction begin/commit/rollback
outbox staging
SaveChangesAsync
reload-after-save
exception/result mapping
```

Do not provide isolated snippets.

Required Save boundary:

```text
commit focused DataGrid edit
-> compute final new/dirty/deleted set
-> validate complete final rule set
-> resolve current tenant and proven ownership scope
-> begin one Npgsql transaction
-> reload current tenant rule set for concurrency verification
-> apply inserts/updates/deletes
-> stage canonical TblLocalOutbox event(s) in the same DbContext/transaction
-> one SaveChangesAsync when possible
-> commit once
-> reload through canonical loader
```

If outbox staging cannot be proven before commit:

```text
explicit failure
rollback rule changes
Committed=False
```

No local-success/sync-missing state.

## Phase D — outbox and receiver/apply

Audit the existing sync conventions and choose only the proven canonical granularity:

```text
one event per rule CRUD
or
one aggregate rule-set replacement event per Save
```

Prove complete sender payload mapping and WPF inbound apply:

```text
validate tenant/envelope
idempotent insert/update/delete or replacement
stable IDs
correct PostgreSQL types
replay creates no duplicates
wrong tenant rejected
no TblLocalOutbox echo
runtime Price Rule state refreshed
```

If receiver/apply cannot be proven within the established framework, use verdict:

```text
BLOCKED_PRICE_RULE_RECEIVER_APPLY_UNPROVEN
```

Do not report ready merely because local Save succeeds.

## Phase E — direct local PostgreSQL proof without Docker

Use the installed Windows PostgreSQL service. Do not require Docker.

Preferred proof order:

1. Use an existing approved disposable local PostgreSQL DB if available.
2. Otherwise create a uniquely named disposable DB through operator-safe credentials already used by the project test harness.
3. Never point automated mutation tests at `obm_pos_dev_v0_pg`.
4. If creation credentials are unavailable, run a transaction-rollback integration probe against an approved disposable DB only; do not fall back to the operator DB.

Prove:

```text
one valid Add Row + Save inserts exactly one rule
expected outbox count is exact
reload returns the saved rule
update works
delete works
validation failure writes zero
outbox failure rolls back rule mutation
no-op writes zero
receiver apply works
receiver replay is idempotent
receiver creates no outbox echo
wrong tenant is rejected
Test Lookup uses production selector
```

## Tests/build

Run:

```text
WPF build
focused Price Rule tests
focused PostgreSQL mapping tests
focused outbox tests
focused receiver/apply tests
```

Preserve:

```text
UseSqlServer active matches = 0
SQL Server direct package refs = 0
ProviderName = Npgsql.EntityFrameworkCore.PostgreSQL
```

## Physical retest gate

Only after direct root-cause proof and local PostgreSQL integration PASS, return a concise operator retest:

```text
1. Stop old WPF processes.
2. Rebuild.
3. Open Price / Amount Rule Settings.
4. Add one valid rule.
5. Save exactly once.
6. Expect Committed=True and OutboxRowsStaged>0 according to the proven contract.
7. Close/reopen and verify persistence.
8. Save again without changes and verify no new outbox.
9. Do not activate Turn Policy.
10. Do not test checkout/payment.
```

## Public report

Create and push only:

```text
report/report091.md
```

It may contain only:

```text
verdict
physical root cause proven yes/no
root cause classification code
local Save boundary proven yes/no
outbox contract proven yes/no
receiver apply proven yes/no
real local PostgreSQL proof yes/no
build/test counts
current DB automatically mutated no
checkout tested no
one aggregate evidence SHA-256
```

Do not expose paths, source details, schema details, exception text, identifiers, payloads, or private values in the public report.

## Valid verdicts

```text
OBM_POS_PRICE_RULE_SAVE_SYNC_READY_FOR_PHYSICAL_RETEST
BLOCKED_PRICE_RULE_POLICY_SCOPE_REQUIRED
BLOCKED_PRICE_RULE_RECEIVER_APPLY_UNPROVEN
BLOCKED_PRICE_RULE_ROOT_CAUSE_STILL_UNPROVEN
BLOCKED_PRICE_RULE_BUILD_OR_TEST
```
