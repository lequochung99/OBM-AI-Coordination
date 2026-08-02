# Prompt 115 — Resolve the canonical API destination-routing table contract and complete the physical POS1 → API happy path through the canonical Provider

## Starting checkpoint

Prompt114 returned:

```text
BLOCKED_MAIN_DEV_DESTINATION_ROUTING
```

Coordination references:

```text
report/report114.md
report114 commit:
4746d3e668d469a549aa0f69068527e382f8317d

prompt114 private artifact aggregate SHA-256:
29013015c51314c63748cb174754a1571e3677ecd1854d4089a70ed5c1a7e22c
```

Report114 proves the earlier infrastructure blockers are closed:

```text
protected provisioning source recovered = OBM_PLATFORM_V2_P6_POS_PG_ADMIN
canonical grant/provisioning source-of-truth corrected
runtime role CREATEDB true -> false
runtime role is NOSUPERUSER/NOCREATEDB/NOCREATEROLE and not an owner
no broad GRANT ALL
migration history proof = yes
API pending migrations = 0
runtime positive and negative permission proofs = yes
ApiServer starts non-interactively on loopback against the canonical API Development DB
admin/provisioning credential is absent from the normal runtime process
```

The new exact blocker is:

```text
current API destination-routing code queries TblTenantPosDevice
current mapped schema/migration does not contain TblTenantPosDevice
current canonical schema contains TblPosLocal
there is not yet enough direct contract evidence to replace, remap, or create either table safely
```

No Development identity/routing prerequisites, Price Rule Save, outbox upload, EventLog/Delivery write, or SignalR publish occurred.

## Authoritative user locks

Do not begin Service Category Weight or Booking Weight in this task or after a blocked result.

Do not tell the user to start manual POS1 testing until this task physically proves the complete canonical happy path and returns:

```text
MANUAL_POS1_TEST_READY=true
```

Every blocker must return:

```text
MANUAL_POS1_TEST_READY=false
```

## Canonical Provider lock

The WPF caller must not manage token details.

The outbound path remains:

```text
WPF domain Save + TblLocalOutbox
-> existing periodic WPF outbox worker
-> existing canonical Provider
-> Provider obtains/resumes the existing application credential internally
-> Provider attaches canonical authentication and application-identity headers internally
-> canonical API grouped-sync endpoint
-> API authenticates the application's runtime/legal identity
-> API validates authenticated TenantGuid and SourceClientId
-> destination routing
-> one EventLog/Delivery transaction and commit
-> existing post-commit SignalR notification
```

Do not add manual token access, manual Authorization header construction, direct HttpClient sync, a new Provider, a new token service, or a parallel authentication path.

## Canonical sync lock

There remains exactly one production sync flow:

```text
ONE TblLocalOutbox
ONE periodic WPF worker
ONE canonical Provider/API service path
ONE grouped API ingest action/service
ONE EventLog/Delivery transaction path
ONE post-commit SignalR publisher
```

Do not create a second uploader, worker, endpoint, routing service, delivery transport, ACK path, or SignalR path.

## Strict scope

Execute only:

```text
1. Read and verify the complete prompt114 private artifact.
2. Audit the full historical/current contract of TblTenantPosDevice and TblPosLocal.
3. Determine the one canonical table/model boundary for active API destination POS identity and routing.
4. Correct the smallest production-capable model/migration/routing defect at its source.
5. Apply any proven attached migration and explicit runtime grants through the accepted migration/provisioning boundary.
6. Prove one generated Development POS1 source and one generated Development POS2 destination can be resolved through the canonical routing/subscription contract.
7. Resume and complete the physical Price Rule POS1 -> API grouped-sync happy path through the existing periodic worker and canonical Provider.
8. Rerun focused tests, architecture guards, and WPF/API builds.
```

Do not execute:

```text
DB reset/drop/recreate
15-case failure/recovery matrix
POS2 pull/apply/ACK
Service Category Weight
Booking Weight
checkout/payment changes
Queue changes
BookingConsole changes
cloud or production deployment
```

Do not mutate production/customer/reference databases.

## Required evidence intake

Read completely:

```text
prompt/prompt095.md
report/report095.md
prompt/prompt098.md
report/report098.md
prompt/prompt100.md
report/report100.md
prompt/prompt103.md
report/report103.md
prompt/prompt108.md
report/report108.md
prompt/prompt110.md
report/report110.md
prompt/prompt111.md
report/report111.md
prompt/prompt112.md
report/report112.md
prompt/prompt113.md
report/report113.md
prompt/prompt114.md
prompt/prompt114_CANONICAL_PROVIDER_AUTH_ADDENDUM.md
report/report114.md
prompt/prompt112_SQL_TEMPLATE_POLICY_ADDENDUM.md
```

Read and verify at minimum:

```text
E:\Project2026\RecoveryReports\WholeGroupUploaderApiIngestV001
aggregate SHA-256:
9e4ef1e4df63373abb052c055e7a17c75efbca76a81a315b19a0d513fc9bcf42

E:\Project2026\RecoveryReports\MainApiDevResetExecutionV001
aggregate SHA-256:
e9d8298486f31f40581cb4445fa0abac25030bd586303098c05e1a9225f0d0ea

E:\Project2026\RecoveryReports\CanonicalSyncFlowConsolidationV001
aggregate SHA-256:
a7d113ef381c07095b3ccd4145de734d4011e5eb51a78d2f6c7f6095ae868ccd

prompt114 private artifact:
E:\Project2026\RecoveryReports\MainDevApiRoleContractAndSyncHappyPathV001
aggregate SHA-256:
29013015c51314c63748cb174754a1571e3677ecd1854d4089a70ed5c1a7e22c
```

At minimum inspect complete current and historical source for:

### API identity/routing schema

```text
all entity classes named or mapped to TblTenantPosDevice
all entity classes named or mapped to TblPosLocal
all ExternalDbContext DbSet and Fluent API registrations for both concepts
all migrations and model snapshots containing either name
all SQL/bootstrap/import/provisioning scripts containing either name
all repository/service/controller queries reading or writing either concept
all TblSubscription mappings and destination-resolution call sites
all source-exclusion logic
all APIs that create/register/activate/deactivate POS stations/devices
all PlatformAppV0 pairing/POS assignment persistence call chains
all installation/bootstrap contracts that materialize POS identity into the API database
all legacy PlatformV2/V3 paths only as historical evidence; do not reactivate deleted architectures
```

### Identity field semantics

Prove the actual owner and meaning of:

```text
TenantGuid
PosGuid
PosDeviceGuid
PosStationId or station slot
SourceClientId
DestinationClientId
SubscriberId
installation identity
active/enabled status
last-seen/runtime status when relevant
```

### Provider/auth path

Inspect and prove:

```text
canonical Provider interface and implementation
periodic worker -> Provider call chain
Provider-owned credential/session resolution
Provider-owned auth and identity headers
API authentication policy and application identity binding
TenantGuid/SourceClientId match validation
```

Never expose tokens, passwords, complete connection strings, protected values, raw Development identities, or business payload values.

Record before editing:

```text
PROMPT114_ARTIFACT_VERIFIED=true
CANONICAL_API_DB=obm_api_dev_v0_pg
API_PENDING_MIGRATIONS_BEFORE=0
ROLE_CONTRACT_ACCEPTED=true
MANUAL_POS1_TEST_READY=false
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
DB_RESET=FORBIDDEN
PARALLEL_ROUTING_TABLES=FORBIDDEN
PARALLEL_SYNC_PATH=FORBIDDEN
```

## Phase 1 — Produce a complete table/model lineage audit

For both `TblTenantPosDevice` and `TblPosLocal`, produce direct evidence:

```text
concept name
entity class
physical table mapping
DbSet presence
migration creation/drop/rename history
model snapshot presence
primary key
unique keys
foreign keys
required identity fields
CRUD writers
runtime readers
Platform/pairing writers
sync-routing readers
current physical existence in canonical API DB
current row count in Development DB
classification
```

Allowed classifications:

```text
CANONICAL_ACTIVE_TABLE
STALE_CODE_ONLY_CONCEPT
MISSING_ATTACHED_MIGRATION
LEGACY_PLATFORM_CONCEPT
SAME_CONCEPT_DIFFERENT_NAME
DISTINCT_CONCEPT_REQUIRED
UNMAPPED_OR_DEAD_ENTITY
INSUFFICIENT_PROOF_BLOCK
```

Do not decide from names alone.

Produce a field-by-field semantic comparison. A same-name-like field is not sufficient; prove key generation, lifecycle, writer ownership, and source/destination identity format.

## Phase 2 — Select exactly one canonical routing contract

Choose exactly one outcome based on complete evidence.

### Outcome R1 — TblPosLocal is the canonical active routing table

Use only when evidence proves `TblPosLocal` owns the active tenant/POS/device/client identity required by sync routing.

Then:

```text
update the existing canonical routing query/service to use TblPosLocal
remove or make unreachable stale TblTenantPosDevice production references
preserve existing SourceClientId/DestinationClientId formatting and active-state semantics
use TblSubscription or the proven existing subscription owner consistently
add regression guards preventing TblTenantPosDevice from returning as a production dependency
```

Do not create a compatibility view/table named TblTenantPosDevice merely to satisfy stale code.

### Outcome R2 — TblTenantPosDevice is a distinct canonical table whose migration is missing

Use only when complete Platform/pairing/installation/routing evidence proves it has semantics that `TblPosLocal` cannot represent safely.

Then:

```text
repair the real entity mapping
create one attached ExternalDbContext migration
update the model snapshot
apply the migration through the accepted provisioning/migration boundary
apply explicit runtime privileges through the accepted grant manifest
prove pending migrations = 0
prove Platform/pairing or canonical setup boundary writes the required rows
```

Do not manually create the table or patch only the physical DB.

### Outcome R3 — Same physical concept with a proven rename/mapping defect

Use only when lineage proves a single table was renamed or mapped inconsistently.

Then repair the entity/table mapping and attached migration history safely. Do not retain two physical tables or two independently writable entity sets.

### Outcome R4 — Another narrow canonical design

Allowed only with complete direct evidence and the same single-owner/single-table guarantees.

## Forbidden routing fixes

Do not implement:

```text
query TblTenantPosDevice, then fallback to TblPosLocal
UNION both tables
try/catch missing-table and use the other table
create both tables and keep them synchronized
copy POS rows between two competing tables
hardcode POS2 destination IDs
route to every POS client indiscriminately
route by tenant alone without active subscription/client proof
bypass source exclusion
manual delivery insertion
second destination-routing service
```

If the contract remains unprovable after complete audit, stop with:

```text
BLOCKED_MAIN_DEV_DESTINATION_ROUTING_CONTRACT
```

and provide the exact conflicting evidence. Do not guess.

## Phase 3 — Persist the correction at the source of truth

The correction must update the existing canonical source only:

```text
ExternalDbContext model/mapping when required
attached EF/Npgsql migration when required
existing destination-routing service/query
existing subscription registration/lookup
existing provisioning grant manifest when schema objects change
focused architecture/regression tests
```

Requirements:

```text
one canonical routing table/model owner
one routing service/query chain
one subscription contract
one SourceClientId format
one destination-client resolution result per active destination
source client excluded
no production fallback path
```

If a migration is added:

```text
apply it without DB reset
prove accepted migration history exact-once
pending migrations = 0
runtime role remains least privilege
runtime explicit grants cover only required new objects
record the schema/role change for later derived SQL export under E:\Project2026\2SQL PostgreSQL
```

Do not finalize or overwrite the two SQL templates yet.

## Phase 4 — Prove Development routing prerequisites through canonical writers

Create only Development-only prerequisites through existing canonical setup/application boundaries:

```text
one generated tenant
one generated POS1 source identity
one generated POS2 destination identity
active registration/status required by the selected canonical table contract
subscriptions for exactly TblTurnPolicy and TblTurnAmountRule
```

Rules:

```text
no Royal/production/customer identifiers
no hardcoded reusable Development identifiers in production source
no direct manual insert into TblLocalOutbox, TblEventLog, or TblEventDelivery
no fake destination table rows that bypass the canonical writer contract
```

Prove:

```text
source POS1 resolves to one authenticated SourceClientId
POS2 resolves to one concrete active DestinationClientId
both belong to the same generated Development tenant
subscriptions select POS2 for both supported entity types
POS1 is excluded as a destination
no unrelated destination is selected
```

Expected routing result for the one-destination setup:

```text
destination client count = 1
source destination count = 0
```

## Phase 5 — Re-prove canonical Provider authentication

Before domain Save, prove without exposing values:

```text
canonical Provider identified
existing periodic worker calls that Provider
Provider owns credential/session resolution
Provider owns authorization and application-identity headers
manual token access outside Provider count = 0
manual auth-header construction outside Provider count = 0
parallel Provider/direct HttpClient sync path count = 0
API application identity authentication succeeds
API authenticated TenantGuid/SourceClientId matches the request identity
```

Do not broaden installation-scoped WpfJwt or restore Firebase email/password.

## Phase 6 — Complete the physical POS1 → API happy path

### Production Price Rule Save

Use the production Price Rule Save boundary to create one true Development-only change.

Prove:

```text
one DbContext
one explicit local PostgreSQL transaction
domain rows + complete TblLocalOutbox group commit atomically
ExpectedEventCount equals actual group count
contiguous SequenceNumber values
TblTurnPolicy parent first when created
no partial local state
```

Do not manually fabricate outbox rows.

### Existing periodic worker and Provider

Use the registered production periodic worker.

Prove:

```text
one worker cycle
one atomic group claim
one canonical Provider invocation
one grouped HTTP request
zero manual token/header handling outside Provider
zero parallel production path invocations
```

### API routing and durable commit

For the generated TransactionGuid, prove:

```text
production application identity authentication passes
TenantGuid/SourceClientId identity match passes
canonical destination-routing query uses the selected table contract
one destination client is selected
source client is excluded
one API transaction begins
TblEventLog count = ExpectedEventCount
TblEventDelivery destination count = ExpectedEventCount
TblEventDelivery source count = 0
one durable commit
no partial durable state
```

### SignalR and local completion

Prove:

```text
API commit completes before SignalR publish attempt
existing SignalR publisher only
notification-only metadata, no business payload
publish succeeds
all local group rows become Sent together
SentAt populated
claim/lease cleared
no mixed local status
```

Required marker:

```text
SYNC_GROUP_MAIN_DEV_HAPPY_PATH_COMMITTED
```

## Phase 7 — Tests, guards, and builds

Run focused tests for:

```text
TblTenantPosDevice/TblPosLocal lineage decision
single canonical routing-table dependency
canonical destination resolution
active-state filtering
subscription filtering for TblTurnPolicy and TblTurnAmountRule
source exclusion
no fallback/union/dual-table routing
canonical Provider ownership of token/header handling
Provider application-identity authentication
production worker -> Provider -> grouped endpoint chain
API atomic EventLog/Delivery commit
SignalR after commit
all-or-none local completion
```

Rerun:

```text
prompt110 single-sync-path architecture guards
prompt114 role-contract and least-privilege tests
```

Expected:

```text
all pass
0 skipped
parallel production paths = 0
parallel routing table paths = 0
```

Build:

```text
WPF
ApiServer
PlatformAppV0 only if its canonical POS registration writer changed
```

Build/test success does not override failed physical routing or happy-path proof.

## End state

PASS requires:

```text
one proven canonical API POS identity/routing table contract
no TblTenantPosDevice/TblPosLocal ambiguity in production routing
pending migrations = 0
runtime role remains least privilege
canonical Provider owns all sync authentication/header behavior
one physical POS1 -> API grouped-sync happy path committed
one destination delivery set created for POS2
source POS1 excluded
SignalR after commit
local outbox group Sent atomically
Category Weight unchanged
Booking Weight unchanged
MANUAL_POS1_TEST_READY=true
```

Do not tell the user to test before this PASS state.

## Required private artifact

Preserve earlier artifacts unchanged. Create:

```text
E:\Project2026\RecoveryReports\MainDevDestinationRoutingAndSyncHappyPathV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ARTIFACT_VERIFICATION.md
PROMPT114_BLOCKER_INTAKE.md
TABLE_MODEL_LINEAGE_AUDIT.md
TBLTENANTPOSDEVICE_CONTRACT.md
TBLPOSLOCAL_CONTRACT.md
FIELD_SEMANTIC_COMPARISON.md
ROUTING_CONTRACT_DECISION.md
ROUTING_SOURCE_BEFORE.md
ROUTING_SOURCE_AFTER.md
MIGRATION_DECISION.md
MIGRATION_AND_GRANT_PROOF.md
CANONICAL_POS_WRITER_CALL_CHAIN.md
DEVELOPMENT_ROUTING_PREREQUISITES.md
DESTINATION_RESOLUTION_PROOF.md
SOURCE_EXCLUSION_PROOF.md
CANONICAL_PROVIDER_CALL_CHAIN.md
PROVIDER_APPLICATION_IDENTITY_PROOF.md
NO_MANUAL_TOKEN_HANDLING_PROOF.md
PRICE_RULE_SAVE_PROOF.md
WPF_WORKER_PROOF.md
GROUPED_HTTP_PROOF.md
API_TRANSACTION_PROOF.md
SIGNALR_AFTER_COMMIT_PROOF.md
LOCAL_COMPLETION_PROOF.md
HAPPY_PATH_COUNTS.md
MANUAL_TEST_READINESS.md
FOCUSED_TEST_OUTPUT.txt
ARCHITECTURE_GUARD_OUTPUT.txt
ROLE_CONTRACT_REGRESSION_OUTPUT.txt
FINAL_STATE.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

## Public report

Create and push only:

```text
report/report115.md
```

Include:

```text
Verdict
Prompt114 artifact SHA verified yes/no
Role contract remains accepted yes/no
Runtime role CREATEDB after
API pending migrations count
TblTenantPosDevice classification
TblPosLocal classification
Canonical routing table/model selected
Routing contract decision R1/R2/R3/R4
Attached migration created yes/no and identifier/not-applicable
Migration/grants applied through canonical boundary yes/no/not-applicable
Parallel routing table paths remaining count
Canonical POS identity writer identified yes/no
Development POS1 source created yes/no
Development POS2 destination created yes/no
Subscriptions created for TblTurnPolicy/TblTurnAmountRule yes/no
Canonical destination resolution proof yes/no
Destination client count
Source exclusion proof yes/no
Unrelated destination count
Canonical WPF Provider identified yes/no
Existing periodic worker used canonical Provider yes/no
Canonical Provider invocation count
Provider resolved/resumed credential internally yes/no
Provider attached canonical auth/identity headers internally yes/no
Manual token access outside Provider count
Manual auth-header construction outside Provider count
API application identity authenticated yes/no
Authenticated tenant/source identity matched yes/no
Parallel Provider/direct HttpClient path count
Production Price Rule Save used yes/no
Local domain + outbox atomic commit yes/no
Local group expected/actual count
Local group sequence/order proof yes/no
Production worker cycle count
Grouped HTTP request count
Canonical API ingest action used yes/no
API transaction begin count
API durable commit count
TblEventLog group row count
TblEventDelivery destination row count
TblEventDelivery source row count
SignalR attempted after commit yes/no
SignalR publish succeeded yes/no
Destination notification observed yes/no/not-applicable
WPF group rows marked Sent count
All-or-none local completion proof yes/no
Parallel production sync path count
Happy-path marker present yes/no
Manual POS1 test ready true/false
Focused test totals
Architecture guard totals
Role-contract regression totals
WPF build totals
API build totals
PlatformAppV0 build totals/not changed
WPF/API DB reset performed yes/no
Manual outbox/event/delivery insertion used yes/no
Category Weight changed yes/no
Booking Weight changed yes/no
Production/customer/reference DB mutated yes/no
Private artifact yes/no
Aggregate SHA-256
```

Do not expose passwords, tokens, complete connection strings, protected values, raw Development identities, or business payload values.

## Verdicts

PASS:

```text
OBM_MAIN_DEV_DESTINATION_ROUTING_AND_CANONICAL_SYNC_HAPPY_PATH_READY_FOR_MANUAL_POS1_TEST
```

Narrow blockers only:

```text
BLOCKED_MAIN_DEV_DESTINATION_ROUTING_CONTRACT
BLOCKED_MAIN_DEV_DESTINATION_ROUTING_SCHEMA
BLOCKED_MAIN_DEV_DESTINATION_ROUTING_WRITER
BLOCKED_MAIN_DEV_DESTINATION_ROUTING
BLOCKED_MAIN_DEV_CANONICAL_PROVIDER_CREDENTIAL
BLOCKED_MAIN_DEV_SYNC_AUTH
BLOCKED_MAIN_DEV_PRICE_RULE_SAVE
BLOCKED_MAIN_DEV_GROUP_UPLOAD
BLOCKED_MAIN_DEV_API_COMMIT
BLOCKED_MAIN_DEV_SIGNALR
BLOCKED_MAIN_DEV_LOCAL_COMPLETION
BLOCKED_MAIN_DEV_HAPPY_PATH_TESTS
```

Every blocker must include:

```text
exact conflicting table/model/migration/writer evidence
exact failed class/method/query/test
sanitized exception and SQLSTATE when available
whether a migration/grant change was applied
all Development write state
Category Weight changed = no
Booking Weight changed = no
MANUAL_POS1_TEST_READY=false
```

Do not return a generic routing blocker and do not guess the canonical table contract.
