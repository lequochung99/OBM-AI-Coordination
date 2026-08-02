# Prompt 088 — Complete Price / Amount Rule Save, outbox, and WPF receiver apply contract

## Current proven state

Read `report/report087.md` and all local-only Price Weight evidence before editing.

Prompt087 established:

```text
Target rule state = TARGET_PRICE_RULES_EMPTY_EXPECTED
Load/query = corrected
Initialization model = A
Reset Defaults = safe
Save boundary = not proven
Blocker = Price Rule sync contract missing
```

Therefore the empty grid is not a load defect. The target database currently has no Price / Amount rules.

## New physical operator evidence — Save fails after Add Row

After prompt087, the operator physically used the current WPF UI:

```text
Price / Amount Rule Settings
-> Add Row
-> enter a rule in the grid
-> click Save
-> Save fails
```

This confirms that creating rules is required and that the remaining defect is in the real Price Rule Save pipeline, not in Load/ListView.

Treat this physical failure as mandatory reproduction evidence. Do not return a ready verdict based only on unit tests or static inspection.

Before editing the Save path, capture privately:

```text
exact dialog/status/result code
StageId
ExceptionType
InnerExceptionType
sanitized exception message
PostgreSQL SqlState if present
table/column/constraint if present
EF entity states
transaction state
SaveChangesAsync call number
full project stack trace from Save click to failure
```

If the current UI only exposes a generic Save failure, add temporary local diagnostics or use debugger inspection to obtain the inner exception, then remove any raw/private diagnostics before finalizing.

The private handoff must include complete BEFORE and AFTER C# method bodies for:

```text
Save click handler
DataGrid focused-cell/row commit
new/dirty/deleted row detection
validation
production Price Rule save service
current-tenant rule reload
insert/update/delete mapping
transaction begin/commit/rollback
outbox creation
SaveChangesAsync
reload-after-save
exception/result mapping
```

Do not provide isolated snippets. Include repository-relative paths and line ranges.

The remaining task is to make explicit operator setup fully functional:

```text
Add Row or Reset Defaults in memory
-> operator reviews/edits rules
-> Save
-> local PostgreSQL transaction
-> canonical TblLocalOutbox event(s)
-> WPF receiver/apply
-> no echo
-> reload
```

## Architecture and business boundaries

OBM-POS is PostgreSQL/Npgsql-only.

Do not reintroduce SQL Server providers, packages, fallback contexts, or provider switches.

```text
Checkout/payment = primary flow
Price Weight rules = auxiliary TurnEngine configuration
```

Do not test or modify checkout/payment.
Do not change service prices, invoice totals, or finalized sale amounts.
No Draft / no Active Policy remains a valid setup state.
The parent Price Weight Included checkbox must not be required to open, edit, or save tenant rule configuration.
Saving rules must not create or activate a Turn Policy and must not persist the parent Included checkbox.

## Initialization model A — authoritative

Preserve model A:

```text
clean/current tenant with zero rules
-> editor opens with PRICE_RULE_LOAD_EMPTY
-> no DB row is created automatically
-> no seed rule is inserted automatically
```

The owner creates rules explicitly through:

```text
Add Row
or
Reset Defaults -> review -> Save
```

`Reset Defaults` must remain in-memory only until Save.
Do not add Price Rule rows to the clean-install seed in this prompt.
Do not copy salon-specific reference values automatically.

If prompt087 proved product-safe default rows, preserve those exact defaults only in the Reset Defaults in-memory template. If product-safe defaults were not proven, keep Reset Defaults disabled or return an explicit no-default-template result. Do not invent ranges or factors.

## Scope

1. Prove the exact Price Rule entity/schema and ownership contract.
2. Implement Add/Edit/Delete/Reset dirty tracking and validation.
3. Reproduce and fix the operator's actual Add Row -> Save failure with direct exception evidence.
4. Implement one atomic PostgreSQL Save boundary.
5. Add canonical `TblLocalOutbox` events through the existing outbox framework.
6. Implement the missing WPF receiver/apply path through the existing inbound delivery framework.
7. Ensure inbound apply is idempotent and creates no local outbox echo.
8. Ensure Test Lookup uses the same production rule-selection algorithm.
9. Add real PostgreSQL integration proof.
10. Do not mutate the operator's current DB automatically.
11. Do not commit or push OBM source.

## Documentation gate

Read completely before edits:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report086.md
report/report087.md
```

Read all existing TurnEngine/Price Weight local evidence and tests.

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
ARCHITECTURE_DECISION=POSTGRESQL_ONLY
EVIDENCE_MODE=PRICE_RULE_SAVE_SYNC_REAL_SCHEMA
EVIDENCE_ESCALATION=PHYSICAL_SAVE_FAILURE_DIRECT_PROOF
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase A — prove the exact rule contract

Return privately the complete schema/model mapping for every editor field:

| UI field | Entity property | DB column | C# type | PostgreSQL type | Nullable | Precision/length | Runtime use | Sync payload field |
|---|---|---|---|---|---:|---|---|---|
| Rule Name | | | | | | | | |
| Min Amount | | | | | | | | |
| Max Amount | | | | | | | | |
| Factor1 | | | | | | | | |
| Factor2 | | | | | | | | |
| Turn Credit | | | | | | | | |
| Sort Order | | | | | | | | |
| Active | | | | | | | | |
| Notes | | | | | | | | |

Also prove:

```text
- tenant-scoped vs policy-versioned ownership;
- whether rule IDs are stable business IDs;
- whether Max Amount may be null/open-ended;
- whether Min/Max are inclusive or exclusive;
- whether ranges may overlap;
- exact winner algorithm when more than one rule could match;
- whether only active rules participate;
- tie-breaking order;
- which field is the production Price Weight result;
- which fields are legacy/future/display-only;
- whether deletes are hard delete or soft/inactive;
- whether Sort Order must be unique/contiguous;
- exact decimal precision and allowed ranges.
```

Do not rely only on UI labels. Prove from active source/schema/reference evidence.

## Phase B — editor state and dirty tracking

Use one row view model with original snapshots.

Required behavior:

```text
Load existing rows
-> IsDirty=false

Add Row
-> one new in-memory row
-> IsNew=true
-> no DB/outbox mutation

Edit any permitted field
-> IsDirty=true

Delete Selected existing row
-> mark deleted in memory
-> no DB/outbox mutation until Save

Delete Selected new unsaved row
-> remove from collection
-> no DB/outbox mutation

Reset Defaults
-> replace/populate in-memory template only
-> mark applicable rows new/dirty/deleted
-> no DB/outbox mutation until Save
```

Closing with unsaved changes must follow the application's existing dirty-confirmation pattern.

Commit the focused DataGrid cell and row before dirty detection on Save.

## Phase C — validation before transaction

Validate the entire final proposed rule set before opening a transaction.

At minimum prove and enforce the active contract for:

```text
Rule Name required/max length when required
Min Amount valid and non-negative when required
Max Amount null/open-ended or valid according to contract
Min <= Max
numeric precision/scale
Factor1/Factor2/TurnCredit allowed values
Sort Order allowed range
Active rule range overlap policy
open-ended rule uniqueness
stable deterministic lookup order
Notes max length
at least one rule only if business contract requires it
```

One invalid row must block all inserts/updates/deletes.
No partial save.
Show row-specific safe messages.

If the operator's physical Save failed because the row was invalid, the UI must report a precise validation result before transaction start; it must not return a generic Save exception.

## Phase D — canonical atomic Save boundary

Implement one clearly owned save service/method.

Required shape:

```text
UI commits focused edit
-> calculate final dirty/new/deleted set
-> if no changes: PRICE_RULE_SAVE_NO_CHANGES, zero DB/outbox writes
-> validate complete proposed rule set
-> resolve current local tenant
-> begin one PostgreSQL transaction
-> reload current tenant rule set
-> verify optimistic/concurrency assumptions
-> apply inserts
-> apply updates
-> apply deletes according to proven contract
-> add canonical outbox event(s) in the same DbContext/transaction
-> one SaveChangesAsync when possible
-> commit once
-> reload through canonical loader
-> return structured committed result
```

Do not require Draft Policy or Active Policy unless the proven schema has a mandatory policy FK. If a mandatory policy FK exists, explain how model A can allow pre-policy editing without creating a policy. Do not silently create a policy.

### Required structured diagnostics

Success must report concepts equivalent to:

```text
NewRows
DirtyRows
DeletedRows
ValidatedRows
InsertedRows
UpdatedRows
DeletedDbRows
OutboxRows
Committed
ReloadSucceeded
StageId
ResultCode
```

A failure must preserve accurate counters and the real failing stage. Do not repeat the earlier Employee Weight problem where validation counters and Save stage contradicted each other.

### Outbox event granularity

Audit the existing sync conventions and choose the proven canonical model:

```text
A. one outbox event per inserted/updated/deleted rule
or
B. one aggregate rule-set replacement event per operator Save
```

Do not invent a second outbox framework.

The chosen model must support insert, update, and delete idempotently.

If outbox infrastructure is unavailable:

```text
explicit save failure
transaction rollback
zero rule changes committed
```

No silent `OutboxRows=0` success.

A reload failure after commit must report:

```text
Committed=True
PRICE_RULE_SAVED_RELOAD_FAILED
```

It must not be reported as an uncommitted Save failure.

## Phase E — DTO/payload and WPF receiver apply

Trace and implement the complete existing sync path:

```text
local Price Rule change
-> canonical TblLocalOutbox
-> DTO/payload
-> API/relay contract
-> WPF inbound delivery
-> receiver/apply handler
-> local Price Rule table
-> runtime rule cache/state refresh
```

Return privately the relevant complete C# blocks for sender and receiver.

Receiver requirements:

```text
validate tenant/envelope identity
deserialize canonical DTO
idempotent insert/update/delete
preserve stable rule IDs
reject wrong-tenant payloads
apply correct PostgreSQL types/precision
no duplicate rows on replay
no local TblLocalOutbox echo
refresh Price Rule runtime state immediately
```

Do not create a separate TurnEngine sync channel.

If the API/relay DTO contract is missing and cannot be safely added within the existing employee/setup sync pattern, return:

```text
BLOCKED_PRICE_RULE_SYNC_CONTRACT_MISSING
```

Do not falsely report Save boundary proven without receiver apply proof.

## Phase F — Test Lookup contract

`Test Lookup` must be read-only.

It must use the same canonical algorithm as production TurnEngine Price Weight selection, not a duplicated UI-only implementation.

Required result concepts:

```text
SampleAmount
MatchedRule yes/no
MatchedRule safe display name/order
ProductionWeight result
ResultCode
```

Do not expose raw private IDs.
Do not write DB or outbox.

Add tests proving UI Test Lookup and production selector return the same result for boundary values, gaps, open-ended ranges, inactive rules, and tie-breaking.

## Phase G — real PostgreSQL integration proof

Mock-only tests are insufficient for Save/outbox/apply.

Use an approved disposable PostgreSQL harness with the active schema and Npgsql provider.

Prove at least:

```text
zero-rule load succeeds
Add Row + Save inserts rule(s)
operator physical Save failure is reproduced before the fix or its exact exception is proven
edit + Save updates rule(s)
delete + Save removes/deactivates according to contract
exact expected outbox event count
sender payload includes all required fields
receiver apply reproduces the same final rule set
receiver replay is idempotent
receiver creates no outbox echo
outbox failure rolls back rule mutations
validation failure writes nothing
no-op save writes nothing
wrong tenant rejected
focused cell edit committed before save
reload failure distinguished from commit failure
Test Lookup equals production selector
```

Do not write to the operator's current DB automatically.

## Tests/build

Run the WPF build and focused Price Rule/TurnEngine/outbox/inbound-apply tests.

Do not include unrelated migration suites.

Preserve the PostgreSQL-only guard from prompt086:

```text
UseSqlServer active matches = 0
SQL Server direct package refs = 0
production ProviderName = Npgsql.EntityFrameworkCore.PostgreSQL
```

## Physical retest after PASS

1. Rebuild and launch WPF manually.
2. Open Price / Amount Rule Settings with no Draft policy.
3. Confirm `PRICE_RULE_LOAD_EMPTY` without error.
4. Click Add Row or Reset Defaults according to the proven model.
5. Confirm rows appear in memory only.
6. Enter one valid small test rule set approved by the operator.
7. Keep focus in the last edited DataGrid cell and click Save once.
8. Confirm structured committed status and no exception.
9. Close/reopen and verify persistence.
10. Verify expected Price Rule outbox event count safely.
11. Use Test Lookup on boundary amounts and confirm expected matches.
12. Save again without changes and confirm NO_CHANGES/no new outbox.
13. Do not activate a Turn Policy yet.
14. Do not test checkout/payment.

## Safety boundaries

- No automatic mutation of the operator's current DB.
- No automatic WPF launch/clicking.
- No checkout/payment test.
- No Turn Policy creation/activation.
- No Price Rule clean-install seed addition.
- No SQL Server provider/package/config reintroduction.
- No OBM source commit/push; source edits remain local.
- No private identifiers, raw payloads, credentials, or internal code in the public report.

## Private handoff requirements

Return directly to the operator:

1. Exact physical Save failure root cause and inner exception.
2. Exact rule ownership and lookup contract.
3. Exact reason sync contract was previously missing.
4. Complete before/after Save method bodies.
5. Exact transaction boundary and SaveChanges count.
6. Outbox event granularity and payload mapping.
7. WPF receiver apply and no-echo proof.
8. Validation rules.
9. Reset Defaults semantics.
10. Real PostgreSQL integration test code/output.
11. Build/test results.
12. Physical retest steps.
13. Explicit confirmation that checkout and Turn Policy code were unchanged.

## Public report

Create and push only an ultra-minimal:

```text
report/report088.md
```

It may contain only:

```text
verdict
physical Add Row -> Save failure root cause proven yes/no
initialization model A preserved yes/no
save boundary proven yes/no
outbox contract proven yes/no
WPF receiver apply proven yes/no
Test Lookup parity proven yes/no
real PostgreSQL proof yes/no
build/test counts
current DB automatically mutated no
checkout tested no
one aggregate evidence SHA-256
```

## Valid verdicts

```text
OBM_POS_PRICE_RULE_SAVE_SYNC_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_PRICE_RULE_SAVE_ROOT_CAUSE_UNPROVEN
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
