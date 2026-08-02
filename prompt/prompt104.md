# Prompt 104 — Execute the approved R1 Development DB reset and prove the POS1-to-API happy path

## Authoritative operator approval

Prompt103 returned:

```text
BLOCKED_MAIN_DEV_DATABASE_RESET
```

It also proved:

```text
Main Visual Studio WPF Development DB resolved: yes
Main Visual Studio API Development DB resolved: yes
Reset strategy R1 selected
Production/customer DB mutation: no
```

The operator has repeatedly and explicitly declared that the current local Development data is disposable and may be recreated.

This prompt is the explicit authorization to EXECUTE the already selected R1 reset on the two resolved canonical local Development databases.

Do not stop after selecting or planning R1.
Do not return another generic reset blocker unless a safety guard actually fails.

## Scope

Execute only:

```text
1. Stop the canonical WPF/API Development processes.
2. Revalidate the two already resolved Development database targets.
3. Drop and recreate those two local Development databases.
4. Apply the accepted WPF/API PostgreSQL migration chains from zero.
5. Seed the minimum Development identity/routing state.
6. Start the real loopback ApiServer Development lane.
7. Run one production Price Rule sender -> grouped uploader -> API happy-path E2E.
8. Leave both Development databases in a clean documented state.
```

Do not run the full 15-case failure matrix in this task. That will be prompt105 after the happy path is physically proven.

Do not implement POS2 pull/apply/ACK in this task.

## Non-negotiable safety guard

Before destructive work, prove again from the actual active Visual Studio Development configuration:

```text
WPF environment = Development
WPF provider = Npgsql
WPF resolved database name equals the prompt103 resolved WPF Development database

API environment = Development
API provider = Npgsql
API resolved database name equals the prompt103 resolved API Development database

both database hosts are local/approved Development PostgreSQL
both names pass protected-name refusal
neither target is a production/customer/reference database
```

Hard reject these and any equivalent protected names:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
recovery_api_day16_pg
Royal/customer/production databases
```

Do not expose connection strings, passwords, passfile contents, tokens, or raw private identifiers.

If a safety guard fails, return one narrow blocker naming the exact lane and mismatched safe database name. Do not mutate anything.

## Phase 1 — Stop only relevant processes

Stop only proven Development processes for:

```text
WPF debug instance
ApiServer debug instance
outbox publisher worker hosted by those processes
```

Do not stop PostgreSQL globally.
Do not terminate unrelated apps.

Capture safe process-state evidence.

## Phase 2 — Execute R1 for WPF Development DB

Using the existing protected PostgreSQL admin mechanism already available on the machine:

```text
terminate sessions only for the selected WPF Development DB
drop the selected WPF Development DB
create it again as empty UTF8 PostgreSQL
apply the accepted prompt097 WPF migration chain from zero
verify __EFMigrationsHistory
verify pending migrations = 0
```

Do not use EnsureCreated.
Do not manually create business tables outside the attached migration chain.

Physical schema proof must include:

```text
TblTurnPolicy
TblTurnAmountRule
TblLocalOutbox
ExpectedEventCount
EntityGuid NOT NULL
ClaimedBy
ClaimExpiresAt
NextAttemptAt
SentAt
all transaction-group constraints/indexes
```

## Phase 3 — Execute R1 for API Development DB

Using the same approved protected mechanism:

```text
terminate sessions only for the selected API Development DB
drop the selected API Development DB
create it again as empty UTF8 PostgreSQL
apply the accepted prompt098 API migration chain from zero
verify __EFMigrationsHistory
verify pending migrations = 0
```

Physical schema proof must include:

```text
TblEventLog
TblEventDelivery
TblEventDeliveryGroupAck
all transaction-group columns
all checks/unique indexes/retry indexes
TblEventDelivery -> TblEventLog FK
```

## Phase 4 — Minimal Development identity/routing seed

Seed only the minimum required by the actual production auth/routing contracts:

```text
one Development tenant
one active POS1 source-client identity
one active POS2 destination-client identity
entity registration/subscriptions for:
  TblTurnPolicy
  TblTurnAmountRule
```

Use newly generated Development-only identities.
Do not reuse production/customer identifiers.
Do not seed unrelated business history.

Prove safely:

```text
POS1 source authentication identity exists
POS2 destination identity exists
POS1 excluded from destination routing
no unrelated destination subscription exists
```

## Phase 5 — Start the real ApiServer Development lane

Start the actual ApiServer01 Development project from the canonical main source lane against the reset API Development DB.

Requirements:

```text
loopback only
readiness/health = success
resolved database = selected API Development DB
production grouped controller/service loaded
normal production auth policy active
```

Use the existing local POS JWT/test signer/UiTest auth mechanism that executes the production authorization policy.

Do not use AllowAnonymous.
Do not remove claim validation.
Do not publish token values.

## Phase 6 — Run the production local sender

Run the actual prompt099 Price Rule local sender against the reset WPF Development DB.

Do not manually insert the happy-path outbox rows.

Required result:

```text
one Draft TblTurnPolicy created
one TblTurnAmountRule created
one complete TblLocalOutbox transaction group created
ExpectedEventCount = 2
Sequence 1 = TblTurnPolicy I
Sequence 2 = TblTurnAmountRule I
same TenantGuid/SourceClientId/TransactionGuid
unique EventGuid values
non-empty EntityGuid values
Sent = Pending before upload
```

## Phase 7 — Run the production uploader happy path

Invoke the actual prompt100 WPF grouped uploader.

Prove:

### Local claim and HTTP

```text
exactly one complete group selected
both rows claimed atomically
one grouped HTTP request
sequences ordered 1,2
production auth/session envelope used
```

### API durable commit

```text
auth passed
request validated
TblEventLog rows = 2
same TransactionGuid
ExpectedEventCount = 2
SequenceNumber = 1,2
sequence 1 policy

TblEventDelivery rows for POS2 = 2
TblEventDelivery rows for POS1 = 0
one API database transaction/commit boundary
```

### SignalR ordering

```text
SignalR notification attempted only after durable API commit
notification contains availability metadata only
```

### Local completion

```text
both TblLocalOutbox rows marked Sent together
SentAt populated
ServerEventSequence mapped
claim fields cleared
no mixed status
```

Required marker:

```text
SYNC_GROUP_MAIN_DEV_HAPPY_PATH_COMMITTED
```

## Phase 8 — Exact replay smoke

Before closing, run one exact replay/crash-safe smoke using the same immutable request/group identity or the established replay harness.

Prove:

```text
API returns accepted idempotent replay
no duplicate TblEventLog rows
no duplicate TblEventDelivery rows
same event sequence mapping returned
```

Do not run the remaining failure matrix cases in this task.

## Phase 9 — Builds and end state

Build WPF and API after any narrow correction.

Leave the Development DBs present and documented:

```text
migrations current
pending migrations = 0
happy-path E2E data marked Development-only
failure injection disabled
API process state documented
WPF process state documented
```

Do not drop the two Development DBs after success.

## Narrow correction policy

If reset, migration, seed, auth, sender, uploader, or happy path fails:

```text
identify the first exact boundary
capture safe exception/SQL state/method chain
make the smallest correction inside prompts097-100 scope
rerun from the failed boundary
rerun the happy path
```

Do not expand into POS2 pull/apply/ACK.
Do not return a generic reset blocker after a specific failure is known.

## Private artifact

Create a new versioned artifact:

```text
E:\Project2026\RecoveryReports\MainDevResetAndSyncHappyPathV001
```

Required files:

```text
PRIVATE_HANDOFF.md
SAFETY_GUARD.md
PROCESS_STOP_START.md
WPF_DB_RESET.md
API_DB_RESET.md
WPF_MIGRATION_PROOF.md
API_MIGRATION_PROOF.md
SCHEMA_PROOF.md
IDENTITY_ROUTING_SEED.md
AUTH_PROOF.md
PRODUCTION_SENDER_PROOF.md
PRODUCTION_UPLOADER_PROOF.md
HAPPY_PATH_E2E.md
EXACT_REPLAY_SMOKE.md
FINAL_DEV_DB_STATE.md
TEST_OUTPUT.txt
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Do not expose secrets, connection strings, tokens, raw private identifiers, or business payload values.

## Public report

Create and push only:

```text
report/report104.md
```

Include:

```text
Verdict
WPF Development DB safety guard yes/no
API Development DB safety guard yes/no
R1 WPF reset executed yes/no
R1 API reset executed yes/no
WPF pending migrations
API pending migrations
Minimal identity/routing seed yes/no
Loopback production API yes/no
Production auth policy exercised yes/no
Production sender used yes/no
Production uploader used yes/no
Happy-path E2E yes/no
API atomic event/delivery commit yes/no
Source exclusion yes/no
SignalR after commit yes/no
All-or-none local completion yes/no
Exact replay smoke yes/no
WPF/API build totals
Production/customer DB mutation yes/no
Private artifact yes/no
Aggregate SHA-256
```

## Repository rules

OBM source remains local/private.
Only push `report/report104.md` to the coordination repository.
Preserve unrelated dirty local changes.
Do not reset/clean/checkout unrelated source.

## Final verdicts

PASS:

```text
OBM_MAIN_DEV_RESET_AND_SYNC_HAPPY_PATH_READY_FOR_FAILURE_MATRIX
```

Use a narrow blocker otherwise:

```text
BLOCKED_MAIN_DEV_WPF_RESET
BLOCKED_MAIN_DEV_API_RESET
BLOCKED_MAIN_DEV_WPF_MIGRATION
BLOCKED_MAIN_DEV_API_MIGRATION
BLOCKED_MAIN_DEV_IDENTITY_ROUTING_SEED
BLOCKED_MAIN_DEV_AUTH
BLOCKED_MAIN_DEV_LOCAL_SENDER
BLOCKED_MAIN_DEV_GROUP_UPLOAD
BLOCKED_MAIN_DEV_API_COMMIT
BLOCKED_MAIN_DEV_REPLAY
```

Do not return `BLOCKED_MAIN_DEV_DATABASE_RESET` without naming the exact failed operation.
