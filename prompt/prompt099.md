# Prompt 099 — Implement atomic Price Rule three-table local Save and ordered transaction-group outbox

## Approved gates

Prompt097 passed:

```text
OBM_WPF_POSTGRESQL_BASELINE_AND_OUTBOX_SCHEMA_READY_FOR_API_SCHEMA
```

Prompt098 passed:

```text
OBM_API_POSTGRESQL_TRANSACTION_GROUP_SCHEMA_READY_FOR_LOCAL_SENDER
```

Therefore the PostgreSQL schema foundations are accepted:

```text
WPF TblLocalOutbox transaction-group schema: proven
API TblEventLog/TblEventDelivery/TblEventDeliveryGroupAck schema: proven
migration-from-zero for both contexts: proven
current operator databases: untouched
```

This task implements only the WPF local sender boundary for the Price Rule aggregate.

## Authoritative local architecture

A non-noop Price Rule Save is one atomic local PostgreSQL transaction involving exactly these domain/outbox tables:

```text
1. TblTurnPolicy
2. TblTurnAmountRule
3. TblLocalOutbox
```

The transaction-group contract is:

```text
one TransactionGuid per operator Save
one common TenantGuid
one common SourceClientId
one common ExpectedEventCount
contiguous SequenceNumber values 1..N
Sequence 1 = TblTurnPolicy I or U
Sequence 2..N = TblTurnAmountRule I/U/D events
one EventGuid per outbox row
one real non-empty EntityGuid per outbox row
```

This task must make the old invalid state impossible:

```text
Committed=True
changed Price Rules > 0
OutboxRowsStaged=0
```

## Strict scope

Implement and prove:

```text
Price Rule UI commit/validation boundary
true local change detection
Draft policy create/update inside the same DbContext
policy and rule typed payload mapping
ordered TblLocalOutbox staging
one explicit PostgreSQL transaction
one SaveChangesAsync when technically possible
fail-closed count/sequence validation
local reload/result diagnostics
focused disposable PostgreSQL tests
```

Do not implement or modify in this task:

```text
WPF outbox uploader HTTP behavior
API sync-transaction-group endpoint
API ingest service
API event/delivery runtime writes
SignalR publishing
API group pull
WPF inbound group apply
API group ACK endpoint
current operator DB schema or rows
BookingConsole runtime
checkout/payment runtime
unrelated outbox producers
```

No network call is required for PASS.

## Clean-development-data policy

The operator has declared existing development data disposable.

However, do not drop, recreate, migrate, reseed, or mutate the current operator database in this task.

Use a new separately named disposable PostgreSQL WPF database created from the accepted prompt097 migration baseline.

Do not add backfill or compatibility logic for old incomplete Price Rule/outbox rows.

## Required evidence to read before editing

Read completely:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
prompt/prompt091.md when present
prompt/prompt092.md
prompt/prompt093.md
prompt/prompt094.md
prompt/prompt095.md
prompt/prompt097.md
prompt/prompt098.md
report/report091.md
report/report092.md
report/report094.md
report/report095.md
report/report097.md
report/report098.md
```

Read all relevant files from these private artifacts:

```text
E:\Project2026\RecoveryReports\PriceRuleSaveSyncCodeHandoffV001
E:\Project2026\RecoveryReports\PriceRuleThreeTableApiSyncV001 when present
E:\Project2026\RecoveryReports\CleanTransactionGroupSyncV001
E:\Project2026\RecoveryReports\WpfPostgreSqlMigrationBaselineV001
E:\Project2026\RecoveryReports\ApiPostgreSqlTransactionGroupSchemaV001
```

At minimum, read complete actual source evidence for:

```text
PriceWeightSettingWindow Save_Click and grid commit
TurnAmountRuleSettingService load/save/diagnostics
TurnPolicySetupService Draft lookup/create
TblTurnPolicy entity and EF mapping
TblTurnAmountRule entity and EF mapping
TblLocalOutbox final prompt097 entity and mapping
existing Employee Weight atomic save/outbox pattern
existing outbox enqueue helpers
accepted WPF migration executor and disposable DB harness
```

Record before first edit:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
TASK_SCOPE=PRICE_RULE_LOCAL_SENDER_ONLY
LOCAL_TABLES=TblTurnPolicy,TblTurnAmountRule,TblLocalOutbox
NETWORK_SYNC=DEFERRED
PROVIDER=NPGSQL_ONLY
CURRENT_OPERATOR_DB_MUTATION=FORBIDDEN
```

## Proven starting defect

The prior Price Rule Save path:

```text
loads or creates Draft policy outside the Price Rule DbContext boundary
stages TblTurnAmountRule changes
never stages TblLocalOutbox
calls SaveChangesAsync
can return Committed=True with OutboxRowsStaged=0
```

Root classification:

```text
PRICE_RULE_OUTBOX_CALL_MISSING
```

The prior timestamp mismatch is already fixed and must not regress.

Do not re-audit this root cause from zero. Replace the invalid local transaction boundary.

## Phase 1 — Audit exact policy/rule schema and authorization

Return complete actual schema/model proof for:

```text
TblTurnPolicy primary key
TenantGuid
status field and exact Draft value
required version/revision fields that already exist
CreatedAt/CreatedBy
UpdatedAt/UpdatedBy
all other required non-null properties
unique constraints that control one Draft per tenant

TblTurnAmountRule primary key
TenantGuid
TurnPolicyGuid FK
all rule value fields
created/updated audit fields
hard-delete versus soft-delete contract
```

The child Price Rule editor is authorized to create the first Draft only when the operator explicitly clicks Save and there is at least one true Price Rule change.

The following actions must never create/update policy, rule, or outbox rows:

```text
open editor
load
click Edit
Add Row before Save
Reset Defaults before Save
Test Lookup
close/cancel
Save with no true changes
```

If there is an existing Draft, use it.

If more than one Draft exists for one tenant, fail closed with a narrow result code; do not guess which parent to use.

## Phase 2 — One DbContext and one explicit transaction

Refactor Draft policy resolution so the Price Rule service can load or create the Draft using the same supplied `eNailSalonDbContext` and the same explicit PostgreSQL transaction used for rules and outbox.

Required shape:

```text
Commit focused DataGrid edit
Validate final UI set
Resolve TenantGuid and SourceClientId
Create one eNailSalonDbContext
Begin explicit transaction
Reload current Draft policy and current rule set
Detect true change set
If no changes: rollback/no mutation and return PRICE_RULE_SAVE_NO_CHANGES
Create or update Draft parent inside this context
Stage parent I/U event at sequence 1
Stage child I/U/D entity changes
Stage one child outbox event per true change at sequences 2..N
Validate expected count and contiguous sequence
SaveChangesAsync once when possible
Commit once
Reload using a fresh read boundary
```

`GetOrCreateDraftPolicyAsync` must not commit through another context.

Add an overload/helper accepting the supplied context, or replace the hidden child-editor call with an explicit internal method owned by the local aggregate service.

Do not call `SaveChangesAsync` from a policy helper.

Do not stage outbox through a helper that creates another DbContext or commits independently.

## Phase 3 — True change detection

Do not treat every displayed existing row as dirty.

Compare the final proposed UI set against the reloaded current tenant/Draft rule set and classify exactly:

```text
Inserted
TrulyUpdated
Deleted
Unchanged
```

Comparison must cover every persisted business field:

```text
RuleName
MinAmount
MaxAmount
Factor1
Factor2
TurnCredit
SortOrder
IsActive
Notes
and any other actual persisted Price Rule field
```

Normalize only according to the established UI/database contract.

Required semantics:

```text
unchanged existing row -> no entity update, no rule U event
new row -> one rule I event
changed existing row -> one rule U event
deleted existing row -> one rule D event
```

For delete, capture the canonical typed payload/key before removing the entity.

Assign real new `TurnAmountRuleGuid` values before deterministic event ordering.

Reject UI rows that claim an existing key but do not belong to the current tenant/current Draft.

## Phase 4 — Parent policy mutation contract

### First changed Save with no Draft

Create exactly one Draft `TblTurnPolicy` row.

Stage exactly one parent outbox event:

```text
EntityType = nameof(TblTurnPolicy)
EntityGuid = TurnPolicyGuid
Operation = I
SequenceNumber = 1
```

### Later changed Save with existing Draft

Update only proven aggregate/audit metadata on the existing Draft, for example actual existing `UpdatedAt`, `UpdatedBy`, or existing revision/version fields.

Do not invent a new policy revision column in this task.

Stage exactly one parent event:

```text
EntityType = nameof(TblTurnPolicy)
EntityGuid = TurnPolicyGuid
Operation = U
SequenceNumber = 1
```

Do not activate or publish the policy.

Do not change unrelated policy settings.

If the actual schema has no legitimate parent field that may change for a child aggregate Save, it is still permissible to stage a parent U snapshot only if the existing sync/event convention explicitly allows unchanged parent snapshot events. Prove that convention. Otherwise stop with:

```text
BLOCKED_PRICE_RULE_PARENT_UPDATE_CONTRACT
```

Do not fake a parent mutation merely to satisfy EF change tracking.

## Phase 5 — Typed payload contracts

Use typed DTO payloads with explicit sender mapping unless the existing accepted project contract already has a navigation-free canonical DTO.

Do not serialize tracked EF entities with navigation properties.

### TurnPolicy payload

Include every actual field required for idempotent future receiver upsert, including:

```text
TurnPolicyGuid
TenantGuid
status/Draft value
existing version/revision fields
required policy configuration fields
CreatedAt
CreatedBy
UpdatedAt
UpdatedBy
```

Do not include secrets, navigation collections, DbContext state, or unrelated tenant data.

### TurnAmountRule payload

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

Use the actual schema types and field names.

Prove preservation of:

```text
IsActive=false
numeric zero values
MaxAmount=null
TurnCredit=null
Notes=null
Notes="" when empty is distinct by contract
full decimal precision
PostgreSQL local timestamp policy
```

Use deterministic JSON serialization settings compatible with the planned API/runtime contract.

## Phase 6 — Ordered grouped TblLocalOutbox staging

All rows from one operator Save must use:

```text
same TenantGuid
same SourceClientId
same TransactionGuid
same ExpectedEventCount
unique EventGuid
non-empty EntityGuid
SequenceNumber 1..N
Sent = Pending canonical value
AttemptCount = 0
no claim owner
no sent timestamp
```

Expected count:

```text
ExpectedEventCount = 1 parent event + true changed rule count
```

Required ordering:

```text
Sequence 1 = parent policy I/U
Sequence 2..N = child rule events
```

Child event ordering must be deterministic.

Choose and document one stable ordering using actual change data, such as:

```text
operation rank
then effective SortOrder
then TurnAmountRuleGuid
```

All tests and diagnostics must use the same ordering contract.

Each child event:

```text
EntityType = nameof(TblTurnAmountRule)
EntityGuid = TurnAmountRuleGuid
Operation = I, U, or D
```

Use a staging helper that accepts the current DbContext and does not call SaveChanges.

Do not use a generic helper if it creates another context, assigns incompatible transaction metadata, or commits independently.

## Phase 7 — Fail-closed integrity gates

Before `SaveChangesAsync`, validate in memory and through tracked entities:

```text
changed rule count >= 1
parent event count = 1
rule event count = changed rule count
total outbox count = 1 + changed rule count
all ExpectedEventCount values = total outbox count
one TransactionGuid only
one TenantGuid only
one SourceClientId only
sequences exactly 1..N
sequence 1 is TblTurnPolicy I/U
all child sequences are TblTurnAmountRule I/U/D
all EventGuid values non-empty and unique
all EntityGuid values non-empty
all child TurnPolicyGuid values equal the parent key
```

On mismatch:

```text
PRICE_RULE_OUTBOX_COUNT_MISMATCH
PRICE_RULE_OUTBOX_SEQUENCE_INVALID
PRICE_RULE_OUTBOX_METADATA_MISMATCH
or another narrow proven code
Committed=False
```

No domain row may commit when outbox staging/serialization/validation fails.

If `SaveChangesAsync` or commit fails, rollback all three table changes.

## Phase 8 — SaveChanges/commit/reload semantics

Target transaction boundary:

```text
BeginTransactionAsync
stage policy/rules/outbox
SaveChangesAsync call #1
CommitAsync
```

Use one `SaveChangesAsync` unless the real identity-generation contract makes one additional save unavoidable. A second save requires explicit proof and must remain inside the same transaction; do not weaken atomicity.

Set `Committed=True` only after `CommitAsync` succeeds.

Reload failure after commit must return:

```text
PRICE_RULE_SAVED_RELOAD_FAILED
Committed=True
ReloadSucceeded=False
```

A pre-commit failure must return:

```text
Committed=False
ReloadSucceeded=False
```

Do not report a committed transaction as failed merely because UI reload failed.

## Phase 9 — UI boundary

Keep the Price Rule UI behavior clear and non-mutating until Save.

`Save_Click` must:

```text
commit current DataGrid cell/row edit
validate rows
call exactly one canonical local sender service method
show a safe success/no-change/failure result
write a versioned local diagnostic artifact
not call API directly
```

Do not retain a production Save method that bypasses the grouped outbox contract.

Either remove/deprecate the old unsafe method or route all production callers through the new canonical method.

No successful UI message is allowed when:

```text
changed rules > 0
Committed=False
outbox count mismatch
```

## Required diagnostics

Return a safe result containing at least:

```text
ResultCode
StageId
TenantResolved
SourceClientResolved
PolicyCreated
PolicyUpdated
PolicyRowsChanged
RuleInsertedRows
RuleUpdatedRows
RuleDeletedRows
RuleUnchangedRows
ChangedRuleRows
PolicyOutboxRowsStaged
RuleOutboxRowsStaged
OutboxRowsStaged
ExpectedEventCount
TransactionGuidPresent
FirstSequence
LastSequence
TransactionStarted
SaveChangesCallNumber
Committed
ReloadSucceeded
ExceptionType
InnerExceptionType
PostgreSqlState
SafeTable
SafeConstraint
```

Do not expose actual GUID values, rule contents, payload JSON, connection strings, or credentials in UI/report diagnostics.

## Phase 10 — Disposable PostgreSQL physical proof

Create a new versioned disposable WPF PostgreSQL database.

Apply the accepted prompt097 WPF migrations from zero.

Seed only the minimal identity/setup required to execute the Price Rule local sender:

```text
one tenant identity
one SourceClientId/POS identity
minimal required policy dependencies
no historical business transactions
```

Do not use or mutate the operator's current DB.

### Case A — First Save, no Draft, one new rule

Prove in one committed transaction:

```text
TblTurnPolicy rows created = 1
TblTurnAmountRule rows created = 1
TblLocalOutbox rows created = 2
Sequence 1 = TblTurnPolicy I
Sequence 2 = TblTurnAmountRule I
ExpectedEventCount on both = 2
same TransactionGuid
same TenantGuid
same SourceClientId
all EventGuid/EntityGuid non-empty
Committed=True
ReloadSucceeded=True
```

### Case B — Existing Draft, true update to one rule

Prove incremental result:

```text
policy rows created = 0
one parent policy U outbox
one rule U outbox
OutboxRowsStaged = 2
one common new TransactionGuid for this Save
no duplicate rule row
```

### Case C — Insert one additional rule

Prove:

```text
one parent U event
one rule I event
OutboxRowsStaged = 2
```

### Case D — Delete one rule

Prove:

```text
one parent U event
one rule D event
OutboxRowsStaged = 2
rule deletion semantics match proven schema contract
```

### Case E — Mixed change set

At minimum:

```text
one insert
one true update
one delete
one unchanged row
```

Prove:

```text
changed rule count = 3
unchanged count = 1
parent events = 1
rule events = 3
OutboxRowsStaged = 4
sequences exactly 1..4
```

### Case F — No-op Save

Prove:

```text
PRICE_RULE_SAVE_NO_CHANGES
policy writes = 0
rule writes = 0
outbox writes = 0
transaction group not created
```

### Case G — Serialization/outbox failure rollback

Use a controlled test seam that fails before commit without modifying production behavior.

Prove:

```text
policy delta = 0
rule delta = 0
outbox delta = 0
Committed=False
```

### Case H — Database constraint failure rollback

Induce one safe group constraint/unique failure in a disposable transaction.

Prove all three table mutations rollback.

### Case I — Timestamp/null/default fidelity

Prove round-trip payload and persisted entity values for:

```text
false
zero
null decimal values
null/empty notes according to contract
PostgreSQL timestamp policy
```

After evidence capture, drop only the disposable proof database.

## Phase 11 — Focused tests

Add focused tests for:

```text
opening/loading editor causes zero mutations
first Save creates Draft + rule + two ordered outbox rows
existing Draft update creates policy U + rule U events
insert event
update event
delete event
mixed change ordering
unchanged row produces no U event
no-op Save produces no writes
missing tenant fails closed
missing SourceClientId fails closed
multiple Draft policies fail closed
foreign/current-tenant rule validation
outbox count mismatch rollback
serialization failure rollback
SaveChanges failure rollback
reload failure distinguished after commit
typed payload false/zero/null fidelity
all timestamps are PostgreSQL-compatible
one transaction and expected SaveChanges count
current operator DB is never selected
```

Use direct disposable PostgreSQL tests for transaction/constraint claims.

No soft skips are allowed when local PostgreSQL prerequisites are available.

Build success alone is not PASS.

## Required private evidence artifact

Create a new versioned local artifact. Never overwrite earlier evidence.

Suggested folder:

```text
E:\Project2026\RecoveryReports\PriceRuleLocalSenderV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ARCHITECTURE.md
POLICY_CONTRACT.md
RULE_CHANGE_DETECTION.md
TYPED_PAYLOADS.md
OUTBOX_ORDERING.md
LOCAL_TRANSACTION.md
UI_BOUNDARY.md
RESULT_DIAGNOSTICS.md
DISPOSABLE_DB_PROOF.md
CASE_MATRIX.md
TEST_OUTPUT.txt
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

## Mandatory complete private code handoff

Return complete actual C# code, not excerpts, for:

```text
PriceWeightSettingWindow Save_Click
DataGrid commit helper
row validation
canonical Price Rule local sender method
true change-set detector
Draft policy lookup/create using supplied DbContext
parent policy update logic
rule entity insert/update/delete mapping
TurnPolicy typed DTO and mapper
TurnAmountRule typed DTO and mapper
JSON serialization helper/settings
TblLocalOutbox staging helper
in-memory group integrity validator
transaction begin/save/commit/rollback code
reload code
result/diagnostic class and mapping
all focused test methods
```

Include:

```text
repository-relative path
line ranges
complete BEFORE body
complete AFTER body
actual unified diff
actual sanitized SQL/query evidence
```

Do not expose credentials, full connection strings, passfile contents, actual tenant/device/rule GUIDs, payload JSON values, or private business data.

## Public report

Create and push only:

```text
report/report099.md
```

The public report must be redacted and minimal.

Include:

```text
Verdict
One-DbContext local boundary yes/no
Explicit PostgreSQL transaction yes/no
First-Save Draft creation proven yes/no
True I/U/D/no-op detection proven yes/no
Policy-first outbox ordering proven yes/no
Three-table atomic commit proven yes/no
Rollback proofs yes/no
Typed payload fidelity proven yes/no
Disposable PostgreSQL case matrix totals
Focused test totals
WPF build errors/warnings
Current operator DB mutated yes/no
Network/API called yes/no
Private evidence artifact yes/no
Aggregate SHA-256
```

Do not publish source code, payloads, SQL containing private identifiers, credentials, local secret paths, or business row contents.

## Source and coordination repository rules

All OBM source changes remain local/private.

Do not commit or push OBM source code to the coordination repository.

Commit and push only:

```text
report/report099.md
```

Preserve unrelated dirty source changes.

Do not reset, clean, checkout, or overwrite unrelated work.

## Scope exclusions

Must remain behaviorally unchanged:

```text
checkout/payment
invoice creation/settlement
terminal/Dejavoo
gift card
customer/booking
BookingConsole
WPF installation Phase 1
accepted WPF/API migration baselines
Employee Weight
Weird Tip
TurnEngine calculation logic
outbox uploader/runtime network transport
API runtime sync
current operator databases
```

## Final verdicts

PASS only when all local sender physical gates pass:

```text
OBM_PRICE_RULE_LOCAL_TRANSACTION_GROUP_READY_FOR_UPLOADER_API_INGEST
```

Use a narrow blocker otherwise, such as:

```text
BLOCKED_PRICE_RULE_PARENT_UPDATE_CONTRACT
BLOCKED_PRICE_RULE_TRUE_CHANGE_DETECTION
BLOCKED_PRICE_RULE_TYPED_PAYLOAD_CONTRACT
BLOCKED_PRICE_RULE_LOCAL_TRANSACTION
BLOCKED_PRICE_RULE_OUTBOX_GROUP_INTEGRITY
BLOCKED_PRICE_RULE_DISPOSABLE_DB_PROOF
```

Do not implement uploader/API runtime inside this task.
