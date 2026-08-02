# Prompt 111 — Execute one physical canonical main-Development POS1 → API grouped-sync happy path

## Starting checkpoint

Report110 passed with:

```text
OBM_CANONICAL_SINGLE_SYNC_FLOW_CONSOLIDATED_READY_FOR_MAIN_DEV_E2E
```

Coordination references:

```text
report/report110.md
report110 commit: 38ec773e3f16bf812acb1e4f52689427ae96a01a
prompt110 private artifact:
E:\Project2026\RecoveryReports\CanonicalSyncFlowConsolidationV001
aggregate SHA-256:
a7d113ef381c07095b3ccd4145de734d4011e5eb51a78d2f6c7f6095ae868ccd
```

Accepted anchors also include:

```text
report107:
OBM_MAIN_WPF_DEV_RESET_MIGRATION_READY_FOR_API_RESET
artifact SHA-256:
47f68c634a5984611f3cb8b39ba3999f6005a558ad1e0d64bf998f7f4c2a0c58

report108:
OBM_MAIN_API_DEV_RESET_MIGRATION_READY_FOR_SYNC_FLOW_AUDIT
artifact SHA-256:
e9d8298486f31f40581cb4445fa0abac25030bd586303098c05e1a9225f0d0ea

report109:
OBM_LEGACY_FIREBASE_USER_SECRET_REMOVED_WPFJWT_CANONICAL_READY_FOR_SYNC_FLOW_AUDIT
artifact SHA-256:
b97b2eaed1738c497502c92b057c5133bf6b20345d302b3daea44541d0012dfa
```

Report110 proves at source/architecture-test level:

```text
one production WPF periodic outbox worker
one production WPF outbox-claim chain
one production grouped HTTP upload chain
one production API grouped ingest controller/action
one production API durable ingest service chain
one production EventLog/Delivery writer chain
one production post-commit SignalR path
parallel production paths remaining = 0
architecture guards = 5 passed, 0 failed, 0 skipped
focused tests = 8 passed, 0 failed, 0 skipped
```

Runtime E2E has not yet been executed. This task closes exactly one physical happy path before the failure/recovery matrix.

## Authoritative architecture lock

The only canonical sync flow is:

```text
WPF domain Save + TblLocalOutbox in one local PostgreSQL transaction
-> existing periodic WPF outbox worker
-> existing canonical FlushOutbox/API service chain
-> existing standard API grouped sync controller/service chain
-> existing TblEventLog/TblEventDelivery transaction path
-> one successful API database commit
-> existing post-commit SignalR notification
```

Do not create or restore:

```text
second uploader
second periodic bot
second API endpoint/controller family
second ingest service
second event/delivery writer
second ACK/delivery transport
second SignalR publisher
manual HTTP transport that bypasses the canonical worker/service chain
```

SignalR remains notification-only. Do not send business payloads through SignalR.

## Authentication lock

The accepted WPF installation/bootstrap authorization flow remains:

```text
Pairing Code -> redeem -> WpfJwt -> bootstrap/me
```

Do not restore Firebase email/password.

Do not assume that the installation-scoped `WpfJwt` is also the normal runtime sync credential. Audit the actual current production sync authentication/session path and use it exactly as implemented.

Requirements:

```text
no AllowAnonymous
no authorization-policy bypass
no hardcoded reusable token in source
no source/report token disclosure
no new JWT scheme
no new token exchange service
no Firebase fallback
```

If the current Development state cannot establish the existing production sync credential/session contract, stop with the narrow blocker:

```text
BLOCKED_MAIN_DEV_SYNC_AUTH
```

and provide the exact scheme/policy/call-chain boundary. Do not weaken auth.

## Strict scope

Execute only one canonical physical happy path:

```text
1. Verify accepted private artifacts and current source.
2. Resolve and physically verify the canonical WPF and API Development databases.
3. Prove both migration chains remain current with pending migrations = 0.
4. Resolve the actual production sync authentication and source-client identity path.
5. Prepare only the minimal Development-only identity/routing prerequisites required by the existing production contracts.
6. Start the actual ApiServer Development runtime on loopback against the canonical API Development DB.
7. Establish an existing SignalR destination registration/listener when required to prove the existing notification path.
8. Execute one real Price Rule domain Save through the production WPF Save boundary.
9. Allow or trigger one cycle of the existing production periodic outbox worker through its canonical entry chain.
10. Prove one grouped HTTP request reaches the one canonical API ingest action.
11. Prove one atomic API event/delivery commit and post-commit SignalR notification.
12. Prove all local outbox rows in the group are marked Sent together.
13. Run focused regression/architecture guards and build WPF/API after any narrow correction.
```

Do not execute in this task:

```text
15-case failure/recovery matrix
POS2 pull/apply
runtime group ACK
Category Weight implementation
Customer/Booking Weight implementation
checkout/payment changes
BookingConsole changes
WPF or API database reset
migration generation unless a narrow proven source defect requires it
cloud deployment
production deployment
```

Do not mutate production/customer/reference databases.

## Required evidence intake

Read completely:

```text
OBM_POS_NewChat_Handoff_V001_2026-08-02.md when locally available
prompt/prompt099.md
report/report099.md
prompt/prompt100.md
report/report100.md
prompt/prompt107.md
report/report107.md
prompt/prompt108.md
report/report108.md
prompt/prompt109.md
report/report109.md
prompt/prompt110.md
report/report110.md
```

Read and verify at minimum:

```text
E:\Project2026\RecoveryReports\PriceRuleLocalTransactionGroupV001 when present
aggregate SHA-256 from report099:
2c5ceae238a8a276b8903e28aea57f6db132f0316f16642b299cb9b7ce0cd94c

E:\Project2026\RecoveryReports\WholeGroupUploaderApiIngestV001
aggregate SHA-256 from report100:
9e4ef1e4df63373abb052c055e7a17c75efbca76a81a315b19a0d513fc9bcf42

E:\Project2026\RecoveryReports\MainWpfDevResetExecutionV002
aggregate SHA-256:
47f68c634a5984611f3cb8b39ba3999f6005a558ad1e0d64bf998f7f4c2a0c58

E:\Project2026\RecoveryReports\MainApiDevResetExecutionV001
aggregate SHA-256:
e9d8298486f31f40581cb4445fa0abac25030bd586303098c05e1a9225f0d0ea

E:\Project2026\RecoveryReports\LegacyFirebaseUserSecretRemovalV001
aggregate SHA-256:
b97b2eaed1738c497502c92b057c5133bf6b20345d302b3daea44541d0012dfa

E:\Project2026\RecoveryReports\CanonicalSyncFlowConsolidationV001
aggregate SHA-256:
a7d113ef381c07095b3ccd4145de734d4011e5eb51a78d2f6c7f6095ae868ccd
```

At minimum inspect complete current code for:

### WPF

```text
Price Rule UI/application Save entry
atomic TblTurnPolicy + TblTurnAmountRule + TblLocalOutbox transaction
OutboxPublisherWorker registration and ExecuteAsync loop
canonical public FlushOutbox entry
complete-group selection/validation
claim/lease
HTTP request construction
response validation
all-or-none mark-sent/retry
runtime source-client identity and sync-auth/session initialization
```

### API

```text
canonical grouped sync route/action
actual auth scheme/policy applied to that action
request tenant/source identity validation
complete-group validation
idempotency/conflict boundary
routing/subscription resolution
source-event and destination-delivery staging
transaction begin/SaveChanges/commit
post-commit SignalR invocation
hub/client registration and notification payload contract
```

Record before execution:

```text
PROMPT099_ARTIFACT_VERIFIED=true
PROMPT100_ARTIFACT_VERIFIED=true
PROMPT107_ARTIFACT_VERIFIED=true
PROMPT108_ARTIFACT_VERIFIED=true
PROMPT109_ARTIFACT_VERIFIED=true
PROMPT110_ARTIFACT_VERIFIED=true
TASK_SCOPE=ONE_MAIN_DEV_HAPPY_PATH_ONLY
FAILURE_MATRIX=DEFERRED
CANONICAL_WORKER_COUNT=1
CANONICAL_API_INGEST_ACTION_COUNT=1
CANONICAL_SIGNALR_PATH_COUNT=1
```

## Phase 1 — Re-prove the canonical Development lane

Use only:

```text
WPF:
E:\Project2026\4POS\NailSalonNet8

ApiServer:
E:\Project2026\1ApiServer\ApiServer01
```

Prove safely:

```text
WPF Environment = Development
WPF provider = Npgsql/PostgreSQL
WPF runtime DB equals the accepted report107 canonical target
WPF pending migrations = 0

API Environment = Development
API provider = Npgsql/PostgreSQL
API runtime DB equals the accepted report108/report109 canonical target
API pending migrations = 0

WPF DB != API DB
both hosts are loopback or approved local Development
neither target is production/customer/reference data
```

Do not reset either database.

Do not expose complete connection strings, credentials, passfile contents, private identifiers, or tokens.

If either runtime resolves a different DB, stop before writes with:

```text
BLOCKED_MAIN_DEV_RUNTIME_DB_DRIFT
```

## Phase 2 — Resolve the production sync authentication path

Map the exact call chain from WPF worker/service to the canonical API grouped endpoint.

Prove:

```text
actual authentication scheme/policy
how the Development source POS obtains or resumes its credential/session
required tenant/source claims or headers
how API binds authenticated identity to TenantGuid and SourceClientId
invalid/mismatched identity is rejected
Firebase email/password is absent
installation-only WpfJwt is not broadened beyond its current accepted scope
```

Use an existing local Development issuer/provider/session mechanism only when it exercises the production authorization policy and tenant/source validation.

A test signer/provider is acceptable only when already part of the current Development/integration architecture and the request still traverses the production auth policy. Do not add an auth bypass.

## Phase 3 — Minimal Development-only prerequisites

The canonical Development databases were reset and may be empty except for schema.

Create only what the existing runtime contracts require for this one happy path:

```text
one generated Development tenant identity
one generated source POS1 client identity
one generated destination POS2 client identity or equivalent destination subscriber mapping
active routing/subscriptions for exactly:
  TblTurnPolicy
  TblTurnAmountRule
minimal non-business prerequisites required by the real Price Rule Save boundary
```

Rules:

```text
use generated Development-only identities
never reuse Royal/production/customer identifiers
use existing canonical application/setup/test-fixture boundaries where available
no unrelated employee/service/customer/invoice/booking/history seed
no manual TblLocalOutbox insertion
no manual TblEventLog/TblEventDelivery insertion
no manual Price Rule outbox fabrication
```

The domain/outbox/event/delivery rows generated by the actual E2E operation are test output, not installation seed.

Document every prerequisite row and whether it is retained for the following failure-matrix task or cleaned up. Do not call persistent manual SQL inserts “runtime proof.”

If the existing routing model cannot resolve one concrete destination client without inventing a new transport, stop with:

```text
BLOCKED_MAIN_DEV_DESTINATION_ROUTING
```

## Phase 4 — Start the real loopback ApiServer and existing SignalR path

Start the actual ApiServer Development project from the canonical source lane.

Required proof:

```text
loopback-only binding
readiness/health succeeds
runtime DB is canonical API Development DB
canonical grouped sync action is reachable
production auth policy is active
one existing SignalR hub/publisher path is active
```

For notification proof, connect/register one Development-only destination listener through the existing SignalR client/hub contract when required.

The listener may be a test-only observer, but it must use the existing hub route, registration method, and receive method. It must not introduce a second publisher, second hub, second notification transport, or business-payload channel.

## Phase 5 — Generate one real local transaction group

Use the actual production Price Rule Save boundary accepted by prompt099.

Do not manually insert the outbox rows.

Use a new generated test rule/value so the operation is a true change, not a no-op.

Expected local atomic result for first-Save behavior when the clean Development state requires a Draft Policy:

```text
one TblTurnPolicy domain change
one TblTurnAmountRule domain change
one complete TblLocalOutbox group
same TransactionGuid/TenantGuid/SourceClientId
ExpectedEventCount = 2
SequenceNumber 1 = TblTurnPolicy I
SequenceNumber 2 = TblTurnAmountRule I
both rows Pending before worker upload
```

If the current Development state already contains an accepted Draft Policy, use the real resulting operation/count and explain it. Do not force an artificial insert contract that contradicts current state.

Required transaction proof:

```text
one DbContext
one explicit local PostgreSQL transaction
all domain + outbox writes commit together
no partial local state
```

## Phase 6 — Execute the existing periodic WPF outbox worker

The proof must include the existing production periodic worker/bot boundary, not only a direct lower-level HTTP service call.

Allowed execution forms:

```text
start the actual WPF Development runtime and observe one worker cycle
or invoke an existing production worker cycle/test hook that enters the same registered worker -> canonical FlushOutbox chain
```

Not allowed:

```text
new uploader harness
manual HTTP request assembled outside the canonical service
calling the API controller directly
manual outbox status updates
second timer/worker registration
```

Prove:

```text
one worker loop/cycle selected the group
one atomic group claim
one lease owner
complete-group validation passed
one grouped HTTP request
events ordered by SequenceNumber
no second production path invoked
```

## Phase 7 — Prove the physical API commit boundary

For the exact generated TransactionGuid, prove:

```text
production authentication passed
TenantGuid/SourceClientId matched authenticated identity
request group validation passed
one API database transaction began
TblEventLog rows inserted = ExpectedEventCount
TblEventDelivery rows for destination = ExpectedEventCount
TblEventDelivery rows for source = 0
same TransactionGuid throughout
sequences are contiguous and parent-first
one durable commit completed
no partial event/delivery state
```

For the common two-event/one-destination case, expected durable counts are:

```text
TblEventLog = 2
TblEventDelivery for destination POS2 = 2
TblEventDelivery for source POS1 = 0
```

If actual valid routing yields a different destination count, report and prove the exact canonical routing result. Do not broaden routing to make the count pass.

## Phase 8 — Prove post-commit SignalR ordering

Required order:

```text
API database commit succeeds
then existing SignalR publish is attempted
```

Required happy-path proof:

```text
SignalR publish attempted = yes
SignalR publish succeeded = yes
notification received by the registered Development destination listener when the current hub contract supports direct observation
notification contains only safe availability/routing metadata
no complete business payload sent through SignalR
one publisher path used
```

If the existing architecture intentionally has no connected listener requirement and success can only mean publish invocation completed, document the exact hub semantics and prove the call completed successfully after commit.

Do not treat SignalR as the data transport. Destination data remains in TblEventDelivery for later pull.

## Phase 9 — Prove all-or-none WPF completion

After the accepted durable API response, prove for every row in the exact local group:

```text
Sent/status changed together
SentAt populated
ServerEventSequence mapped when the current contract returns it
claim/lease fields cleared
no row remains Pending/Processing/Failed while another is Sent
no duplicate HTTP request for the happy path
```

Expected marker:

```text
SYNC_GROUP_MAIN_DEV_HAPPY_PATH_COMMITTED
```

## Phase 10 — Focused regression and architecture guards

Run focused tests for:

```text
production worker -> canonical FlushOutbox call chain
complete-group claim/validation
one grouped request
production auth/identity match
API atomic event/delivery commit
source exclusion
SignalR after commit
all-or-none local mark-sent
single-path architecture guards from prompt110
```

Expected:

```text
all pass
0 skipped
```

Build WPF and ApiServer after any correction.

If the physical happy path exposes a source defect, make only the smallest production-capable correction inside the accepted prompt099/prompt100/prompt110 boundaries, then rerun the complete happy path from a new generated TransactionGuid and rerun all focused/architecture tests.

Do not widen into the 15-case failure matrix in this task.

## End state

Leave:

```text
canonical WPF Development DB migration-current
canonical API Development DB migration-current
one documented successful Development transaction group
one canonical worker/uploader/API/commit/SignalR chain
no failure-injection setting active
no second production sync path
ApiServer/WPF process state documented
minimal Development-only identity/routing prerequisites documented
```

Do not mutate production/customer/reference databases.

## Required private artifact

Create a new versioned artifact:

```text
E:\Project2026\RecoveryReports\MainDevCanonicalSyncHappyPathV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ARTIFACT_VERIFICATION.md
DEV_RUNTIME_DB_PROOF.md
SYNC_AUTH_CALL_CHAIN.md
IDENTITY_ROUTING_PREREQUISITES.md
API_STARTUP_READINESS.md
SIGNALR_DESTINATION_REGISTRATION.md
PRICE_RULE_DOMAIN_SAVE_PROOF.md
LOCAL_TRANSACTION_GROUP_BEFORE_UPLOAD.md
PERIODIC_WORKER_CALL_CHAIN.md
GROUP_CLAIM_REQUEST_PROOF.md
API_AUTH_VALIDATION_PROOF.md
API_TRANSACTION_COMMIT_PROOF.md
EVENT_DELIVERY_COUNTS.md
SIGNALR_AFTER_COMMIT_PROOF.md
WPF_ALL_OR_NONE_COMPLETION.md
HAPPY_PATH_TIMELINE.md
FOCUSED_TEST_OUTPUT.txt
ARCHITECTURE_GUARD_OUTPUT.txt
FINAL_DEV_STATE.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Do not overwrite or delete earlier artifacts.

## Mandatory direct evidence

For every runtime boundary, include:

```text
repository-relative production file path
class/method
line range
caller -> callee chain
sanitized runtime timestamps/correlation markers
exact safe row counts before/after
transaction state
HTTP status/result code
SignalR ordering markers
```

For any changed method include complete BEFORE and AFTER bodies and unified diff.

Do not expose:

```text
passwords
complete connection strings
JWT/token values
passfile contents
private keys
raw production/customer identifiers
private business payload values
```

## Public report

Create and push only:

```text
report/report111.md
```

Include:

```text
Verdict
Prompt099 artifact SHA verified yes/no
Prompt100 artifact SHA verified yes/no
Prompt107 artifact SHA verified yes/no
Prompt108 artifact SHA verified yes/no
Prompt109 artifact SHA verified yes/no
Prompt110 artifact SHA verified yes/no
Canonical WPF runtime DB proof yes/no
WPF pending migrations count
Canonical API runtime DB proof yes/no
API pending migrations count
Production sync auth path proven yes/no
Firebase/email-password used yes/no
Development identity/routing prerequisites ready yes/no
ApiServer loopback readiness yes/no
Existing SignalR destination registration/listener proof yes/no/not-required-with-reason
Production Price Rule Save used yes/no
Local domain + outbox atomic commit yes/no
Local group expected/actual event count
Local group sequence/order proof yes/no
Existing periodic WPF worker used yes/no
Production worker cycle count
Grouped HTTP request count
Canonical API ingest action used yes/no
Production auth/identity validation passed yes/no
API transaction begin count
API durable commit count
TblEventLog group row count
TblEventDelivery destination row count
TblEventDelivery source row count
Source exclusion proof yes/no
SignalR attempted after commit yes/no
SignalR publish succeeded yes/no
Destination notification observed yes/no/not-required-with-reason
WPF group rows marked Sent count
All-or-none local completion proof yes/no
Parallel production path invoked count
Happy-path marker present yes/no
Focused test totals
Architecture guard totals
WPF build totals
API build totals
WPF/API DB reset performed yes/no
Manual outbox/event/delivery insertion used yes/no
Production/customer/reference DB mutated yes/no
Private artifact yes/no
Aggregate SHA-256
```

## Verdicts

PASS only when one physical main-Development transaction group completes through the one canonical production chain:

```text
OBM_MAIN_DEV_CANONICAL_SYNC_HAPPY_PATH_READY_FOR_FAILURE_MATRIX
```

Narrow blockers only:

```text
BLOCKED_MAIN_DEV_ARTIFACT_VERIFICATION
BLOCKED_MAIN_DEV_RUNTIME_DB_DRIFT
BLOCKED_MAIN_DEV_SYNC_AUTH
BLOCKED_MAIN_DEV_IDENTITY_PREREQUISITES
BLOCKED_MAIN_DEV_DESTINATION_ROUTING
BLOCKED_MAIN_DEV_API_STARTUP
BLOCKED_MAIN_DEV_SIGNALR_REGISTRATION
BLOCKED_MAIN_DEV_PRICE_RULE_SAVE
BLOCKED_MAIN_DEV_PERIODIC_WORKER
BLOCKED_MAIN_DEV_GROUP_UPLOAD
BLOCKED_MAIN_DEV_API_VALIDATION
BLOCKED_MAIN_DEV_API_COMMIT
BLOCKED_MAIN_DEV_SIGNALR_AFTER_COMMIT
BLOCKED_MAIN_DEV_LOCAL_COMPLETION
BLOCKED_MAIN_DEV_HAPPY_PATH_TESTS
```

A blocked result must identify the exact failed boundary, exact production class/method, sanitized exception chain, SQLSTATE when available, safe row/transaction state, and whether either Development DB was mutated.

Do not return a generic E2E blocker.
