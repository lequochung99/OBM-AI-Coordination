# Prompt 094 — Implement atomic three-table Price Rule save and standard API synchronization

## Operator architecture decision — authoritative

The operator has selected the P2 architecture.

Every non-noop Price Rule Save must participate in one aggregate transaction involving exactly these local persistence tables:

```text
1. TblTurnPolicy
2. TblTurnAmountRule
3. TblLocalOutbox
```

The resulting outbox transaction group must then move through the existing standard OBM synchronization flow:

```text
WPF local PostgreSQL
-> TblLocalOutbox publisher
-> ApiServer ingest
-> durable API event/delivery storage
-> SignalR notification after durable commit
-> destination POS pull
-> ordered local apply
-> no local outbox echo
```

This decision supersedes the P1/P2 ambiguity in `prompt/prompt093.md`.

Do not execute prompt093 separately. Use its investigation requirements and evidence expectations where compatible, but P2 is now fixed and mandatory.

## Current proven state

Prompt092 and the private artifact prove:

```text
Price Rule local Save succeeds
Timestamp mismatch is fixed
TblTurnAmountRule changes commit locally
OutboxRowsStaged remains 0
Root cause = PRICE_RULE_OUTBOX_CALL_MISSING
No Price Rule outbox method is called
No explicit transaction currently exists in the Price Rule service
No proven Price Rule DTO/mapper/API registration/receiver dispatch exists
```

The current physically observed invalid state is:

```text
Committed=True
OutboxRowsStaged=0
```

This task must make that state impossible whenever Price Rule changes exist.

## Required architecture

### Aggregate root

`TblTurnPolicy` is the aggregate parent for `TblTurnAmountRule`.

The child editor is authorized to create the first Draft Policy when the operator explicitly clicks Save.

The child editor must not create a policy merely by opening, loading, clicking Edit, Add Row, Reset Defaults, Test Lookup, or closing.

### First changed Save when no Draft exists

Required local transaction:

```text
create one Draft TblTurnPolicy
-> stage TblTurnPolicy outbox I event
-> stage Price Rule inserts/updates/deletes
-> stage one Price Rule outbox event per changed rule
-> one SaveChangesAsync when possible
-> commit once
```

### Later changed Save when Draft exists

Required local transaction:

```text
update the existing Draft TblTurnPolicy aggregate metadata
-> stage TblTurnPolicy outbox U event
-> stage Price Rule inserts/true updates/deletes
-> stage one Price Rule outbox event per changed rule
-> one SaveChangesAsync when possible
-> commit once
```

Use existing `UpdatedAt`, `UpdatedBy`, version/revision, or equivalent aggregate metadata already present in the schema.

Do not invent a new policy revision column without explicit operator approval. If the existing schema has no safe parent field to update, return a blocker with the complete schema and proposed smallest migration instead of silently faking a policy update.

### No-op Save

```text
no policy write
no rule write
no outbox write
PRICE_RULE_SAVE_NO_CHANGES
```

## Event granularity and ordering

Use one outbox CRUD event per changed entity.

For one transaction group:

```text
TransactionGuid = one stable value for the operator Save
Sequence = deterministic ascending integer
```

Ordering is mandatory:

```text
Sequence 1 = TblTurnPolicy I or U
Sequence 2..N = TblTurnAmountRule I/U/D events
```

Entity contracts:

```text
TblTurnPolicy:
  EntityType = nameof(TblTurnPolicy)
  EntityGuid = TurnPolicyGuid
  Operation = I on first Draft creation, otherwise U

TblTurnAmountRule:
  EntityType = nameof(TblTurnAmountRule)
  EntityGuid = TurnAmountRuleGuid
  Operation = I, U, or D according to true change set
```

For one new rule on the first Save, expected outbox count is:

```text
Policy events = 1
Rule events = 1
OutboxRowsStaged = 2
```

For one existing rule update:

```text
Policy events = 1 U
Rule events = 1 U
OutboxRowsStaged = 2
```

For one rule delete:

```text
Policy events = 1 U
Rule events = 1 D
OutboxRowsStaged = 2
```

For multiple changed rules:

```text
OutboxRowsStaged = 1 policy event + changed rule count
```

## Fail-closed rules

A changed Save must never commit only local domain rows.

Required guards:

```text
changed rules > 0 AND policy outbox events != 1
-> fail before SaveChangesAsync

changed rules > 0 AND Price Rule outbox events != changed rule count
-> fail before SaveChangesAsync

changed rules > 0 AND total staged outbox != 1 + changed rule count
-> PRICE_RULE_OUTBOX_COUNT_MISMATCH
-> rollback

outbox infrastructure unavailable
-> explicit failure
-> rollback policy + rules

payload serialization failure
-> rollback policy + rules

SaveChanges failure
-> rollback all three table mutations
```

A successful changed Save must satisfy:

```text
PolicyRowsChanged=1
RuleRowsChanged>=1
OutboxRowsStaged=1+RuleRowsChanged
Committed=True
```

## Standard API synchronization flow

Do not build a second Price Rule transport.

Use the existing canonical OBM outbox/API/delivery/pull pipeline.

Audit and return complete code for the actual existing components and exact table names:

```text
local TblLocalOutbox publisher
API ingest endpoint/service
API durable event table
API durable delivery table
subscription/entity registration
transaction GUID and sequence persistence
source-client identity envelope
SignalR notification publisher
WPF delivery pull
WPF ApplyDeliveryAsync dispatch
acknowledgement/retry state
```

### API ingest requirements

The API must:

```text
validate tenant, source client, entity type, operation, transaction GUID, and sequence
persist the incoming event durably
create destination deliveries durably
commit event/delivery storage before SignalR notification
preserve transaction group and sequence
exclude the source client from destination delivery when canonical behavior requires it
return an idempotent result for replayed outbox uploads
```

Do not consider SignalR itself the data transport.

SignalR only notifies destination clients that durable deliveries are available.

### Entity registration

Audit whether API relay accepts arbitrary entity names or uses an allowlist/subscription registry.

Add the smallest required support for:

```text
TblTurnPolicy
TblTurnAmountRule
```

Do not modify unrelated entity contracts.

If API support cannot safely carry both parent and child events in order, return:

```text
BLOCKED_PRICE_RULE_API_TRANSACTION_GROUP_CONTRACT
```

with complete C# and schema proof.

## Destination POS apply architecture

The destination POS must process policy before rules.

Preferred model when the existing pull contract exposes transaction groups:

```text
pull one transaction group
-> validate contiguous sequence
-> begin one local PostgreSQL transaction
-> apply TblTurnPolicy event first
-> apply TblTurnAmountRule events in sequence
-> SaveChangesAsync
-> commit
-> acknowledge all deliveries
```

If current delivery APIs return individual events only, prove ordered delivery and implement safe dependency behavior:

```text
policy event applies first
child rule event with missing parent is not acknowledged as success
child is deferred/retried until parent exists
```

Do not insert orphan Price Rules.
Do not discard a child event because the parent has not arrived yet.

### WPF inbound dispatch

Add or prove dispatch cases for:

```text
case nameof(TblTurnPolicy)
case nameof(TblTurnAmountRule)
```

Use the existing `PullDbApiBot.ApplyDeliveryAsync` pipeline.

Do not create a separate TurnEngine synchronization channel.

### Policy receiver

Required behavior:

```text
validate current tenant
I/U idempotent upsert by TenantGuid + TurnPolicyGuid
preserve Draft status/version contract
normalize PostgreSQL local timestamps
replay produces one final row
no local TblLocalOutbox echo
```

Do not activate the policy during inbound apply.

### Price Rule receiver

Required behavior:

```text
validate current tenant
validate TurnAmountRuleGuid and TurnPolicyGuid
verify parent policy exists
I/U idempotent upsert by TenantGuid + TurnAmountRuleGuid
D idempotent delete/deactivate according to proven schema contract
apply false, zero, null, and empty-clear values correctly
preserve decimal precision
normalize local timestamp kinds
replay produces one final state
no local TblLocalOutbox echo
```

Do not overwrite CreatedAt/CreatedBy on an existing row unless the established sync convention requires it.

## Payload contracts

Choose and prove either:

```text
E = direct entity payload, only when already safe and canonical
D = typed DTO with explicit sender/receiver mappers
```

A typed DTO is preferred if direct entity serialization could include navigation state or provider-specific properties.

### TblTurnPolicy payload

Include every required parent field, including:

```text
TurnPolicyGuid
TenantGuid
policy status
policy version/revision fields that actually exist
CreatedAt/CreatedBy
UpdatedAt/UpdatedBy
other required non-null columns
```

### TblTurnAmountRule payload

Include:

```text
TurnAmountRuleGuid
TenantGuid
TurnPolicyGuid
RuleName
MinAmount
MaxAmount
Factor1
Factor2
TurnCredit
SortOrder
IsActive
Notes
CreatedAt
CreatedBy
UpdatedAt
UpdatedBy
```

Prove that payloads preserve these clear/default values:

```text
IsActive=false
SortOrder=0 when valid
MaxAmount=null
TurnCredit=null
Notes=null or empty clear
```

## True change detection

Do not treat every displayed row as dirty.

Determine the final change set by comparing the proposed UI set with reloaded DB rows:

```text
Inserted
Truly updated
Deleted
Unchanged
```

Required behavior:

```text
unchanged row -> no U event
inserted row -> one I event
true update -> one U event
deleted row -> one D event
```

For delete events, capture the canonical payload/key before deleting the entity.

## One local transaction and one SaveChanges

The final sender shape must be:

```text
Commit DataGrid cell/row
-> validate final rule set
-> determine true changes
-> if no changes return
-> verify outbox infrastructure
-> create one DbContext
-> begin explicit PostgreSQL transaction
-> load or create Draft TblTurnPolicy inside this context
-> update parent aggregate metadata
-> stage policy I/U
-> stage rule I/U/D
-> stage ordered TblLocalOutbox events
-> verify counts and sequence
-> one SaveChangesAsync when possible
-> commit once
-> reload
```

`GetOrCreateDraftPolicyAsync` must not use another DbContext or commit independently.

Refactor it or add an overload so Draft lookup/creation occurs inside the Price Rule service's supplied DbContext and transaction.

A reload failure after commit must return:

```text
Committed=True
ReloadSucceeded=False
PRICE_RULE_SAVED_RELOAD_FAILED
```

and must not be reported as an uncommitted Save failure.

## Runtime refresh

Audit whether Price Rule lookup uses a fresh DbContext query or a cache.

After local Save and inbound apply:

```text
refresh/invalidate only the proven runtime owner
```

Do not invent a cache when the runtime already queries PostgreSQL per lookup.

## Required diagnostics

Extend the safe Price Rule result with at least:

```text
PolicyCreated
PolicyUpdated
PolicyRowsChanged
RuleInsertedRows
RuleUpdatedRows
RuleDeletedRows
RuleUnchangedRows
PolicyOutboxRowsStaged
RuleOutboxRowsStaged
OutboxRowsStaged
TransactionGuidPresent
FirstSequence
LastSequence
SaveChangesCallNumber
TransactionStarted
Committed
ReloadSucceeded
ResultCode
```

Do not expose GUID values, payloads, credentials, connection strings, or private rule contents.

## Documentation and evidence gate

Read completely before editing:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report091.md
report/report092.md
prompt/prompt092.md
prompt/prompt093.md
```

Read the full private prompt092 artifact:

```text
E:\Project2026\RecoveryReports\PriceRuleSaveSyncCodeHandoffV001
```

Create a new versioned local artifact, never overwrite prior evidence:

```text
PriceRuleThreeTableApiSyncV001
```

Required files:

```text
PRIVATE_HANDOFF.md
ARCHITECTURE.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
LOCAL_TRANSACTION.md
OUTBOX_CONTRACT.md
API_FLOW.md
POLICY_RECEIVER.md
PRICE_RULE_RECEIVER.md
SCHEMA_DTO_MAPPING.md
TEST_OUTPUT.txt
SHA256SUMS.txt
```

## Mandatory detailed C# handoff

Return complete actual C# method bodies before and after, paths, and line ranges for:

```text
Price Rule Save_Click
DataGrid commit helper
true change-set detection
validation
Draft policy lookup/create inside supplied DbContext
parent policy metadata update
Price Rule insert/update/delete mapping
policy payload mapper
Price Rule payload mapper
outbox staging helper
transaction begin/commit/rollback
all SaveChangesAsync calls
API ingest and registration changes
SignalR notification call site
WPF pull/dispatch
TblTurnPolicy receiver
TblTurnAmountRule receiver
runtime refresh/invalidation
result/diagnostic mapping
```

Also include a unified diff for every changed method.

Do not return only a summary or test counts.

## Tests — direct PostgreSQL, API, and receiver

Docker is not required.

Run build and focused tests. Use approved local PostgreSQL and test API harnesses without exposing credentials.

Prove at least:

```text
FIRST SAVE, NO DRAFT:
- one TblTurnPolicy row created
- one or more TblTurnAmountRule rows created
- one policy I outbox event
- one rule I event per rule
- sequence policy before rules
- one transaction GUID
- one local commit

UPDATE SAVE:
- existing Draft policy updated
- one policy U event
- one rule U event per true changed rule
- unchanged rules emit no event

DELETE SAVE:
- policy U event
- rule D event

NO-OP SAVE:
- zero writes in all three tables
- zero outbox

FAILURE:
- outbox unavailable rolls back policy and rules
- serialization failure rolls back policy and rules
- SaveChanges failure rolls back all three tables
- count mismatch fails before commit

API:
- replayed upload is idempotent
- durable event/delivery commit occurs before SignalR notify
- transaction GUID and sequence preserved
- source client excluded according to standard behavior

POS2 APPLY:
- policy applies before rules
- missing parent defers child rather than acknowledging success
- I/U/D replay idempotent
- wrong tenant rejected
- no local outbox echo
- false/zero/null clear values applied
- offline reconnect resumes pending delivery
```

Do not automatically mutate the operator's current DB.

## Physical acceptance after implementation

### First Save with no Draft

```text
PolicyCreated=True
PolicyRowsChanged=1
RuleInsertedRows=1
PolicyOutboxRowsStaged=1
RuleOutboxRowsStaged=1
OutboxRowsStaged=2
Committed=True
ReloadSucceeded=True
```

Verify safely:

```text
TblTurnPolicy changed rows = 1
TblTurnAmountRule changed rows = 1
TblLocalOutbox new rows = 2
sequence 1 = TblTurnPolicy I
sequence 2 = TblTurnAmountRule I
same transaction group
```

### Update one existing rule

```text
PolicyUpdated=True
PolicyRowsChanged=1
RuleUpdatedRows=1
PolicyOutboxRowsStaged=1
RuleOutboxRowsStaged=1
OutboxRowsStaged=2
Committed=True
```

### Save without changes

```text
PRICE_RULE_SAVE_NO_CHANGES
PolicyRowsChanged=0
RuleRowsChanged=0
OutboxRowsStaged=0
```

Do not test checkout/payment in this task.

## Safety boundaries

- PostgreSQL/Npgsql only.
- Direct Windows WPF.
- Docker not required.
- No automatic current-DB mutation.
- No Price Rule clean-install seed.
- No policy activation.
- No checkout/payment changes or tests.
- No unrelated Queue, BookingConsole, installation, customer, gift-card, terminal, or invoice changes.
- No OBM source commit/push.
- No credentials, raw GUIDs, raw payloads, or private row contents in the public report.

## Public report

Create and push only:

```text
report/report094.md
```

It may contain only:

```text
verdict
three-table local transaction proven yes/no
policy-first outbox ordering proven yes/no
API durable ingest/delivery proven yes/no
SignalR-after-commit proven yes/no
POS receiver parent-child apply proven yes/no
idempotency/no-echo proven yes/no
private detailed C# handoff created yes/no
real PostgreSQL/API proof yes/no
build/test counts
current DB automatically mutated no
checkout tested no
one aggregate evidence SHA-256
```

## Valid verdicts

```text
OBM_POS_PRICE_RULE_THREE_TABLE_API_SYNC_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_PRICE_RULE_POLICY_AGGREGATE_SCHEMA
```

```text
BLOCKED_PRICE_RULE_API_TRANSACTION_GROUP_CONTRACT
```

```text
BLOCKED_PRICE_RULE_RECEIVER_PARENT_CHILD_ORDERING
```

```text
BLOCKED_PRICE_RULE_REAL_POSTGRESQL_API_PROOF
```

```text
BLOCKED_PRICE_RULE_BUILD_OR_TEST
```
