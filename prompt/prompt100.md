# Prompt 100 — Implement whole-group WPF uploader and atomic API transaction-group ingest

## Approved gates

The following gates have passed:

```text
Prompt097:
OBM_WPF_POSTGRESQL_BASELINE_AND_OUTBOX_SCHEMA_READY_FOR_API_SCHEMA

Prompt098:
OBM_API_POSTGRESQL_TRANSACTION_GROUP_SCHEMA_READY_FOR_LOCAL_SENDER

Prompt099:
OBM_PRICE_RULE_LOCAL_TRANSACTION_GROUP_READY_FOR_UPLOADER_API_INGEST
```

Therefore the accepted starting state is:

```text
WPF TblLocalOutbox transaction-group schema: physically proven
API TblEventLog/TblEventDelivery/TblEventDeliveryGroupAck schema: physically proven
Price Rule local three-table transaction: physically proven
Policy-first local outbox ordering: proven
True I/U/D/no-op detection: proven
Current operator databases: untouched
Network/API path for the grouped events: not implemented yet
```

This task implements the next two boundaries only:

```text
1. WPF whole-transaction-group uploader
2. API whole-transaction-group durable ingest and delivery creation
```

## Strict scope

Implement and prove:

```text
whole-group claim/lease on TblLocalOutbox
complete-group validation before HTTP
one grouped API request
all-or-none local mark-sent/retry behavior
API request/auth/entity validation
whole-group idempotency and conflict detection
API source-event persistence
API destination-delivery persistence
one API database transaction and one commit for the complete group
source-client exclusion
SignalR notification only after durable API commit
safe behavior when SignalR fails after commit
crash/replay recovery between API commit and local mark-sent
focused disposable PostgreSQL + HTTP/API tests
```

Do not implement or modify in this task:

```text
API transaction-group pull endpoint
WPF transaction-group inbound apply
TblTurnPolicy receiver
TblTurnAmountRule receiver
API transaction-group ACK endpoint/runtime
POS2 local apply
Price Rule local Save semantics already accepted in prompt099
WPF/API schema already accepted in prompts097/098 except a narrowly proven migration defect
checkout/payment
BookingConsole runtime
installation Phase 1
operator's current WPF/API databases
cloud deployment
```

No destination POS apply is required for PASS. It is sufficient to prove durable API destination deliveries are created correctly and SignalR notification occurs after commit.

## Clean-development-data policy

The operator has declared current development data disposable.

Do not add compatibility behavior for:

```text
old incomplete TblLocalOutbox groups
old flat sync-event-batch rows
old event-level partial-success responses
old prefix ACK behavior
mixed old/new transaction-group payloads
```

However, do not automatically drop, recreate, migrate, reseed, or mutate the operator's current databases in this task.

Use newly named disposable WPF and API PostgreSQL databases built from the accepted prompt097 and prompt098 migration chains.

## Required evidence to read before editing

Read completely:

```text
<WPF_ROOT>/AGENTS.md
<API_ROOT>/AGENTS.md when present
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
prompt/prompt095.md
prompt/prompt097.md
prompt/prompt098.md
prompt/prompt099.md
report/report095.md
report/report097.md
report/report098.md
report/report099.md
```

Read all relevant private evidence from:

```text
E:\Project2026\RecoveryReports\CleanTransactionGroupSyncV001
E:\Project2026\RecoveryReports\WpfPostgreSqlMigrationBaselineV001
E:\Project2026\RecoveryReports\ApiPostgreSqlTransactionGroupSchemaV001
E:\Project2026\RecoveryReports\PriceRuleLocalTransactionGroupV001 when present
```

At minimum, inspect complete actual code for:

```text
TblLocalOutbox final model/mapping
Price Rule grouped outbox rows created by prompt099
MyApiProviderService.FlushOutboxAsync and all helpers
OutboxPublisherWorker
current SyncEventBatchRequest/SyncEventRequest
EntitiesController sync-event-batch endpoint
EntitiesService.ProcessEventBatchAsync
PayloadHelpers.LogEventAsync or equivalent event/delivery writer
TblEventLog final model/mapping
TblEventDelivery final model/mapping
TblSubscription and destination routing
TblTenantPosDevice/source-client identity
SignalR hub/notification publishing call chain
API transaction helper behavior
```

Record before editing:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
TASK_SCOPE=GROUP_UPLOADER_AND_API_INGEST_ONLY
WPF_GROUP_UNIT=TenantGuid+SourceClientId+TransactionGuid
API_DURABILITY_UNIT=COMPLETE_TRANSACTION_GROUP
SIGNALR_ROLE=POST_COMMIT_NOTIFICATION_ONLY
PULL_APPLY_ACK=DEFERRED
CURRENT_OPERATOR_DB_MUTATION=FORBIDDEN
```

## Proven starting defects

The prior WPF uploader behavior was event/batch oriented:

```text
claims an arbitrary row batch
then groups by TransactionGuid in memory
uploads events through sync-event-batch
marks individual rows Sent or Failed from per-event responses
can mark one event sent while another event in the same transaction group fails
```

The prior API ingest behavior was not a proven whole-group contract:

```text
flat event batch DTO
processing ordered by OccurredAt rather than canonical group SequenceNumber
multiple helper SaveChanges calls
SignalR may be invoked inside event/delivery helper before the complete request group commit is proven
no complete-group idempotency/conflict boundary
```

Do not preserve those defects for the new canonical Price Rule group path.

## Phase 1 — Audit canonical status, claim, and routing contracts

Before editing, return exact proof for:

```text
TblLocalOutbox Sent/status numeric meanings
Processing/lease behavior
AttemptCount and retry policy
current worker concurrency model
current HTTP authentication/session initialization
canonical SourceClientId format and source exclusion behavior
how destination POS clients are discovered for one tenant/entity subscription
whether TblSubscription identifies entity subscriptions per subscriber/client
whether one subscriber can map to multiple destination SourceClientId values
```

Do not guess status values.

Do not broaden routing to unrelated clients or entities.

For this task, canonical supported entity types are exactly:

```text
TblTurnPolicy
TblTurnAmountRule
```

Add the smallest registration/allowlist/subscription support required for these two types.

If the existing routing model cannot identify concrete destination POS client IDs safely, stop with:

```text
BLOCKED_SYNC_GROUP_DESTINATION_ROUTING_CONTRACT
```

and provide complete code/schema evidence.

## Phase 2 — Canonical grouped request/response DTOs

Create shared or equivalent explicit DTOs compatible between WPF and API.

### Request

Required shape:

```csharp
public sealed class SyncTransactionGroupRequest
{
    public Guid TenantGuid { get; set; }
    public string SourceClientId { get; set; } = string.Empty;
    public Guid TransactionGuid { get; set; }
    public int ExpectedEventCount { get; set; }
    public List<SyncTransactionGroupEventRequest> Events { get; set; } = new();
}

public sealed class SyncTransactionGroupEventRequest
{
    public long OutboxId { get; set; }
    public Guid EventGuid { get; set; }
    public int SequenceNumber { get; set; }
    public string EntityType { get; set; } = string.Empty;
    public Guid EntityGuid { get; set; }
    public string Operation { get; set; } = string.Empty;
    public string PayloadJson { get; set; } = string.Empty;
    public DateTime OccurredAt { get; set; }
}
```

Use actual project naming conventions when different, but preserve these concepts.

Do not send TenantGuid, SourceClientId, TransactionGuid, or ExpectedEventCount redundantly per child event unless a shared existing DTO contract requires it.

### Response

Required shape:

```csharp
public sealed class SyncTransactionGroupResponse
{
    public bool Accepted { get; set; }
    public bool IdempotentReplay { get; set; }
    public string ResultCode { get; set; } = string.Empty;
    public Guid TransactionGuid { get; set; }
    public int ExpectedEventCount { get; set; }
    public bool DurableCommitCompleted { get; set; }
    public bool SignalRNotificationSucceeded { get; set; }
    public List<SyncTransactionGroupEventResponse> Events { get; set; } = new();
}

public sealed class SyncTransactionGroupEventResponse
{
    public long OutboxId { get; set; }
    public Guid EventGuid { get; set; }
    public int SequenceNumber { get; set; }
    public long EventSequence { get; set; }
}
```

The WPF uploader must require:

```text
Accepted = true
DurableCommitCompleted = true
TransactionGuid matches request
ExpectedEventCount matches request
response event count matches request
EventGuid/SequenceNumber mappings exactly match the request
all EventSequence values > 0
```

A missing, partial, duplicate, or mismatched response must not mark any local row sent.

## Phase 3 — Whole-group local validation

Before claiming or uploading, validate the complete group:

```text
row count = ExpectedEventCount
ExpectedEventCount >= 1
all rows have the same TenantGuid
all rows have the same SourceClientId
all rows have the same TransactionGuid
all rows have the same ExpectedEventCount
sequences are exactly 1..N
EventGuid values are non-empty and unique
EntityGuid values are non-empty
operations are I/U/D only
entity types are supported
sequence 1 is TblTurnPolicy I or U
sequences 2..N are TblTurnAmountRule I/U/D
all rows are eligible pending/retry rows
```

Reject incomplete/corrupt groups before HTTP with a narrow safe result code.

Do not upload a prefix.

Do not upload child sequence 2 when sequence 1 is missing.

## Phase 4 — Atomic group claim/lease

The uploader selection unit is:

```text
TenantGuid + SourceClientId + TransactionGuid
```

Required behavior:

```text
select the oldest eligible complete group
claim every row in that group atomically
set one common ClaimedBy value
set one common ClaimExpiresAt value
set Processing status according to the proven enum
commit the claim transaction
load/retain the full ordered claimed group
```

Concurrency requirements:

```text
two uploader workers cannot successfully claim the same group
one worker cannot claim only a subset of the group
expired leases are recoverable
non-expired leases are respected
claim failure leaves the group unchanged or safely retryable
```

Use a PostgreSQL-safe method, selecting one proven approach:

```text
SELECT ... FOR UPDATE SKIP LOCKED inside one transaction
atomic conditional UPDATE with returned rows
serializable transaction with retry
another already proven project-standard PostgreSQL claim mechanism
```

Do not rely only on an in-process lock because multiple POS processes/workers may exist.

The claim transaction is separate from the HTTP call. Do not hold a PostgreSQL transaction open during network I/O.

## Phase 5 — One grouped HTTP request

Upload exactly one complete claimed transaction group per request to a canonical endpoint such as:

```text
POST /api/entities/sync-transaction-group
```

Use the actual controller route conventions.

Requirements:

```text
one request contains exactly one group
Events ordered by SequenceNumber
no separate request per event
normal WPF API authentication/session headers preserved
cancellation supported
safe timeout behavior
no credential/payload logging
```

If the HTTP call fails, times out, returns non-success, or returns an invalid response:

```text
mark the complete local group retryable
mark no row sent
increment retry metadata consistently for all rows
clear/release group claim
set common NextAttemptAt according to canonical backoff
store only a sanitized safe error
```

Do not permanently fail only one event from the group.

## Phase 6 — All-or-none local completion

After a valid durable API success response, update the complete claimed group in one local PostgreSQL transaction:

```text
verify rows still belong to the same claimed group
verify no rows disappeared or changed ownership
map each response EventGuid/SequenceNumber to exactly one outbox row
set all rows Sent
set SentAt
set each ServerEventSequence
clear Processor/ClaimedBy/ClaimExpiresAt
clear safe error/retry scheduling fields according to canonical policy
commit once
```

If local mark-sent fails after API durable commit:

```text
leave/recover the group as retryable or lease-expirable
next upload replays the identical complete group
API idempotency returns the existing durable result
then local mark-sent completes all rows
```

This crash window must be explicitly tested.

Do not add a local state that assumes SignalR notification success is required for durability.

## Phase 7 — API endpoint authentication and complete-group validation

Add the canonical endpoint/controller action.

Validate authenticated claims/envelope against the request:

```text
TenantGuid matches authorized tenant
SourceClientId matches authorized source client
source client is active/current according to the established POS identity contract
TransactionGuid is non-empty
ExpectedEventCount >= 1
Events.Count = ExpectedEventCount
sequences exactly 1..N
EventGuid values non-empty and unique
EntityGuid values non-empty
operations I/U/D only
entity types supported
sequence 1 is TblTurnPolicy I/U
sequences 2..N are TblTurnAmountRule I/U/D
no mixed aggregate metadata
payload JSON is present and syntactically valid
```

Validate typed payload envelope consistency without applying business rows to the API domain database in this task:

```text
policy payload TenantGuid/key matches request/event
rule payload TenantGuid/key matches request/event
rule payload TurnPolicyGuid matches sequence-1 policy key
```

Reject malformed or inconsistent groups before durable writes.

Use narrow result codes and HTTP semantics:

```text
400 for validation failure
401/403 for auth/identity mismatch
409 for idempotency conflict/partial conflicting group
200 for accepted new group or exact idempotent replay
```

Do not expose sensitive payload details in responses/errors.

## Phase 8 — Whole-group idempotency and conflict contract

The API idempotency identity is:

```text
TenantGuid + SourceClientId + TransactionGuid
```

### Exact complete replay

When the complete group already exists, prove every persisted event matches the request by at least:

```text
ExpectedEventCount
SequenceNumber
EventGuid
EntityType
EntityGuid
Operation
canonical payload hash or exact canonical payload
OccurredAt policy when part of the immutable contract
```

Then return:

```text
Accepted = true
IdempotentReplay = true
DurableCommitCompleted = true
existing EventSequence mappings
no duplicate TblEventLog rows
no duplicate TblEventDelivery rows
```

### Conflict

Return 409 and do not mutate durable history when:

```text
only a prefix of the TransactionGuid already exists
stored group count is incomplete
same group/sequence has a different EventGuid
same EventGuid has conflicting content
same TransactionGuid has a different ExpectedEventCount
same immutable event metadata has a different payload
stored destination deliveries are incomplete/corrupt relative to the accepted source group
```

Do not silently repair or overwrite conflicting durable history in this task.

## Phase 9 — Destination routing and source exclusion

Resolve destinations using the actual tenant/subscription/POS identity model.

Required behavior:

```text
only active destination clients for the same tenant
only clients subscribed to the event entity type according to the canonical routing contract
exclude request.SourceClientId
no duplicate destination client for one event
```

For every source event and every resolved destination client, create exactly one `TblEventDelivery` with:

```text
TenantGuid
SourceClientId
DestinationClientId
TransactionGuid
SequenceNumber
ExpectedEventCount
EventSequence FK
EntityType
EntityGuid
Operation
PayloadJson
Pending status
CreatedAt/OccurredAt
```

The accepted API schema unique indexes must enforce no duplicate destination delivery.

If there are zero destination clients, the source group may still be durably accepted when that matches canonical behavior. Document the decision and prove it. Do not invent a fake destination.

## Phase 10 — One API database transaction

The durable API boundary is one complete request group.

Required order:

```text
validate request before mutation
begin one ExternalDbContext PostgreSQL transaction
check exact replay/conflict
load destination routing
stage all TblEventLog rows
stage all TblEventDelivery rows
SaveChangesAsync once when technically possible
commit once
only after commit publish SignalR notification
```

One additional `SaveChangesAsync` inside the same transaction is acceptable only when generated `EventSequence` identities are technically required before delivery FK staging and EF relationship fix-up cannot safely avoid it.

Regardless of SaveChanges count:

```text
there is exactly one database transaction
there is exactly one commit
no source event commits without all destination deliveries
no SignalR call occurs before commit
```

On any pre-commit failure:

```text
rollback complete group
zero new source events
zero new destination deliveries
```

Do not call the prior helper if it commits or publishes SignalR per event.

Refactor or create a group-specific writer that receives the supplied DbContext and never commits/publishes independently.

## Phase 11 — SignalR after durable commit

SignalR is notification only.

Preferred notification granularity:

```text
one notification per destination client per committed TransactionGuid
```

Use the actual hub/client registration model and document the final payload semantics.

The notification should contain only safe routing/hint metadata needed to trigger a pull, such as:

```text
TransactionGuid presence indicator or safe group reference
ExpectedEventCount
newest EventSequence or availability hint
```

Do not send the complete business payload through SignalR.

Prove ordering:

```text
API transaction CommitAsync completes successfully
then SignalR publish method is invoked
```

If SignalR fails after commit:

```text
durable source events remain committed
durable destination deliveries remain pending
API response remains Accepted=true and DurableCommitCompleted=true
SignalRNotificationSucceeded=false or equivalent warning
WPF uploader still marks the group sent because durable API commit succeeded
POS destinations can later recover by polling/pull
```

Do not roll back or return a false ingest failure after durable commit merely because notification failed.

## Phase 12 — Diagnostics and result codes

WPF uploader diagnostics must include safe counts/flags:

```text
GroupFound
GroupClaimed
ExpectedEventCount
ActualRowCount
FirstSequence
LastSequence
HttpRequestSent
HttpStatusClass
ApiAccepted
ApiIdempotentReplay
ApiDurableCommitCompleted
ApiSignalRNotificationSucceeded
RowsMarkedSent
RowsMarkedRetryable
AttemptCount
ClaimReleased
ResultCode
```

API diagnostics/tests must include safe counts/flags:

```text
RequestEventCount
ReplayDetected
SourceEventsInserted
DestinationClientCount
DeliveriesInserted
SaveChangesCallCount
TransactionStarted
Committed
SignalRPublishAttempted
SignalRPublishSucceeded
ResultCode
```

Do not log:

```text
connection strings
passwords/tokens
raw tenant/device GUIDs in public artifacts
business payload contents
private policy/rule values
```

Required narrow result codes should include or map to:

```text
SYNC_GROUP_ACCEPTED
SYNC_GROUP_REPLAY_ACCEPTED
SYNC_GROUP_VALIDATION_FAILED
SYNC_GROUP_AUTH_MISMATCH
SYNC_GROUP_SEQUENCE_INVALID
SYNC_GROUP_PARENT_NOT_FIRST
SYNC_GROUP_PAYLOAD_MISMATCH
SYNC_GROUP_CONFLICT
SYNC_GROUP_API_COMMIT_FAILED
SYNC_GROUP_SIGNALR_FAILED_AFTER_COMMIT
OUTBOX_GROUP_INCOMPLETE
OUTBOX_GROUP_CLAIM_FAILED
OUTBOX_GROUP_HTTP_FAILED
OUTBOX_GROUP_RESPONSE_INVALID
OUTBOX_GROUP_MARK_SENT_FAILED
OUTBOX_GROUP_SENT
```

## Phase 13 — Disposable end-to-end sender-to-API proof

Create separately named disposable PostgreSQL databases:

```text
one WPF POS1 DB from prompt097 migration chain
one API DB from prompt098 migration chain
```

Seed only the minimum safe setup required:

```text
one tenant identity
one source POS client
one destination POS client
entity subscriptions/routing for TblTurnPolicy and TblTurnAmountRule
minimal Price Rule local Save prerequisites
```

Do not seed unrelated business history.

Required E2E case:

```text
1. Execute first Price Rule Save on POS1.
2. Prove local group has sequence 1 policy I and sequence 2 rule I.
3. Run whole-group uploader.
4. Prove one grouped HTTP request.
5. API accepts and commits the complete source group.
6. API creates complete destination deliveries for POS2.
7. API commits before SignalR invocation.
8. WPF marks both outbox rows sent together.
```

Expected durable counts for one destination:

```text
POS1 TblLocalOutbox rows in group = 2, all Sent after success
API TblEventLog rows in group = 2
API TblEventDelivery rows for destination = 2
sequence values = 1 and 2
ExpectedEventCount = 2 everywhere
one TransactionGuid throughout
source client receives no destination delivery
```

Do not pull/apply on POS2 in this task.

## Phase 14 — Required failure/replay test matrix

Prove at least:

```text
A. Incomplete local group: no HTTP, no row sent.
B. Local sequence gap: no HTTP, no row sent.
C. Two uploader workers: exactly one claims/uploads the group.
D. HTTP/network failure: all rows retryable, zero rows sent.
E. Invalid/partial API response: all rows retryable, zero rows sent.
F. API validation failure: zero source events and zero deliveries.
G. API failure after staging but before commit: complete rollback.
H. Exact API replay: zero duplicate events/deliveries, accepted replay response.
I. Conflicting replay: 409, durable history unchanged.
J. API committed but local mark-sent simulated failure: replay succeeds later and all rows become sent.
K. SignalR failure after API commit: durable rows remain, API accepted, local group marked sent.
L. Source exclusion: zero deliveries to source client.
M. Multiple destination clients: complete group delivered exactly once per destination.
N. Unsupported entity or wrong parent ordering: rejected before durable write.
O. Mixed tenant/source metadata or payload key mismatch: rejected before durable write.
```

Use real PostgreSQL constraints and a real HTTP/API test host where practical.

Do not replace database proof with mocks alone.

## Phase 15 — Builds and focused tests

Run:

```text
WPF build
API build
focused WPF uploader tests
focused API group ingest tests
real PostgreSQL disposable E2E tests
```

No soft skips when prerequisites are available.

Build success alone is not PASS.

Report existing unrelated warnings separately; do not widen scope merely to eliminate historical warnings.

## Required private evidence artifact

Create a new versioned local artifact. Never overwrite earlier evidence.

Suggested folder:

```text
E:\Project2026\RecoveryReports\WholeGroupUploaderApiIngestV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
CURRENT_FLOW_AUDIT.md
DTO_CONTRACT.md
WPF_GROUP_VALIDATION.md
WPF_GROUP_CLAIM.md
WPF_HTTP_UPLOAD.md
WPF_COMPLETION_RETRY.md
API_AUTH_VALIDATION.md
API_IDEMPOTENCY.md
API_ROUTING.md
API_TRANSACTION.md
SIGNALR_AFTER_COMMIT.md
E2E_PROOF.md
FAILURE_MATRIX.md
TEST_OUTPUT.txt
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

## Mandatory private code handoff

Return complete actual code, not excerpts, for:

```text
final grouped request/response DTOs
FlushOutboxAsync public entry and grouped implementation
complete-group selection/validation
claim/lease transaction
HTTP request construction
response validation
all-or-none mark-sent
all-or-none retry/release
API controller endpoint
API auth validator for group request
API request validation
idempotency replay/conflict methods
routing/destination resolution
source event/delivery staging
API transaction begin/save/commit/rollback
post-commit SignalR method and call site
focused tests and disposable E2E harness
```

Include:

```text
repository-relative path
line range
complete BEFORE method/body
complete AFTER method/body
actual unified diff
actual SQL/query evidence for claim, durable writes, and count verification
```

Do not expose credentials, full connection strings, tokens, passfile contents, raw identities, or business payload values.

## Public report

Create and push only:

```text
report/report100.md
```

The public report must be minimal and redacted, containing:

```text
Verdict
Whole-group WPF selection/validation yes/no
Atomic group claim/lease yes/no
One grouped HTTP request yes/no
All-or-none local completion/retry yes/no
API whole-group validation/auth yes/no
API complete-group idempotency/conflict yes/no
API source events + destination deliveries one transaction yes/no
Source exclusion yes/no
SignalR after commit yes/no
SignalR failure-after-commit recovery yes/no
Crash/replay recovery yes/no
Disposable POS1-to-API E2E yes/no
Failure matrix pass/fail totals
Focused test totals
WPF build errors/warnings
API build errors/warnings
Current operator DBs mutated yes/no
POS2 pull/apply performed yes/no
Private evidence artifact yes/no
Aggregate SHA-256
```

Do not publish source code, SQL, payloads, secrets, private paths containing secret material, raw identifiers, or customer/business data.

## Source and coordination repository rules

OBM source changes and detailed evidence remain local/private.

Do not commit or push OBM source to the coordination repository.

Only commit and push:

```text
report/report100.md
```

Preserve unrelated dirty local changes.

Do not reset, clean, checkout, overwrite, or discard unrelated work.

## Scope exclusions

Must remain behaviorally unchanged:

```text
checkout/payment
invoice settlement
terminal/Dejavoo
gift cards
customer/booking
BookingConsole runtime
WPF installation Phase 1
migration baselines from prompts097/098
Price Rule local transaction semantics from prompt099
Employee Weight
Weird Tip
TurnEngine calculations
current operator databases
```

## Final verdicts

PASS only when all durable uploader/API gates and the disposable POS1-to-API E2E pass:

```text
OBM_TRANSACTION_GROUP_UPLOADER_AND_API_INGEST_READY_FOR_POS_GROUP_PULL_APPLY_ACK
```

Use a narrow blocker otherwise, for example:

```text
BLOCKED_OUTBOX_GROUP_CLAIM_CONTRACT
BLOCKED_SYNC_GROUP_AUTH_CONTRACT
BLOCKED_SYNC_GROUP_DESTINATION_ROUTING_CONTRACT
BLOCKED_SYNC_GROUP_IDEMPOTENCY_CONTRACT
BLOCKED_SYNC_GROUP_API_TRANSACTION
BLOCKED_SYNC_GROUP_SIGNALR_COMMIT_ORDER
BLOCKED_SYNC_GROUP_E2E
```

Do not implement POS group pull/apply/ACK inside this task.
