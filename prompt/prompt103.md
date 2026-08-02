# Prompt 103 — Run transaction-group E2E on the canonical Visual Studio development databases

## Operator decision — authoritative

Prompt102 returned:

```text
BLOCKED_SYNC_E2E_DISPOSABLE_DB_PROVISIONING
```

The operator has now made an explicit decision:

```text
Stop requiring newly provisioned disposable PostgreSQL databases for this transaction-group E2E.
Use the canonical Visual Studio WPF and ApiServer Development lanes and the Development connection settings already configured there.
Existing Development data is disposable and does not need to be preserved.
The Development databases may be reset/recreated when required to align schema and migration history.
```

This decision supersedes the disposable-database requirement in prompts101 and 102 for this development-stage verification.

Do not return another generic disposable-environment blocker.

## Safety boundary

This authorization applies only to the canonical local Development databases resolved from the active Visual Studio Development configurations.

Never mutate:

```text
production databases
customer/salon databases
Royal Nail SPA production data
source/reference database enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
any database not explicitly resolved as the active local Development target
```

Before any destructive action, prove locally:

```text
Environment = Development
host is local/approved development PostgreSQL
resolved WPF database name
resolved API database name
both names pass protected-name refusal rules
neither connection targets production/customer infrastructure
```

Do not expose passwords, connection strings, passfile contents, JWTs, or raw private identifiers.

## Canonical lanes

Use the actual main Visual Studio source/runtime lanes:

```text
WPF:
E:\Project2026\4POS\NailSalonNet8

ApiServer:
E:\Project2026\1ApiServer\ApiServer01
```

Use the actual Development connection configuration already used when these projects run from Visual Studio.

Do not create a parallel fake API, duplicate uploader, duplicate sender, alternate DbContext, or test-only transport.

The E2E must execute the production code implemented by prompts099 and 100.

## Current accepted implementation

The following gates already passed:

```text
Prompt097 — WPF PostgreSQL migration baseline and grouped TblLocalOutbox schema
Prompt098 — API PostgreSQL event/delivery/group-ACK schema
Prompt099 — atomic Price Rule local three-table Save and grouped outbox
```

Prompt100 implemented and built:

```text
whole-group WPF selection/validation
atomic group claim/lease
one grouped HTTP request
all-or-none local completion/retry
API whole-group validation/auth
complete-group idempotency/conflict handling
atomic TblEventLog + TblEventDelivery persistence
source-client exclusion
SignalR after commit
SignalR post-commit failure handling
crash/replay recovery
```

Prompt101/102 did not disprove this code. They failed to execute the E2E because disposable provisioning never started.

## Strict scope

Use the canonical Development databases and execute:

```text
Visual Studio WPF Development DB
-> production Price Rule sender
-> production TblLocalOutbox group
-> production WPF uploader
-> real loopback ApiServer
-> production authenticated group endpoint
-> Visual Studio API Development DB
-> atomic event/delivery commit
-> post-commit SignalR notification attempt
-> all-or-none WPF outbox completion
```

Then execute all 15 failure/recovery cases from prompt101.

Do not implement in this task:

```text
POS2 pull/apply
TblTurnPolicy inbound receiver
TblTurnAmountRule inbound receiver
group ACK runtime endpoint
cloud deployment
checkout/payment changes
BookingConsole runtime changes
production deployment
```

## Phase 1 — Resolve and classify the active Development databases

Read the actual Visual Studio Development configuration and runtime connection resolution code.

Return safe proof:

```text
WPF provider = Npgsql
WPF environment = Development
WPF database name = safe name only

API provider = Npgsql
API environment = Development
API database name = safe name only
```

Use the existing protected credential mechanism and existing connection settings.

Do not request the operator to paste passwords into chat.

If either database is not clearly a local Development target, stop with a narrow blocker identifying only that lane.

## Phase 2 — Stop active processes before reset/migration

Before changing schema or data, stop only development processes associated with these lanes:

```text
WPF debug instance
ApiServer debug instance
outbox publisher worker
other process holding the selected Development DBs when proven related
```

Do not stop PostgreSQL globally unless required and explicitly safe.

Do not kill unrelated applications.

## Phase 3 — Choose the smallest safe reset strategy

Because old Development data is disposable, choose exactly one strategy after inspecting migration history.

### Strategy R1 — Full Development DB recreation — preferred

Use when migration history/schema is stale, mixed, or cannot reliably recreate only the sync tables.

```text
capture safe pre-reset schema/history counts
terminate sessions only for the selected Development DB
DROP selected Development DB
CREATE empty UTF8 selected Development DB
apply accepted migration chain from zero
pending migrations = 0
```

Apply separately to WPF and API Development databases.

This is preferred because it keeps physical schema and `__EFMigrationsHistory` consistent.

### Strategy R2 — Sync-table-only reset

Use only when the existing migration chain has a proven attached migration that can recreate the sync subsystem without corrupting migration history.

For API, dependency order is:

```text
drop/recreate TblEventDeliveryGroupAck
drop/recreate TblEventDelivery
drop/recreate TblEventLog
```

`TblEventDelivery.EventSequence` depends on `TblEventLog.EventSequence`.

For WPF, inspect actual FK/dependency order for:

```text
TblTurnAmountRule
TblTurnPolicy
TblLocalOutbox
```

Do not use `DROP ... CASCADE` blindly.

Do not manually drop tables while leaving migration history claiming they still exist.

If R2 cannot guarantee migration-history consistency, use R1.

Record the selected strategy and proof.

## Phase 4 — Apply migrations using the main code lanes

Use the migration mechanisms accepted in prompts097/098 and the same source currently opened in Visual Studio.

Required proof for both WPF and API Development DBs:

```text
migration command/service completed
provider = Npgsql
pending migrations = 0
__EFMigrationsHistory matches source
no manual table creation outside attached migration mechanism
```

Verify physical schema:

### WPF

```text
TblTurnPolicy exists
TblTurnAmountRule exists
TblLocalOutbox exists
ExpectedEventCount exists
EntityGuid NOT NULL
claim/retry/sent fields exist
transaction-group constraints/indexes exist
```

### API

```text
TblEventLog exists
TblEventDelivery exists
TblEventDeliveryGroupAck exists
transaction-group columns exist
constraints/indexes/FK exist
```

## Phase 5 — Seed minimal Development E2E identity/routing

Seed only what the production authentication and routing contracts require:

```text
one Development tenant
one active POS1 source-client identity
one active POS2 destination-client identity
entity routing/subscription for exactly:
  TblTurnPolicy
  TblTurnAmountRule
```

Use generated Development identities, not production/customer identifiers.

Do not seed unrelated business history.

Prove:

```text
POS1 can authenticate to the grouped endpoint
POS2 is selected as destination
POS1 is excluded as destination
no unrelated destination exists
```

## Phase 6 — Start the real ApiServer from the main Visual Studio lane

Start the actual ApiServer Development project on loopback only using its existing Development configuration and the reset/migrated API Development DB.

Required proof:

```text
readiness/health succeeds
loopback binding only
resolved DB is the selected Development API DB
grouped endpoint is reachable
production controller/service code loaded
```

Use the actual production auth policy.

Allowed test identity mechanisms:

```text
existing local POS JWT issuer
existing UiTest/test signer
existing integration authentication provider that executes production authorization policy
```

Not allowed:

```text
AllowAnonymous
removing authorization
bypassing tenant/source validation
hardcoded reusable token in source/report
```

## Phase 7 — Create the happy-path group through the production WPF sender

Run the actual Price Rule Save/local sender implemented in prompt099 against the WPF Development DB.

Do not manually insert happy-path outbox rows.

Required local result:

```text
one Draft TblTurnPolicy
one Price Rule change
one complete TblLocalOutbox group
ExpectedEventCount = 2
Sequence 1 = TblTurnPolicy I
Sequence 2 = TblTurnAmountRule I
same TransactionGuid/TenantGuid/SourceClientId
Sent = Pending before upload
```

## Phase 8 — Execute production uploader → API E2E

Invoke the actual WPF uploader implemented in prompt100.

Prove:

### WPF claim/request

```text
one complete group selected
both rows claimed atomically
one HTTP request
ordered events 1,2
```

### API durability

```text
authentication passed
request validated
TblEventLog rows = 2
TblEventDelivery rows for POS2 = 2
TblEventDelivery rows for POS1 = 0
same TransactionGuid
ExpectedEventCount = 2
sequences 1,2
policy event first
one API database commit boundary
```

### SignalR

```text
notification attempted only after durable API commit
notification contains availability metadata, not business payload
```

### WPF completion

```text
both local outbox rows marked Sent together
SentAt populated
ServerEventSequence mapped
claim fields cleared
no mixed status
```

Expected marker:

```text
SYNC_GROUP_MAIN_DEV_E2E_COMMITTED
```

## Phase 9 — Execute the full 15-case matrix

Run all prompt101 cases on the reset Development test lane:

```text
1 incomplete local group
2 sequence gap/duplicate
3 invalid parent-first ordering
4 concurrent uploader workers
5 lease recovery
6 API unavailable/timeout
7 auth/identity mismatch
8 API validation rejection
9 exact idempotent replay
10 conflicting replay
11 API persistence failure before commit
12 SignalR failure after commit
13 crash after API commit before local mark-sent
14 invalid/partial API response
15 source exclusion/destination routing
```

Use isolated transaction groups/test rows per case.

Test-only failure hooks must default off and be impossible to activate in normal production configuration without an explicit Development/E2E setting.

No soft skip.

If a case fails:

```text
identify first exact failing boundary
make the smallest correction inside prompt100 scope
rerun failed case
rerun all 15 cases
```

## Phase 10 — Focused tests and build

Run actual focused tests for:

```text
group validation
claim concurrency
lease recovery
all-or-none retry
response validation
auth/identity
exact replay
conflicting replay
pre-commit rollback
SignalR ordering/failure
crash/replay
source exclusion/routing
```

Report actual totals.

Build WPF and API after any correction.

## Phase 11 — End state

Do not drop the canonical Development databases after this task; they become the current clean Visual Studio Development test lane.

Leave them in a documented state:

```text
migrations current
pending migrations = 0
E2E/test data clearly Development-only
no running failure-injection setting
ApiServer/WPF process state documented
```

Do not reset or mutate any production/customer database.

## Required private evidence artifact

Create a new versioned artifact:

```text
E:\Project2026\RecoveryReports\MainVisualStudioDevSyncGroupE2EV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
DEV_DB_RESOLUTION.md
RESET_STRATEGY.md
PROCESS_STOP_START.md
MIGRATION_PROOF.md
SCHEMA_PROOF.md
IDENTITY_ROUTING_SEED.md
AUTH_PROOF.md
PRODUCTION_SENDER_PROOF.md
PRODUCTION_UPLOADER_PROOF.md
HAPPY_PATH_E2E.md
FAILURE_MATRIX.md
FOCUSED_TEST_OUTPUT.txt
FINAL_DEV_DB_STATE.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Do not expose secrets, full connection strings, tokens, raw private identifiers, or business payload values.

## Public report

Create and push only:

```text
report/report103.md
```

It must include:

```text
Verdict
Main Visual Studio WPF Development DB resolved yes/no
Main Visual Studio API Development DB resolved yes/no
Reset strategy R1/R2
WPF migrations/pending count
API migrations/pending count
Loopback production API yes/no
Production auth policy exercised yes/no
Production sender used yes/no
Production uploader used yes/no
Happy-path E2E yes/no
API atomic event/delivery commit yes/no
Source exclusion yes/no
SignalR after commit yes/no
All-or-none local completion yes/no
Exact replay yes/no
Conflicting replay yes/no
Crash/replay yes/no
Failure matrix totals
Focused test totals
WPF/API build totals
Production/customer DB mutation yes/no
Private artifact yes/no
Aggregate SHA-256
```

## Source/repository rules

OBM source remains local/private.

Do not push OBM source changes to the coordination repository.

Only push:

```text
report/report103.md
```

Preserve unrelated dirty local source changes.

Do not reset/clean/checkout unrelated source.

## Final verdicts

PASS only when the main Visual Studio Development lane physically proves the complete POS1-to-API boundary and all 15 cases:

```text
OBM_MAIN_DEV_TRANSACTION_GROUP_E2E_READY_FOR_POS2_PULL_APPLY_ACK
```

Use a narrow blocker otherwise, naming the actual boundary, for example:

```text
BLOCKED_MAIN_DEV_WPF_DB_RESOLUTION
BLOCKED_MAIN_DEV_API_DB_RESOLUTION
BLOCKED_MAIN_DEV_DATABASE_RESET
BLOCKED_MAIN_DEV_MIGRATION_APPLY
BLOCKED_MAIN_DEV_AUTH
BLOCKED_MAIN_DEV_ROUTING
BLOCKED_MAIN_DEV_GROUP_UPLOAD
BLOCKED_MAIN_DEV_API_COMMIT
BLOCKED_MAIN_DEV_FAILURE_MATRIX
```

Do not return another disposable-provisioning blocker.
