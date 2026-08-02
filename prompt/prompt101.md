# Prompt 101 — Prove whole-group POS1 uploader to API ingest E2E and complete failure matrix

## Starting verdict

Prompt100 returned:

```text
BLOCKED_SYNC_GROUP_E2E
```

The public report states the implementation builds and the following code boundaries are present:

```text
whole-group WPF selection/validation
atomic group claim/lease
one grouped HTTP request
all-or-none local completion/retry
API whole-group validation/auth
API complete-group idempotency/conflict handling
API source events and destination deliveries in one transaction
source-client exclusion
SignalR after commit
SignalR failure-after-commit handling
crash/replay recovery implementation
```

However, physical acceptance was correctly withheld because:

```text
disposable POS1-to-API E2E was not executed
failure matrix: 0 passed, 15 not executed
focused tests were not executed
```

This task is therefore a verification-and-narrow-correction task for the existing prompt100 implementation.

## Strict scope

Prove the complete runtime boundary:

```text
POS1 disposable PostgreSQL
-> complete grouped TblLocalOutbox rows
-> WPF whole-group claim
-> real loopback HTTP request
-> authenticated ApiServer endpoint
-> API whole-group validation
-> atomic TblEventLog + TblEventDelivery persistence
-> API commit
-> post-commit SignalR notification attempt
-> valid response
-> all-or-none POS1 local mark-sent
```

Do not implement in this task:

```text
API transaction-group pull endpoint
POS2 pull
POS2 local group apply
TblTurnPolicy inbound receiver
TblTurnAmountRule inbound receiver
API group ACK endpoint
cloud deployment
current operator DB reset
checkout/payment changes
BookingConsole runtime changes
```

If a test reveals a narrow defect inside the already approved prompt100 uploader/API-ingest boundary, fix the smallest defect and rerun the full matrix.

Do not broaden the design or create a second sync transport.

## Database and environment rules

Use separately named, versioned disposable PostgreSQL databases created from the accepted migration chains:

```text
one disposable WPF/POS1 DB from prompt097 migrations
one disposable API DB from prompt098 migrations
```

Do not use or mutate:

```text
obm_pos_dev_v0_pg
obm_pos_v1_local_pos1_pg
enailsalon_phasee1_pos1_pg
recovery_api_day16_pg
obm_api_dev_v0_pg
any current/protected/production DB
```

Use loopback-only API hosting.

Use protected credentials/environment/passfiles without exposing secrets.

Do not print:

```text
passwords
full connection strings
passfile contents
JWTs/tokens
raw tenant/POS/device GUIDs
private Price Rule payload values
```

## Required evidence to read before execution

Read completely:

```text
<WPF_ROOT>/AGENTS.md
<API_ROOT>/AGENTS.md when present
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
prompt/prompt097.md
prompt/prompt098.md
prompt/prompt099.md
prompt/prompt100.md
report/report097.md
report/report098.md
report/report099.md
report/report100.md
```

Read every relevant file in the prompt100 private evidence artifact, including actual code and test plans.

Also read the accepted migration and sender artifacts from prompts097–099.

Record before execution:

```text
DOCS_READ_BEFORE_TEST_GATE=PASS
TASK_SCOPE=POS1_TO_API_GROUP_E2E_ONLY
POS2_PULL_APPLY_ACK=DEFERRED
CURRENT_OPERATOR_DB_MUTATION=FORBIDDEN
LOOPBACK_API_ONLY=TRUE
```

## Phase 1 — Audit actual prompt100 implementation before running tests

Return exact repository-relative paths and complete method bodies for the current implementation of:

```text
WPF grouped DTOs
API grouped DTOs
WPF group selection query
WPF group integrity validation
WPF claim/lease transaction
WPF grouped HTTP call
WPF response validation
WPF all-or-none mark-sent transaction
WPF all-or-none retry transaction
API grouped controller endpoint
API auth/envelope validation
API group validation
API idempotent replay detection
API conflict detection
API event/delivery persistence transaction
source-client exclusion
post-commit SignalR invocation
SignalR failure handling
```

Confirm that the runtime code being tested is the prompt100 implementation, not a proposed artifact or test-only duplicate.

## Phase 2 — Build disposable environment

Create the two disposable databases from zero:

```text
POS1 DB:
- apply accepted WPF migrations
- pending migrations = 0

API DB:
- apply accepted API migrations
- pending migrations = 0
```

Seed only the minimum identity/routing state required:

```text
one tenant
one POS1 source-client identity
one POS2 destination-client identity
subscriptions/registration for exactly:
  TblTurnPolicy
  TblTurnAmountRule
```

Do not seed unrelated business history.

Start the real ApiServer on loopback against the disposable API DB.

Prove:

```text
health/readiness succeeds
authenticated grouped endpoint is reachable
API is connected only to the disposable API DB
```

## Phase 3 — Create a real local grouped outbox transaction

Use the accepted prompt099 Price Rule local sender against the disposable POS1 DB.

Do not manually insert outbox rows when the production sender can produce them.

Required initial group:

```text
ExpectedEventCount = 2
Sequence 1 = TblTurnPolicy I
Sequence 2 = TblTurnAmountRule I
same TenantGuid
same SourceClientId
same TransactionGuid
unique non-empty EventGuid values
non-empty EntityGuid values
Sent = Pending
```

Capture only sanitized counts and contract markers.

## Phase 4 — Happy-path physical E2E

Execute the real WPF uploader once.

Prove in order:

### POS1 claim

```text
exactly one complete group selected
both rows claimed by one worker
same ClaimExpiresAt
no subset claim
```

### HTTP

```text
exactly one HTTP request
request contains exactly one transaction group
Events ordered 1,2
normal auth/session envelope present
```

### API durable transaction

Before commit, prove SignalR has not been called.

After ingest succeeds, prove:

```text
TblEventLog rows = 2
same TransactionGuid
ExpectedEventCount = 2
sequences = 1,2
policy event first
no duplicate EventGuid
```

For the destination POS2 client, prove:

```text
TblEventDelivery rows = 2
same TransactionGuid
ExpectedEventCount = 2
sequences = 1,2
DestinationClientId = POS2 identity
source POS1 delivery rows = 0
```

Prove event rows and all delivery rows were committed by one API transaction boundary.

### SignalR

Prove notification is attempted only after the durable API commit.

SignalR remains notification-only; do not transport event payload through SignalR.

### POS1 completion

Validate the API response and prove both local outbox rows transition together:

```text
Sent = Sent canonical value
SentAt populated
ServerEventSequence mapped correctly
claim fields cleared
no partial completion
```

Expected happy-path result:

```text
SYNC_GROUP_E2E_COMMITTED
```

## Phase 5 — Mandatory failure/recovery matrix

Execute every case below. No soft skip.

### Case 1 — Incomplete local group

Remove or withhold sequence 2 in a disposable/test transaction.

Expected:

```text
uploader rejects before HTTP
HTTP requests = 0
no row marked Sent
```

### Case 2 — Sequence gap/duplicate

Test a non-contiguous or duplicated sequence group.

Expected:

```text
reject before HTTP or DB constraint rejects construction
no partial upload
```

### Case 3 — Invalid parent-first ordering

Sequence 1 is not `TblTurnPolicy I/U`.

Expected:

```text
reject before HTTP
```

### Case 4 — Concurrent uploader workers

Run two real workers/tasks against the same pending group.

Expected:

```text
exactly one worker claims/uploads the group
API accepted request count = 1, except an explicitly forced replay case
no duplicate local completion
```

### Case 5 — Lease recovery

Create/force an expired claim.

Expected:

```text
complete group becomes claimable again
non-expired claim is respected
```

### Case 6 — HTTP unavailable/timeout

Stop or block the API before upload.

Expected:

```text
complete group becomes retryable
all rows share retry state/backoff
no row Sent
claim released or lease-recoverable
```

### Case 7 — API auth/identity mismatch

Use wrong tenant or source-client identity envelope.

Expected:

```text
401/403
no API event rows
no API delivery rows
whole local group retryable/not Sent
```

### Case 8 — API validation rejection

Send count mismatch, sequence gap, unsupported operation/entity, missing key, or payload-envelope mismatch through focused endpoint tests.

Expected:

```text
400
zero durable writes
```

### Case 9 — Exact idempotent replay

Replay the exact complete request after durable commit.

Expected:

```text
200 accepted/idempotent result
no duplicate TblEventLog
no duplicate TblEventDelivery
same event sequence mapping returned
```

### Case 10 — Conflicting replay

Reuse the same group identity with changed EventGuid/entity/operation/payload hash/count/sequence.

Expected:

```text
409 conflict
no durable history overwritten
```

### Case 11 — API persistence failure before commit

Inject a deterministic failure after staging part of the group but before commit.

Expected:

```text
zero committed source events for the group
zero committed deliveries for the group
SignalR calls = 0
local group retryable/not Sent
```

### Case 12 — SignalR failure after API commit

Inject notification failure after durable commit.

Expected:

```text
API event/delivery rows remain committed
response states DurableCommitCompleted=true
SignalRNotificationSucceeded=false or equivalent safe marker
POS1 marks complete local group Sent
```

SignalR failure must not cause duplicate durable data.

### Case 13 — Crash window: API committed, POS1 not marked Sent

Inject failure after API returns durable success but before local mark-sent commits.

Expected first attempt:

```text
API durable rows exist
local group remains Processing/retryable/lease-recoverable
```

Expected replay:

```text
API returns exact idempotent result
local complete group then marks Sent atomically
no API duplicates
```

### Case 14 — Invalid/partial API response

Return a test response missing one event mapping or containing mismatched group metadata.

Expected:

```text
no local row marked Sent
complete group retryable
```

### Case 15 — Source exclusion and destination routing

Use one source POS and at least one destination POS registration.

Expected:

```text
no delivery to source client
exactly one complete delivery group per valid destination client
no unrelated client/entity delivery
```

## Phase 6 — Focused automated tests

Run focused tests covering at least:

```text
local group validation
atomic claim concurrency
lease recovery
all-or-none retry
response integrity validation
API auth and request validation
exact replay
conflicting replay
API rollback before commit
SignalR-after-commit ordering
SignalR post-commit failure
crash/replay recovery
source exclusion/routing
```

The public report must list actual totals.

Build success alone is not acceptance.

## Phase 7 — Final database assertions

At the end of the successful happy-path/replay test, prove sanitized counts:

```text
POS1:
- one completed local outbox group
- all rows Sent
- no mixed statuses

API:
- source event count equals ExpectedEventCount
- destination delivery count equals ExpectedEventCount × valid destination count
- source delivery count = 0
- no duplicate source events
- no duplicate deliveries
```

Do not advance to POS2 apply or ACK.

## Narrow correction policy

If any case fails:

```text
identify the first exact failing boundary
capture complete safe exception/SQL state/method chain
make the smallest correction inside prompt100 scope
rerun the failed case
rerun the complete 15-case matrix
```

Do not claim PASS with unexecuted cases.

Do not defer a test merely because implementation builds.

## Required private evidence artifact

Create a new versioned local artifact, never overwrite prior evidence.

Suggested folder:

```text
E:\Project2026\RecoveryReports\TransactionGroupUploaderApiE2EV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ENVIRONMENT.md
ACTUAL_CODE.md
DISPOSABLE_DB_SETUP.md
API_RUNTIME.md
HAPPY_PATH_E2E.md
FAILURE_MATRIX.md
CONCURRENCY_LEASE_PROOF.md
AUTH_VALIDATION_PROOF.md
IDEMPOTENCY_CONFLICT_PROOF.md
API_ROLLBACK_PROOF.md
SIGNALR_ORDERING_PROOF.md
SIGNALR_FAILURE_PROOF.md
CRASH_REPLAY_PROOF.md
SOURCE_ROUTING_PROOF.md
TEST_OUTPUT.txt
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Include complete actual changed C# methods and actual test methods in the private handoff.

## Public report

Create and push only:

```text
report/report101.md
```

The redacted public report must include:

```text
Verdict
Disposable POS1 DB created/applied yes/no
Disposable API DB created/applied yes/no
Loopback authenticated API yes/no
Production local sender used yes/no
Whole-group HTTP E2E yes/no
API atomic event/delivery commit yes/no
Source exclusion yes/no
SignalR-after-commit proof yes/no
All-or-none POS1 completion yes/no
Exact replay proof yes/no
Conflicting replay proof yes/no
Crash/replay proof yes/no
Failure matrix passed/failed/not-executed totals
Focused test totals
WPF/API build errors/warnings
Current operator DBs mutated yes/no
POS2 pull/apply/ACK performed yes/no
Private artifact yes/no
Aggregate SHA-256
```

Do not expose credentials, source code, raw IDs, payloads, connection strings, or private database rows.

## Coordination/source rules

OBM source changes and detailed evidence remain local/private.

Do not commit or push OBM source to the coordination repository.

Commit/push only:

```text
report/report101.md
```

Preserve unrelated dirty local work.

Do not reset, clean, checkout, or overwrite unrelated files.

## Final verdicts

PASS only when the real E2E and all required failure cases are executed successfully:

```text
OBM_TRANSACTION_GROUP_UPLOADER_API_INGEST_E2E_READY_FOR_POS_GROUP_PULL_APPLY_ACK
```

Use a narrow blocker otherwise, for example:

```text
BLOCKED_SYNC_GROUP_DISPOSABLE_ENVIRONMENT
BLOCKED_SYNC_GROUP_AUTH_RUNTIME
BLOCKED_SYNC_GROUP_CLAIM_CONCURRENCY
BLOCKED_SYNC_GROUP_API_ATOMICITY
BLOCKED_SYNC_GROUP_IDEMPOTENCY
BLOCKED_SYNC_GROUP_SIGNALR_ORDERING
BLOCKED_SYNC_GROUP_CRASH_REPLAY
BLOCKED_SYNC_GROUP_FAILURE_MATRIX
```

Do not implement POS2 pull/apply/ACK inside this task.
