# Prompt 116 — Attach the canonical TblTenantPosDevice schema and writer lifecycle, then complete the physical POS1 → API happy path

## Starting checkpoint

Prompt115 returned:

```text
BLOCKED_MAIN_DEV_DESTINATION_ROUTING_SCHEMA
```

Coordination references:

```text
report/report115.md
report115 commit:
566c4277c2ba8dacc180ce72a7024c7100dfca1c

prompt115 private artifact aggregate SHA-256:
9d009c66e7b02f1e16aab7fdccc88fdf8ffd19f0ecbcd756c4b94f35acaf5b75
```

Report115 proves:

```text
prompt114 artifact verified
API role contract remains accepted
runtime CREATEDB=false
API pending migrations=0
TblPosLocal is the canonical active table for logical POS rows only
TblPosLocal is not proven sufficient for runtime device/destination routing
TblTenantPosDevice has a distinct historical runtime POS-device/routing contract
R2 is semantically indicated
current ExternalDbContext migration/current DB lacks TblTenantPosDevice
no current canonical writer was proven to persist TblTenantPosDevice
no Development routing prerequisites or sync writes occurred
MANUAL_POS1_TEST_READY=false
```

This task must establish the exact active writer lifecycle before creating schema, attach the real `TblTenantPosDevice` model/migration to `ExternalDbContext`, apply the existing least-privilege grants, materialize Development POS1/POS2 through the canonical writer, and complete the physical happy path.

## Authoritative user locks

Do not begin Service Category Weight or Booking Weight in this task or after a blocked result.

Do not tell the user to test manually until this task returns PASS with:

```text
MANUAL_POS1_TEST_READY=true
```

Every blocker must return:

```text
MANUAL_POS1_TEST_READY=false
```

## Canonical identity separation

Preserve the distinct concepts proven by prompt115:

```text
TblPosLocal
- logical POS/station row
- slot/name/configuration semantics proven by current active schema
- not automatically the runtime device/destination-routing owner

TblTenantPosDevice
- physical/runtime POS device identity and routing registration
- distinct lifecycle and SourceClientId/DestinationClientId semantics
- must have one canonical writer and one attached schema owner
```

Do not merge the two concepts merely because they share TenantGuid/PosGuid-like fields.

Do not make both tables competing routing sources.

## Canonical Provider and sync locks

The WPF caller does not manage token details.

The only outbound path remains:

```text
WPF domain Save + TblLocalOutbox
-> existing periodic WPF worker
-> existing canonical Provider
-> Provider resolves/resumes application credential internally
-> Provider attaches canonical authentication/application-identity headers internally
-> canonical API grouped-sync action
-> API authenticates application identity and matches TenantGuid/SourceClientId
-> canonical destination routing through TblTenantPosDevice + existing subscription contract
-> one EventLog/Delivery transaction and commit
-> existing post-commit SignalR notification
```

Do not add manual token access, manual Authorization/header construction outside the Provider, a new Provider, direct HttpClient sync, a second worker, endpoint, routing service, delivery path, ACK path, or SignalR publisher.

## Strict scope

Execute only:

```text
1. Read and verify the complete prompt115 private artifact.
2. Recover the exact historical and intended current writer lifecycle for TblTenantPosDevice.
3. Prove the exact minimal TblTenantPosDevice entity/mapping/schema contract from real call sites and historical artifacts.
4. Reconnect or implement the smallest production-capable writer inside the existing active PlatformAppV0/API pairing-registration boundary.
5. Only after the writer contract is proven, create one attached ExternalDbContext migration and update the model snapshot.
6. Apply the migration through the accepted migration/provisioning boundary without DB reset.
7. Reapply explicit least-privilege runtime grants through the accepted grant manifest.
8. Prove pending migrations=0 and physical writer/read/routing behavior.
9. Create Development POS1/POS2 and subscriptions through canonical application/setup writers.
10. Complete the physical Price Rule POS1 → API happy path through the periodic worker and canonical Provider.
11. Rerun focused tests, architecture guards, role-contract tests, and builds.
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
cloud/production deployment
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
prompt/prompt109.md
report/report109.md
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
prompt/prompt115.md
report/report115.md
prompt/prompt112_SQL_TEMPLATE_POLICY_ADDENDUM.md
```

Read and verify:

```text
prompt115 private artifact:
E:\Project2026\RecoveryReports\MainDevDestinationRoutingAndSyncHappyPathV001
aggregate SHA-256:
9d009c66e7b02f1e16aab7fdccc88fdf8ffd19f0ecbcd756c4b94f35acaf5b75

prompt114 private artifact:
E:\Project2026\RecoveryReports\MainDevApiRoleContractAndSyncHappyPathV001
aggregate SHA-256:
29013015c51314c63748cb174754a1571e3677ecd1854d4089a70ed5c1a7e22c

prompt110 private artifact:
E:\Project2026\RecoveryReports\CanonicalSyncFlowConsolidationV001
aggregate SHA-256:
a7d113ef381c07095b3ccd4145de734d4011e5eb51a78d2f6c7f6095ae868ccd
```

At minimum inspect complete current and retained historical source for:

```text
TblTenantPosDevice entity classes and mappings
all former migrations/DDL for TblTenantPosDevice
all former create/update/activate/deactivate writers
all PlatformAppV0 tenant/POS selection and pairing-code creation paths
pairing redeem and WpfJwt issuance path
bootstrap/me and installation checkpoint flow
POS station/device activation and replacement flows
TblPosLocal current writers/readers and logical-station lifecycle
SourceClientId construction and validation
TblSubscription creation/lookup
canonical API destination-routing query
source exclusion
installation/reinstallation/device-replacement behavior
```

Historical PlatformV2/V3 code may be used as contract evidence only. Do not reactivate deleted architecture wholesale.

Never expose tokens, passwords, complete connection strings, protected values, raw Development identities, or business payload values.

Record before editing:

```text
PROMPT115_ARTIFACT_VERIFIED=true
ROLE_CONTRACT_ACCEPTED=true
RUNTIME_CREATEDB=false
API_PENDING_MIGRATIONS_BEFORE=0
TBLPOSLOCAL_ROLE=LOGICAL_POS_ONLY
TBLTENANTPOSDEVICE_ROLE=RUNTIME_DEVICE_ROUTING
MANUAL_POS1_TEST_READY=false
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
DB_RESET=FORBIDDEN
PARALLEL_ROUTING_TABLES=FORBIDDEN
PARALLEL_SYNC_PATH=FORBIDDEN
```

## Phase 1 — Prove the canonical writer lifecycle before schema generation

Produce a complete writer-lifecycle decision for `TblTenantPosDevice`:

```text
WRITER_OWNER_PROJECT=<PlatformAppV0/API/existing setup boundary>
WRITER_SERVICE_CLASS=<exact class>
WRITER_METHOD=<exact method>
CREATE_TRIGGER=<tenant/POS selection, pairing-code issue, redeem, bootstrap completion, activation, other>
UPDATE_TRIGGER=<restart/redeem/heartbeat/device replacement/other>
DEACTIVATE_TRIGGER=<replacement/revocation/other>
IDEMPOTENCY_KEY=<exact proven key>
TENANT_BINDING=<exact proven field>
POS_STATION_BINDING=<exact proven field>
DEVICE_BINDING=<exact proven field>
SOURCE_CLIENT_ID_OWNER=<exact writer/helper>
ACTIVE_STATUS_OWNER=<exact field/service>
```

The lifecycle must be justified from complete call chains and state transitions.

Do not create the migration until this writer contract is proven.

If no safe active writer lifecycle can be proven, stop with:

```text
BLOCKED_MAIN_DEV_POS_DEVICE_WRITER_CONTRACT
```

Do not create a table with no canonical writer.

## Phase 2 — Define the exact minimal entity and schema contract

Using direct evidence, define only required fields and constraints.

At minimum resolve:

```text
primary key
TenantGuid
PosGuid and/or PosStationId relation
PosDeviceGuid or exact physical-device key
SourceClientId/DestinationClientId representation
SubscriberId when stored versus derived
active/enabled/revoked status
created/updated/activated timestamps under the established PostgreSQL timestamp policy
installation/attempt identity only when proven required
unique keys preventing duplicate active device/client registrations
foreign keys to canonical tenant/POS station owners when physically supported
indexes required by destination routing and activation lookup
```

Do not copy every historical column automatically.

Do not add speculative heartbeat, network, hardware, secret, token, or business fields.

Document why `TblPosLocal` cannot replace this contract and why it remains a separate logical POS table.

## Phase 3 — Implement the writer inside the active canonical boundary

Reconnect or implement the smallest writer in the existing active PlatformAppV0/API pairing-registration lifecycle.

Requirements:

```text
one canonical writer service/method
idempotent create/update
same tenant/POS/device/client identity contract used by routing
no direct raw-SQL writer duplicated beside EF/application service
no second registration endpoint solely for sync
no manual Development-only production path
no token persisted in TblTenantPosDevice
no PostgreSQL credential or WpfJwt stored in the device table
```

The writer must participate in the existing pairing/activation transaction or an explicitly proven adjacent atomic boundary.

If pairing code issuance is too early and redeem/activation is the correct lifecycle, write at the correct later point. Do not choose based on convenience.

Add tests for:

```text
first registration
idempotent replay/restart
same station with same device
same station with replacement device
cross-tenant mismatch rejected
SourceClientId uniqueness
active-device selection
revoked/inactive device excluded
```

## Phase 4 — Attach schema through ExternalDbContext migration

Only after Phases 1–3 are proven:

```text
add the canonical DbSet/mapping if missing
create one semantic attached Npgsql migration
update ExternalDbContext model snapshot
create TblTenantPosDevice with only proven columns/constraints/indexes
preserve TblPosLocal unchanged as logical POS table
```

Do not manually create the table.

Do not create a compatibility view.

Do not rename TblPosLocal to TblTenantPosDevice.

Apply through the accepted protected migration/provisioning boundary.

Reapply the explicit runtime grant manifest for the new table/index sequences as required.

Required proof:

```text
migration history contains new migration exactly once
pending migrations=0
runtime role remains NOSUPERUSER/NOCREATEDB/NOCREATEROLE
runtime role is not object/schema/database owner
no broad GRANT ALL
runtime has only explicit required DML/SELECT privileges
rolled-back write/read/update proof through runtime/application boundary
negative DDL/admin proof remains PASS
```

Record the new schema/role contract for the later derived SQL export under:

```text
E:\Project2026\2SQL PostgreSQL
```

Do not finalize or overwrite the two SQL templates yet.

## Phase 5 — Prove canonical Development POS registration and routing

Create through the canonical application/setup writer:

```text
one generated Development tenant
one logical POS1 row through the canonical TblPosLocal/station owner when required
one logical POS2 row through the same owner when required
one active TblTenantPosDevice row for POS1 source
one active TblTenantPosDevice row for POS2 destination
subscriptions for exactly TblTurnPolicy and TblTurnAmountRule
```

No raw manual inserts into `TblTenantPosDevice`, `TblLocalOutbox`, `TblEventLog`, or `TblEventDelivery` are allowed.

A test fixture may invoke the real canonical writer/service, but may not bypass it with direct entity insertion.

Prove:

```text
POS1 authenticated SourceClientId resolves to exactly one active device row
POS2 resolves to exactly one active destination client row
same generated Development tenant
POS2 selected for TblTurnPolicy and TblTurnAmountRule
POS1 excluded as destination
inactive/revoked/unsubscribed devices excluded
unrelated destination count=0
destination client count=1
```

## Phase 6 — Re-prove canonical Provider authentication

Prove physically, without exposing values:

```text
existing periodic worker calls the canonical Provider
Provider invocation count=1 for the group
Provider owns credential/session resolution
Provider owns authorization/application-identity headers
manual token access outside Provider count=0
manual auth-header construction outside Provider count=0
parallel Provider/direct HttpClient path count=0
API application identity authentication succeeds
authenticated TenantGuid/SourceClientId matches the registered POS1 device identity
```

Do not broaden installation-scoped WpfJwt and do not restore Firebase email/password.

## Phase 7 — Complete the physical POS1 → API happy path

### Production Price Rule Save

Use the production Price Rule Save boundary to create one true Development-only change.

Prove:

```text
one DbContext
one explicit local PostgreSQL transaction
domain rows + complete TblLocalOutbox group commit atomically
ExpectedEventCount equals actual group count
contiguous SequenceNumber
TblTurnPolicy parent first when created
no partial local state
```

Do not fabricate outbox rows.

### Periodic worker and Provider

Use the registered production periodic worker.

Prove:

```text
one worker cycle
one atomic group claim
one canonical Provider invocation
one grouped HTTP request
zero parallel production-path invocations
```

### API routing and commit

For the generated TransactionGuid, prove:

```text
application identity authentication passes
TenantGuid/SourceClientId match passes
routing reads the canonical TblTenantPosDevice contract
one active subscribed POS2 destination selected
source POS1 excluded
one API transaction begins
TblEventLog count=ExpectedEventCount
TblEventDelivery destination count=ExpectedEventCount
TblEventDelivery source count=0
one durable commit
no partial durable state
```

### SignalR and local completion

Prove:

```text
API commit completes before SignalR publish attempt
existing SignalR publisher only
notification-only metadata
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

## Phase 8 — Tests, architecture guards, and builds

Run focused tests for:

```text
TblTenantPosDevice writer lifecycle
writer idempotency and replacement/revocation behavior
entity constraints and unique keys
attached migration and pending=0
explicit runtime grants
runtime negative DDL/admin permissions
TblPosLocal/TblTenantPosDevice separation
single routing-table dependency
active/subscription filtering
source exclusion
canonical Provider token/header ownership
Provider application-identity authentication
periodic worker -> Provider -> grouped endpoint
API atomic EventLog/Delivery commit
SignalR after commit
all-or-none local completion
```

Rerun:

```text
prompt110 single-sync-path architecture guards
prompt114 role-contract tests
prompt115 no-fallback/no-dual-routing guards
```

Expected:

```text
all pass
0 skipped
parallel production sync paths=0
parallel routing paths=0
```

Build:

```text
WPF
ApiServer
PlatformAppV0 when the active POS registration/pairing writer changed
```

Build/test success does not override failed physical writer, routing, Provider, or happy-path proof.

## End state

PASS requires:

```text
one canonical TblTenantPosDevice writer lifecycle
one attached ExternalDbContext migration and model mapping
pending migrations=0
least-privilege role contract preserved
TblPosLocal remains the separate logical POS table
one generated Development POS1 source and POS2 destination registered canonically
one concrete subscribed POS2 destination selected
POS1 source excluded
canonical Provider owns all authentication/header behavior
one physical POS1 -> API grouped-sync happy path committed
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
E:\Project2026\RecoveryReports\MainDevPosDeviceSchemaWriterAndSyncHappyPathV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ARTIFACT_VERIFICATION.md
PROMPT115_BLOCKER_INTAKE.md
POS_DEVICE_WRITER_LINEAGE.md
POS_DEVICE_LIFECYCLE_DECISION.md
TBLPOSLOCAL_TBLTENANTPOSDEVICE_SEPARATION.md
POS_DEVICE_SCHEMA_CONTRACT.md
WRITER_BEFORE.md
WRITER_AFTER.md
WRITER_IDEMPOTENCY_PROOF.md
ENTITY_MAPPING.md
MIGRATION_SOURCE.md
MIGRATION_APPLY_PROOF.md
MIGRATION_HISTORY.md
RUNTIME_GRANT_PROOF.md
RUNTIME_NEGATIVE_PERMISSION_PROOF.md
DEVELOPMENT_POS_REGISTRATION.md
DESTINATION_ROUTING_PROOF.md
CANONICAL_PROVIDER_CALL_CHAIN.md
PROVIDER_AUTH_HEADER_OWNERSHIP.md
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
ROLE_CONTRACT_TEST_OUTPUT.txt
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
report/report116.md
```

Include:

```text
Verdict
Prompt115 artifact SHA verified yes/no
Canonical writer owner/service/method
Writer lifecycle trigger
Writer contract proven before migration yes/no
TblPosLocal role
TblTenantPosDevice role
Canonical entity/mapping added yes/no
Attached migration identifier
Migration applied exact-once yes/no
API pending migrations count
Runtime explicit grants applied yes/no
Runtime role CREATEDB after
Runtime negative DDL/admin proof yes/no
Canonical writer first registration proof yes/no
Canonical writer idempotent replay proof yes/no
Replacement/revocation proof yes/no/not-required with reason
Development POS1 source registered through writer yes/no
Development POS2 destination registered through writer yes/no
Subscriptions created through canonical boundary yes/no
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
Parallel routing path count
Happy-path marker present yes/no
Manual POS1 test ready true/false
Focused test totals
Architecture guard totals
Role-contract regression totals
WPF build totals
API build totals
PlatformAppV0 build totals/not changed
WPF/API DB reset performed yes/no
Manual device/outbox/event/delivery insertion used yes/no
Category Weight changed yes/no
Booking Weight changed yes/no
Production/customer/reference DB mutated yes/no
Private artifact yes/no
Aggregate SHA-256
```

Do not expose passwords, tokens, complete connection strings, passfile contents, raw Development identities, or business payload values.

## Verdicts

PASS:

```text
OBM_MAIN_DEV_POS_DEVICE_SCHEMA_WRITER_AND_CANONICAL_SYNC_HAPPY_PATH_READY_FOR_MANUAL_POS1_TEST
```

Narrow blockers only:

```text
BLOCKED_MAIN_DEV_POS_DEVICE_WRITER_CONTRACT
BLOCKED_MAIN_DEV_POS_DEVICE_SCHEMA_CONTRACT
BLOCKED_MAIN_DEV_POS_DEVICE_MIGRATION
BLOCKED_MAIN_DEV_POS_DEVICE_GRANTS
BLOCKED_MAIN_DEV_POS_DEVICE_REGISTRATION
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

Every blocked result must include:

```text
exact failed class/service/method/command
writer/schema/migration state
sanitized exception chain and SQLSTATE when available
which DB writes/migrations/grants occurred
MANUAL_POS1_TEST_READY=false
```

Do not return a generic schema, routing, writer, Provider, or sync blocker.
