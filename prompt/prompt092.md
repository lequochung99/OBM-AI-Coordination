# Prompt 092 — Mandatory C# code handoff and closure of Price Rule outbox/receiver sync

## Physical result — authoritative

The operator rebuilt and physically retested the direct Windows WPF runtime after the timestamp correction.

The Price Rule local Save now succeeds:

```text
PRICE_RULE_SAVE_DIAGNOSTIC_COMMITTED
StageId=ReloadAfterSave
NewRows=1
ValidatedRows=1
OutboxRowsStaged=0
Committed=True
ReloadSucceeded=True
ExceptionType=<none>
InnerExceptionType=<none>
PostgreSqlState=<none>
```

This proves:

```text
Add Row works
validation works
PostgreSQL insert works
timestamp mismatch is fixed
local transaction commits
reload/persistence works
```

It also proves the remaining defect:

```text
OutboxRowsStaged=0
```

Therefore local Price Rule persistence is closed, but Price Rule synchronization is not closed.

## Operator escalation — mandatory detailed reporting

The operator has received several high-level reports with similar wording while the physical behavior remained broken.

For this task, a short verdict/count-only handoff is unacceptable.

The private handoff must contain actual C# code, exact call chains, line ranges, schema/DTO mappings, transaction ownership, and unified diffs. The public GitHub report must remain ultra-minimal, but the detailed private/local report is mandatory.

Use evidence level:

```text
EVIDENCE_ESCALATION=100_PERCENT_DIRECT_CODE_AND_RUNTIME_PROOF
```

## Scope

1. Explain exactly why a committed Price Rule Save staged zero outbox rows.
2. Prove the current Save transaction boundary with actual code.
3. Implement the canonical Price Rule outbox sender path through the existing outbox framework.
4. Implement or complete the WPF receiver/apply path through the existing inbound framework.
5. Prove idempotent insert/update/delete and no local outbox echo.
6. Preserve PostgreSQL/Npgsql-only runtime.
7. Preserve initialization model A: no automatic Price Rule seed.
8. Do not test checkout/payment.
9. Do not create or activate a Turn Policy.
10. Do not automatically mutate the operator's current DB; later operator-triggered WPF Save is the only authorized current-DB mutation.
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
report/report088.md
report/report091.md
prompt/prompt090.md
prompt/prompt091.md
```

Read all local Price Rule diagnostic/evidence artifacts from prompts 085–091.

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
RUNTIME_MODE=DIRECT_WINDOWS_WPF_LOCAL_POSTGRESQL
DOCKER_REQUIRED=NO
EVIDENCE_ESCALATION=100_PERCENT_DIRECT_CODE_AND_RUNTIME_PROOF
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Required local evidence artifact

Create a new versioned local folder, never overwriting prior evidence, for example:

```text
PriceRuleSaveSyncCodeHandoffV001
```

Required files:

```text
PRIVATE_HANDOFF.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SCHEMA_DTO_MAPPING.md
TRANSACTION_BOUNDARY.md
TEST_OUTPUT.txt
SHA256SUMS.txt
```

Do not push these detailed files to GitHub.

## Phase A — freeze and report the current successful local Save code

Before changing sync behavior, return complete current method bodies with repository-relative path and line range for:

1. Price Rule Save button/click handler.
2. Focused DataGrid binding/cell/row commit helper.
3. New/dirty/deleted row detection.
4. Validation methods.
5. Tenant and policy/scope resolution.
6. Current production Price Rule Save service.
7. Insert/update/delete entity mapping.
8. Timestamp assignment.
9. Transaction begin/commit/rollback.
10. Every `SaveChangesAsync` call.
11. Diagnostic result construction.
12. Reload-after-save.
13. Exception/result mapping.

Do not provide isolated excerpts. Include complete control-flow blocks.

The handoff must show exactly where the physical result values are assigned:

```text
NewRows=1
ValidatedRows=1
OutboxRowsStaged=0
Committed=True
ReloadSucceeded=True
```

## Phase B — prove exactly why OutboxRowsStaged is zero

Trace every branch from the Save service to any outbox method.

Classify the proven reason using one narrow code:

```text
PRICE_RULE_OUTBOX_CALL_MISSING
PRICE_RULE_OUTBOX_CALL_AFTER_COMMIT
PRICE_RULE_OUTBOX_FACTORY_NULL_SILENT_SKIP
PRICE_RULE_OUTBOX_ENTITY_NOT_REGISTERED
PRICE_RULE_DTO_MAPPER_MISSING
PRICE_RULE_OPERATION_BRANCH_SKIPPED_NEW_ROWS
PRICE_RULE_OUTBOX_PAYLOAD_SERIALIZATION_SKIPPED
PRICE_RULE_OUTBOX_RESULT_COUNTER_WRONG
PRICE_RULE_OUTBOX_INTENTIONALLY_UNSUPPORTED
OTHER_PROVEN_CAUSE
```

Return the actual C# branch that produced `0`.

Do not infer from counters alone.

The private handoff must answer:

```text
Was an outbox method called? yes/no
If called, which method?
What arguments were passed?
Did a factory/repository resolve?
Did entity registration/mapping exist?
Did the method add a TblLocalOutbox entity to the same DbContext?
Was SaveChanges called before or after outbox staging?
Was any exception swallowed?
Why did the final count equal zero?
```

## Phase C — prove the canonical sync contract

Trace the existing architecture end to end:

```text
local Price Rule insert/update/delete
-> canonical TblLocalOutbox
-> DTO/payload
-> API/relay entity contract
-> WPF inbound delivery dispatch
-> receiver/apply handler
-> local Price Rule table
-> runtime Price Rule cache/state refresh
```

Do not create a second outbox or TurnEngine-specific transport.

### Required schema/DTO matrix

Return this matrix with actual names and types in the private handoff:

| Meaning | Entity property | DB column | C# type | PostgreSQL type | DTO field | Payload field | Receiver field |
|---|---|---|---|---|---|---|---|
| Rule ID | | | | | | | |
| Tenant | | | | | | | |
| Rule Name | | | | | | | |
| Min Amount | | | | | | | |
| Max Amount | | | | | | | |
| Factor1 | | | | | | | |
| Factor2 | | | | | | | |
| Turn Credit | | | | | | | |
| Sort Order | | | | | | | |
| Active | | | | | | | |
| Notes | | | | | | | |
| Created/Updated timestamps | | | | | | | |

Also prove:

```text
entity name used in outbox
operation codes for insert/update/delete
stable business/entity key
transaction GUID/sequence behavior
idempotency behavior
payload serialization behavior for null/default values
wrong-tenant rejection
```

If sender or receiver contract is absent, implement the smallest additive path through the existing generic infrastructure. If the API/relay contract cannot safely carry the entity, return a blocker with the exact missing boundary.

## Phase D — canonical atomic Save boundary

The final local operator Save must follow:

```text
commit focused DataGrid cell/row
-> calculate new/dirty/deleted rows
-> no changes: zero DB/outbox writes
-> validate complete final rule set
-> resolve current tenant/scope
-> begin one PostgreSQL transaction
-> reload current tenant rules
-> apply inserts/updates/deletes
-> build canonical DTO/payload for every intended operation
-> add expected TblLocalOutbox row(s) to the SAME DbContext/transaction
-> one SaveChangesAsync when possible
-> commit once
-> reload
```

Required guarantees:

```text
rule mutation and outbox mutation commit together
outbox unavailable => explicit Save failure + rollback
no silent OutboxRowsStaged=0 success when changes exist
no partial insert/update/delete
reload failure after commit is reported separately
```

Choose and prove one canonical event granularity:

```text
A. one outbox event per changed rule
or
B. one aggregate rule-set replacement event per operator Save
```

Do not mix models.

The expected physical result for one new rule must be explicit, for example under model A:

```text
NewRows=1
ValidatedRows=1
OutboxRowsStaged=1
Committed=True
ReloadSucceeded=True
```

## Phase E — WPF receiver/apply

Implement or complete the inbound apply path.

Required behavior:

```text
incoming Price Rule insert/update/delete
-> validate tenant/envelope
-> deserialize canonical payload
-> apply idempotently by stable key
-> preserve PostgreSQL decimal/timestamp types
-> no duplicate row on replay
-> no local TblLocalOutbox echo
-> refresh runtime rule state/cache
```

Return complete receiver C# method body and dispatch registration, with path and line ranges.

Prove:

```text
insert replay = one final row
update replay = one final updated row
delete replay = absent/inactive according to proven contract
wrong tenant rejected
receiver outbox writes = 0
```

## Phase F — mandatory BEFORE/AFTER code report

The private handoff must include complete BEFORE and AFTER method bodies for every changed method, including:

```text
Save click
Save service
outbox builder/enqueue
DTO mapper
inbound dispatch
receiver apply
runtime refresh
```

Also include a unified diff for all changed methods.

A summary such as “outbox added” or “receiver implemented” is not sufficient.

## Phase G — direct Windows and PostgreSQL proof

Docker is not used or required.

Run focused tests and a real PostgreSQL proof through an approved local disposable database/harness when available. If credential automation is unavailable, create an operator-run package using interactive `psql -W` without exposing credentials.

Prove at least:

```text
new rule Save -> rule + expected outbox commit
update rule Save -> rule + expected outbox commit
delete rule Save -> delete/inactive + expected outbox commit
outbox failure -> rule rollback
no-op Save -> zero writes
receiver apply reproduces final state
receiver replay idempotent
receiver no echo
wrong tenant rejected
focused cell committed before Save
PostgreSQL-only provider guard remains PASS
```

Do not write automatically to the operator's current DB.

## Phase H — physical retest contract

After source/test proof, instruct the operator to:

```text
1. Rebuild and run direct Windows WPF.
2. Open Price / Amount Rule Settings.
3. Add one valid rule.
4. Save once.
5. Confirm:
   NewRows=1
   ValidatedRows=1
   OutboxRowsStaged=<expected nonzero count>
   Committed=True
   ReloadSucceeded=True
6. Close/reopen and verify persistence.
7. Verify the matching TblLocalOutbox row safely.
8. Save without changes and verify zero new outbox.
9. Do not test checkout/payment.
```

## Safety boundaries

- PostgreSQL/Npgsql only.
- No Docker requirement.
- No automatic mutation of the operator's current DB.
- No checkout/payment test.
- No Turn Policy creation/activation.
- No Price Rule clean-install seed.
- No OBM source commit/push.
- No credentials, connection strings, raw GUIDs, raw payloads, or private rule values in GitHub report.

## Private handoff requirements — mandatory

Return directly to the operator:

1. Exact reason `OutboxRowsStaged=0`.
2. Complete actual C# code before and after.
3. Exact transaction boundary.
4. Exact SaveChangesAsync count.
5. Exact outbox event granularity.
6. Exact DTO/payload mapping.
7. Complete WPF receiver/apply code.
8. No-echo and idempotency proof.
9. Test commands and outputs.
10. Physical retest steps.
11. Explicit list of unrelated paths unchanged.
12. Local evidence artifact path and SHA-256.

Do not return only a verdict and test counts.

## Public report

Create and push only an ultra-minimal:

```text
report/report092.md
```

It may contain only:

```text
verdict
OutboxRowsStaged zero root cause proven yes/no
atomic save/outbox proven yes/no
receiver apply proven yes/no
private C# handoff created yes/no
real PostgreSQL proof yes/no
build/test counts
current DB automatically mutated no
checkout tested no
one aggregate evidence SHA-256
```

## Valid verdicts

```text
OBM_POS_PRICE_RULE_OUTBOX_RECEIVER_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_PRICE_RULE_OUTBOX_CONTRACT_MISSING
```

```text
BLOCKED_PRICE_RULE_RECEIVER_APPLY_CONTRACT
```

```text
BLOCKED_PRICE_RULE_REAL_POSTGRESQL_PROOF
```

```text
BLOCKED_PRICE_RULE_BUILD_OR_TEST
```
