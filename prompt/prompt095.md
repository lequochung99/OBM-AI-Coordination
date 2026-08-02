# Prompt 095 — Build the clean-slate transaction-group sync contract on fresh POS1, API, and POS2 databases

## Operator clean-slate decision — authoritative

The current databases are development databases. Existing development data does not need to be preserved.

The operator has decided:

```text
Do not let legacy development rows, old outbox records, or backward compatibility block the canonical synchronization architecture.
Focus on the correct forward architecture.
After implementation succeeds, the current development databases will be deleted and recreated from zero for physical verification.
```

Therefore this task explicitly permits:

```text
new PostgreSQL schema columns
new indexes and constraints
new WPF migrations
new ApiServer migrations
breaking changes to obsolete development-only sync contracts
removal of dead compatibility branches
fresh-database-only validation
```

This task does not require:

```text
backfilling old development events
preserving old development outbox rows
supporting mixed old/new transaction-group payloads
maintaining compatibility with disposable dev data
migrating manually created Price Rules from the current dev DB
```

Do not automatically drop or mutate the operator's current databases during implementation. Build and prove the new architecture on fresh, separately named disposable databases. At the end, produce operator-run reset scripts for the canonical dev databases, but do not run those scripts automatically.

## Supersedes the prompt094 blocker

Prompt094 returned:

```text
BLOCKED_PRICE_RULE_API_TRANSACTION_GROUP_CONTRACT
```

The blocker existed because the current API/pull/ack contract did not prove atomic ordered transaction groups.

The operator now authorizes implementing the missing canonical contract instead of preserving the old development behavior.

Use the complete prompt094 private evidence artifact as the source audit, but do not stop merely because a migration or protocol change is required.

## Canonical architecture — fixed

A changed Price Rule Save is one aggregate transaction involving these local persistence tables:

```text
1. TblTurnPolicy
2. TblTurnAmountRule
3. TblLocalOutbox
```

The end-to-end synchronization unit is one transaction group:

```text
TransactionGuid
Sequence 1..N
ExpectedEventCount=N
TenantGuid
SourceClientId
```

Required ordering for Price Rule aggregate changes:

```text
Sequence 1 = TblTurnPolicy I or U
Sequence 2..N = TblTurnAmountRule I/U/D events
```

The complete canonical flow is:

```text
POS1 local PostgreSQL transaction
-> complete TblLocalOutbox transaction group
-> upload the whole group to ApiServer in one request
-> API validates and durably persists the whole group plus all destination deliveries in one transaction
-> API commits
-> SignalR notification after commit
-> POS2 pulls one complete transaction group
-> POS2 applies the entire group inside one local PostgreSQL transaction
-> parent policy first, child rules next
-> POS2 commits
-> POS2 acknowledges the complete group
```

No component may acknowledge or mark only a prefix of a transaction group as complete.

## Required local sender behavior

### First changed Save with no Draft

One explicit local transaction must:

```text
create one Draft TblTurnPolicy
stage TblTurnPolicy I outbox event at sequence 1
stage Price Rule I/U/D entity changes
stage one TblTurnAmountRule outbox event per true change at sequence 2..N
verify group count and contiguous sequence
call SaveChangesAsync once when possible
commit once
```

### Later changed Save with existing Draft

One explicit local transaction must:

```text
update existing Draft TblTurnPolicy aggregate metadata
stage TblTurnPolicy U event at sequence 1
stage true Price Rule I/U/D changes
stage one rule outbox event per true change
verify group count and contiguous sequence
call SaveChangesAsync once when possible
commit once
```

### No-op Save

```text
no policy write
no rule write
no outbox write
PRICE_RULE_SAVE_NO_CHANGES
```

### Fail-closed

A changed Save must fail before commit when:

```text
policy event count != 1
rule event count != true changed rule count
total event count != 1 + changed rule count
sequence is not contiguous
outbox infrastructure is unavailable
payload serialization fails
transaction-group metadata is incomplete
```

The state below must become impossible:

```text
Committed=True
OutboxRowsStaged=0
```

## Canonical local outbox transaction-group schema

Audit the existing `TblLocalOutbox` model. Reuse equivalent existing fields when they already satisfy the contract. Otherwise add the smallest clean migration.

The canonical concepts must be persisted, not inferred only in memory:

```text
LocalOutboxGuid or existing stable PK
TenantGuid
SourceClientId or canonical source-client envelope reference
TransactionGuid
Sequence
ExpectedEventCount
EntityType
EntityGuid
Operation
PayloadJson or canonical payload column
CreatedAt
Status
AttemptCount
NextAttemptAt or equivalent retry field
LastError or safe error state when canonical
SentAt/CompletedAt when canonical
```

Required constraints/indexes:

```text
unique TenantGuid + SourceClientId + TransactionGuid + Sequence
Sequence >= 1
ExpectedEventCount >= 1
Sequence <= ExpectedEventCount
index for pending-group selection
index for retry scheduling
```

Do not create redundant columns when equivalent canonical fields already exist.

Do not implement backfill for old development outbox rows. The fresh database will contain only the new schema.

## Canonical local uploader behavior

The uploader must treat a transaction group as the upload unit.

Required behavior:

```text
select the oldest eligible pending TransactionGuid
load all rows for that group
verify count == ExpectedEventCount
verify sequences are contiguous 1..N
upload the complete group in one API request
mark the complete group sent only after complete API success
on failure, keep the complete group pending/retryable
never mark a subset sent
never upload child rows while parent sequence is missing
```

Concurrency requirements:

```text
one uploader claims a group once
multiple uploader workers cannot concurrently send the same group
replayed API request is safe through idempotency
```

Use the project's existing claiming/lease mechanism if one exists; otherwise add the smallest proven group-level claim mechanism.

## Canonical API request contract

Create or extend the standard API ingest request to carry one complete transaction group.

Required request concepts:

```text
TenantGuid
SourceClientId
TransactionGuid
ExpectedEventCount
Events[] ordered by Sequence
```

Each event must include:

```text
Sequence
EntityType
EntityGuid
Operation
Payload
```

The API must reject:

```text
empty group
count mismatch
sequence gap
sequence duplicate
sequence outside 1..N
mixed tenant
mixed source client
mixed TransactionGuid
unsupported entity or operation
invalid parent-child order for the Price Rule aggregate
```

Do not use SignalR as the data payload transport.

## Canonical API database contract

Audit actual API event and delivery table names. Extend the real canonical tables rather than inventing parallel tables unless no event/delivery system exists.

The durable API schema must preserve:

```text
TenantGuid
SourceClientId
TransactionGuid
Sequence
ExpectedEventCount
EntityType
EntityGuid
Operation
Payload
Event identity/idempotency key
DestinationClientId
Delivery status
Attempt/retry state
AcknowledgedAt or equivalent completion state
CreatedAt
```

Required uniqueness and ordering:

```text
unique source event by TenantGuid + SourceClientId + TransactionGuid + Sequence
unique destination delivery for one source event + destination client
index destination pending groups by TenantGuid + DestinationClientId + CreatedAt/TransactionGuid/Sequence
```

## Canonical API ingest transaction

One API database transaction must:

```text
validate the entire request group
check idempotency for the entire group
persist all source events
resolve all destination subscribers
persist all destination deliveries
preserve TransactionGuid, Sequence, and ExpectedEventCount
SaveChangesAsync
commit
```

Only after durable commit may the API publish SignalR notification.

Required replay behavior:

```text
same complete group replay
-> no duplicate source events
-> no duplicate destination deliveries
-> return success/idempotent result
```

Partial replay or conflicting payload under the same group/sequence must return an explicit conflict, not silently overwrite durable history.

If SignalR notification fails after commit:

```text
durable deliveries remain pending
API returns/records a safe post-commit notification result
POS clients can still recover by polling/pull
```

## Entity registration

Add clean canonical registration for:

```text
TblTurnPolicy
TblTurnAmountRule
```

Do not broaden unrelated entity permissions.

The source POS must be excluded from destination delivery according to the existing canonical source-client identity model.

## Canonical destination pull contract

The pull unit must be one complete transaction group.

Required response concepts:

```text
TransactionGuid
ExpectedEventCount
Events[] ordered by Sequence
```

Required API pull behavior:

```text
select the oldest complete pending group for the destination client
return all deliveries in that group
never split one group across pages
never return sequence 2 without sequence 1
verify the stored group is complete before returning it
```

A page size may limit the number of groups, but must never split a group.

If a stored group is incomplete or corrupt, do not deliver a prefix; mark/report an explicit group-integrity failure.

## Canonical POS2 group apply

POS2 must apply one complete group inside one local PostgreSQL transaction.

Required order:

```text
validate group metadata and contiguous sequence
begin local transaction
apply TblTurnPolicy I/U first
apply TblTurnAmountRule I/U/D in sequence
SaveChangesAsync once when possible
commit
acknowledge complete group
```

If any event fails:

```text
rollback complete local group
acknowledge nothing
leave complete group retryable
```

Do not acknowledge parent only.
Do not insert orphan child rules.
Do not discard child events whose parent is missing.

## Receiver contracts

### TblTurnPolicy receiver

```text
validate current tenant
I/U idempotent upsert by TenantGuid + TurnPolicyGuid
preserve Draft status/version fields
normalize PostgreSQL local timestamp kinds
do not activate policy
no local TblLocalOutbox echo
```

### TblTurnAmountRule receiver

```text
validate current tenant
validate TurnAmountRuleGuid and TurnPolicyGuid
verify parent policy exists within the same transaction/current DB
I/U idempotent upsert by TenantGuid + TurnAmountRuleGuid
D idempotent delete/deactivate according to proven current schema contract
preserve false, zero, null, and empty-clear values
preserve decimal precision
normalize PostgreSQL local timestamps
no local TblLocalOutbox echo
```

## Group acknowledgement contract

Add or extend the standard acknowledgement endpoint so acknowledgment is by complete destination transaction group.

Required ACK identity:

```text
TenantGuid
DestinationClientId
TransactionGuid
ExpectedEventCount
```

The API must acknowledge the complete group only when:

```text
all deliveries for the destination group exist
all sequences are present
client confirms complete local commit
```

ACK replay must be idempotent.

Do not retain an individual-event ACK path for these new clean transaction-group deliveries if it permits prefix completion. Because the databases will be recreated, compatibility with old development delivery rows is not required.

## Payload contracts

Use typed DTOs unless direct entity payload is already proven safe and navigation-free.

### TblTurnPolicy payload

Include all required actual fields:

```text
TurnPolicyGuid
TenantGuid
status
version/revision fields that actually exist
CreatedAt/CreatedBy
UpdatedAt/UpdatedBy
all other required non-null schema fields
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

Prove correct serialization and receiver clearing for:

```text
IsActive=false
SortOrder=0 when valid
MaxAmount=null
TurnCredit=null
Notes=null
Notes=""
```

## True change detection

Compare the proposed UI rule set against reloaded DB state and produce exactly:

```text
Inserted
TrulyUpdated
Deleted
Unchanged
```

Do not emit update events for unchanged rows.

For delete, capture payload/key before entity removal.

## Fresh-database migrations

Create clean forward migrations for both WPF and ApiServer.

Requirements:

```text
all migrations apply from an empty PostgreSQL database
no manual SQL required except operator-run database create/drop wrapper
no backfill of old development events
no conditional compatibility branch for old transaction rows
pending migrations = 0 after apply
```

If obsolete development migrations conflict with clean creation, repair the migration chain in the smallest controlled way and document it. Do not preserve an invalid chain merely to protect disposable data.

Do not delete historical migration source blindly. Produce an explicit migration-chain decision and evidence.

## Fresh three-database E2E proof

Create three separately named, versioned disposable PostgreSQL databases:

```text
POS1 fresh local DB
API fresh DB
POS2 fresh local DB
```

Do not use the operator's current canonical dev DBs for this proof.

Apply migrations from zero to all three databases.

Seed only the minimal canonical identity and setup required for the flow:

```text
one tenant
POS1 source identity
POS2 destination identity
subscription/routing records
no business transaction history
```

Required E2E:

### First Price Rule Save

```text
POS1 creates Draft TblTurnPolicy
POS1 creates one Price Rule
POS1 creates two ordered local outbox rows
sequence 1 policy I
sequence 2 rule I
same TransactionGuid
local three-table commit succeeds
```

### Upload/API

```text
POS1 uploader sends one complete group request
API persists complete source group and POS2 deliveries in one transaction
API replay creates no duplicates
SignalR occurs only after commit
```

### POS2 pull/apply

```text
POS2 pulls one complete group
POS2 applies policy then rule in one local transaction
POS2 creates no local outbox echo
POS2 ACKs complete group
replay leaves one final policy and one final rule
```

### Update

```text
POS1 true rule update
policy U sequence 1
rule U sequence 2
POS2 receives final updated value
```

### Delete

```text
POS1 rule delete
policy U sequence 1
rule D sequence 2
POS2 reaches final deleted/inactive state
```

### No-op

```text
POS1 Save without changes
no policy write
no rule write
no outbox group
```

### Failures

Prove:

```text
missing sequence prevents upload
API sequence gap rejected
API partial group not committed
API replay idempotent
SignalR failure after commit does not lose delivery
POS2 apply failure rolls back whole group
POS2 parent apply success + child failure does not ACK parent
POS2 retry applies whole group idempotently
wrong tenant rejected
source POS receives no echo delivery
```

## Diagnostics

Extend safe diagnostics to include:

```text
PolicyRowsChanged
RuleInsertedRows
RuleUpdatedRows
RuleDeletedRows
RuleUnchangedRows
LocalGroupExpectedEventCount
LocalGroupActualEventCount
FirstSequence
LastSequence
LocalTransactionCommitted
UploadGroupAccepted
ApiEventsPersisted
ApiDeliveriesPersisted
SignalRPublishedAfterCommit
DestinationGroupPulled
DestinationGroupApplied
DestinationGroupAcknowledged
ReceiverOutboxRowsCreated
ResultCode
```

Do not expose GUID values, payload values, credentials, or connection strings.

## Operator-run reset package

After all fresh E2E tests pass, create a versioned operator-run reset package. Do not execute it.

It must:

```text
stop relevant local apps or require operator confirmation
optionally create a final backup anchor
verify the exact approved database names
refuse protected/production database names
drop only approved dev databases
recreate clean UTF8 PostgreSQL databases
apply current migrations from zero
run minimal seed/bootstrap
print row-count and migration proof
```

Include a dry-run mode.

Do not embed credentials. Use interactive password or existing protected configuration.

## Documentation and evidence

Read before edits:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report092.md
report/report094.md
prompt/prompt094.md
```

Read the complete prompt094 private artifact.

Create a new versioned local artifact, for example:

```text
CleanTransactionGroupSyncV001
```

Required files:

```text
PRIVATE_HANDOFF.md
ARCHITECTURE.md
WPF_SCHEMA.md
API_SCHEMA.md
MIGRATIONS.md
LOCAL_SENDER.md
UPLOADER.md
API_INGEST.md
API_PULL_ACK.md
POS_GROUP_APPLY.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
FRESH_DB_E2E.md
TEST_OUTPUT.txt
RESET_PACKAGE.md
SHA256SUMS.txt
```

## Mandatory detailed handoff

Return complete actual C# and migration code, repository-relative paths, and line ranges for:

```text
Price Rule Save_Click
true change detection
Draft policy create/update in supplied DbContext
Price Rule entity changes
local transaction-group outbox creation
local pending-group selector/claim
complete-group upload request DTO and uploader
API group ingest endpoint/service
API event/delivery transaction
SignalR-after-commit call site
API group pull endpoint/service
API group ACK endpoint/service
WPF complete-group pull client
WPF complete-group local apply transaction
TblTurnPolicy receiver
TblTurnAmountRule receiver
no-echo behavior
all new migrations and indexes
fresh DB E2E harness
operator reset scripts
```

Include complete BEFORE and AFTER method bodies and unified diffs. Do not return only a summary or test counts.

## Safety boundaries

```text
PostgreSQL/Npgsql only
no Docker requirement
no automatic mutation/drop of current operator DBs
no checkout/payment test
no automatic production deployment
no old dev data backfill requirement
no OBM source commit/push
no credentials or private payloads in public report
```

## Public report

Create and push only:

```text
report/report095.md
```

It may contain only:

```text
verdict
clean transaction-group schema implemented yes/no
local three-table atomic save proven yes/no
whole-group uploader proven yes/no
API whole-group ingest/delivery proven yes/no
SignalR-after-commit proven yes/no
POS whole-group apply/ACK proven yes/no
idempotency/no-echo proven yes/no
fresh POS1/API/POS2 databases proven yes/no
reset package created yes/no
private detailed handoff created yes/no
build/test counts
current operator DB automatically mutated no
checkout tested no
one aggregate evidence SHA-256
```

## Valid verdicts

```text
OBM_CLEAN_TRANSACTION_GROUP_SYNC_READY_FOR_DEV_DB_RESET
```

```text
BLOCKED_CLEAN_TRANSACTION_GROUP_SCHEMA
```

```text
BLOCKED_CLEAN_API_GROUP_INGEST
```

```text
BLOCKED_CLEAN_POS_GROUP_APPLY
```

```text
BLOCKED_CLEAN_FRESH_DB_E2E
```

```text
BLOCKED_CLEAN_BUILD_OR_TEST
```
