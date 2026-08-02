# Prompt 102 — Close the disposable POS1-to-API sync-group environment and execute the full E2E matrix

## Starting verdict

Prompt101 returned:

```text
BLOCKED_SYNC_GROUP_DISPOSABLE_ENVIRONMENT
```

The public report proves that no disposable environment was actually established:

```text
Disposable POS1 DB created/applied: no
Disposable API DB created/applied: no
Loopback authenticated API: no
Production local sender used: no
Whole-group HTTP E2E: no
Failure matrix: 15 not executed
Focused tests: not executed
Current operator DBs mutated: no
```

The prompt100 implementation still builds and remains the implementation under test.

This task must close the environment boundary first, then run the complete prompt101 E2E and failure matrix. Do not redesign the sync architecture.

## Strict scope

Build and execute a repeatable private disposable test environment for:

```text
production Price Rule local sender
-> disposable POS1 PostgreSQL DB
-> production whole-group WPF uploader
-> real loopback HTTP
-> authenticated production ApiServer grouped endpoint
-> disposable API PostgreSQL DB
-> atomic TblEventLog + TblEventDelivery persistence
-> post-commit SignalR notification attempt
-> all-or-none POS1 local completion
```

Then execute all 15 prompt101 cases.

Do not implement in this task:

```text
API group pull
POS2 pull/apply
TblTurnPolicy inbound receiver
TblTurnAmountRule inbound receiver
group ACK endpoint/runtime
cloud deployment
current operator DB reset
checkout/payment changes
BookingConsole runtime changes
```

A POS2 identity and subscription rows are required only as the destination routing target in the API DB. No POS2 process or local DB apply is required.

## Source of truth

Read completely before any action:

```text
prompt/prompt097.md
prompt/prompt098.md
prompt/prompt099.md
prompt/prompt100.md
prompt/prompt101.md
report/report097.md
report/report098.md
report/report099.md
report/report100.md
report/report101.md
```

Read every file in the private prompt101 artifact identified by aggregate hash:

```text
179e605dc759a98e41fa9ee44994c641994438b592e469275ddd7aaba5554de4
```

Also read the complete prompt097–100 private artifacts and the current actual local source changes.

Record before editing or provisioning:

```text
DOCS_READ_BEFORE_ENVIRONMENT_GATE=PASS
TASK_SCOPE=DISPOSABLE_POS1_TO_API_E2E
PRODUCT_IMPLEMENTATION_UNDER_TEST=PROMPT100_CURRENT_LOCAL_SOURCE
POS2_APPLY_ACK=DEFERRED
LOOPBACK_ONLY=TRUE
CURRENT_OPERATOR_DB_MUTATION=FORBIDDEN
SECRETS_IN_ARTIFACTS=FORBIDDEN
```

## Phase 1 — Prove the exact prompt101 environment blocker

Do not merely repeat `BLOCKED_SYNC_GROUP_DISPOSABLE_ENVIRONMENT`.

From the private artifact and actual machine state, classify every concrete blocker using narrow codes such as:

```text
SYNC_E2E_POSTGRESQL_ADMIN_CREDENTIAL_MISSING
SYNC_E2E_WPF_MIGRATION_CONNECTION_MISSING
SYNC_E2E_API_MIGRATION_CONNECTION_MISSING
SYNC_E2E_DISPOSABLE_DB_CREATION_SCRIPT_MISSING
SYNC_E2E_LOOPBACK_API_STARTUP_PROFILE_MISSING
SYNC_E2E_API_AUTH_TEST_IDENTITY_MISSING
SYNC_E2E_TOKEN_MINTING_HARNESS_MISSING
SYNC_E2E_ROUTING_SEED_HARNESS_MISSING
SYNC_E2E_PRODUCTION_SENDER_HARNESS_MISSING
SYNC_E2E_PRODUCTION_UPLOADER_HARNESS_MISSING
SYNC_E2E_FAILURE_INJECTION_HARNESS_MISSING
SYNC_E2E_TEST_RUNNER_MISSING
```

Return locally:

```text
exact failed command or missing prerequisite
whether PostgreSQL was reached
whether API process was started
whether authentication was attempted
whether any disposable DB was created
smallest correction
```

Do not claim a credential is missing until all approved protected sources have been checked safely.

## Phase 2 — Credential and secret resolution

Use only existing approved protected mechanisms:

```text
PGPASSFILE
Windows user environment variables
.NET user-secrets
existing protected local credential files
existing canonical startup scripts
```

Allowed dedicated environment variable names include:

```text
OBM_WPF_MIGRATION_CONNECTION
OBM_API_MIGRATION_CONNECTION
```

or the actual canonical equivalents established by prompts097/098.

Requirements:

```text
no password literal in source/scripts
no connection string committed to GitHub
no full connection string in evidence
no passfile contents in evidence
no JWT/token in evidence
no fallback to the current operator DB
```

If a required credential truly does not exist, stop before source mutation and return one exact operator action with the variable/passfile name and safe command shape, without asking the operator to paste the secret into chat.

Do not return a generic environment blocker.

## Phase 3 — Create a reusable private E2E runner

Create the smallest repeatable test-only runner/harness, using existing project conventions.

Preferred form:

```text
PowerShell orchestration script
+ focused .NET integration/E2E test project or existing test project
```

Suggested versioned private location:

```text
E:\Project2026\RecoveryReports\TransactionGroupUploaderApiE2EEnvironmentV001
```

The orchestration must perform:

```text
preflight
protected DB-name rejection
unique versioned disposable DB name generation
POS1 DB create
API DB create
WPF migrations apply from zero
API migrations apply from zero
pending migrations verification
minimal identity/routing seed
loopback API startup
readiness wait with timeout
short-lived test authentication acquisition
production local sender execution
production uploader execution
assertions
failure injection cases
API process shutdown
DB cleanup
artifact generation
```

The runner must be idempotent and must clean up only databases/processes it created.

Do not use Docker unless the existing approved environment already depends on it. Local Windows PostgreSQL and loopback ApiServer are canonical for this task.

## Phase 4 — Disposable database provisioning

Create two new, separately named, versioned PostgreSQL databases:

```text
POS1 disposable WPF DB
API disposable DB
```

Never use or mutate:

```text
obm_pos_dev_v0_pg
obm_pos_v1_local_pos1_pg
enailsalon_phasee1_pos1_pg
recovery_api_day16_pg
obm_api_dev_v0_pg
any protected/current/production DB
```

For both DBs prove:

```text
did not exist before test
created UTF8
migration chain applied from zero
pending migrations = 0
__EFMigrationsHistory correct
provider = Npgsql
```

Do not use `EnsureCreated` as a substitute for migrations.

## Phase 5 — Minimal API identity and routing seed

Seed only what the production authentication and routing contracts require:

```text
one tenant
one active POS1 source-client identity
one active POS2 destination-client identity
exact entity registration/subscriptions for:
  TblTurnPolicy
  TblTurnAmountRule
```

Use the actual production schema and identity conventions.

Do not seed unrelated business data.

Do not hardcode private production identifiers.

All disposable identities must be generated for the test and kept out of the public report.

Prove:

```text
POS1 is authorized as the request source
POS2 is a valid destination
source exclusion can be asserted
no unrelated destination is registered
```

## Phase 6 — Loopback production ApiServer

Start the actual ApiServer process against only the disposable API DB.

Requirements:

```text
127.0.0.1/localhost only
unique test port
Development/UiTest or dedicated E2E environment
no production/public binding
real production grouped controller/service code
real ExternalDbContext
real SignalR service boundary, with injectable observation/failure hook when already supported
```

Do not replace the production grouped endpoint with a fake HTTP server for the happy path.

Prove:

```text
process ID captured safely
readiness endpoint returns success
resolved DB name is disposable API DB
normal current DB remains untouched
grouped endpoint route is reachable
```

## Phase 7 — Authentication harness without bypass

Use the actual authentication/authorization policy protecting the grouped endpoint.

Allowed approaches:

```text
existing integration test token signer
existing UiTest provider/test identity
existing local POS JWT issuance helper
existing test server authentication configuration that still executes the production authorization policy
```

Not allowed:

```text
mark endpoint AllowAnonymous
remove authorization
hardcode a reusable token
skip tenant/source-client claim validation
send invented headers that production code does not accept
```

The token/claims must represent the disposable POS1 source identity and tenant.

Evidence may contain only safe markers:

```text
authenticated = true
required claim names present = true
tenant/source identity matched = true
token value redacted
```

## Phase 8 — Production sender and uploader harness

Invoke the actual prompt099 local sender to create the group. Do not manually insert the happy-path outbox rows.

Invoke the actual prompt100 uploader service/method. Do not duplicate uploader logic in test code.

A test adapter may resolve production services and provide the disposable connection/API base address, but must not reimplement business behavior.

Initial group contract:

```text
ExpectedEventCount = 2
Sequence 1 = TblTurnPolicy I
Sequence 2 = TblTurnAmountRule I
one TransactionGuid
one TenantGuid
one SourceClientId
unique EventGuid values
non-empty EntityGuid values
Sent = Pending before upload
```

## Phase 9 — Happy-path real E2E

Run the real boundary once and prove:

### POS1

```text
production sender committed Policy + Rule + 2 outbox rows
one complete group claimed
both rows share claim owner/expiry
exactly one grouped HTTP request
```

### API

```text
authentication passed
request validated
2 TblEventLog rows committed
2 TblEventDelivery rows committed for POS2
0 delivery rows committed for POS1
same TransactionGuid
ExpectedEventCount = 2
SequenceNumber = 1,2
policy event first
one API transaction commit
```

### SignalR

```text
notification attempted only after API durable commit
notification carries availability metadata only, not business payload
```

### POS1 completion

```text
both rows marked Sent together
SentAt populated
ServerEventSequence populated correctly
claim fields cleared
no mixed status
```

Expected marker:

```text
SYNC_GROUP_E2E_COMMITTED
```

## Phase 10 — Execute all 15 prompt101 cases

Run every case from prompt101 with no soft skip:

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

Use deterministic test/failure-injection hooks that are test-only and cannot activate in normal production configuration.

Every hook must default off and require an explicit E2E-only setting.

If any case fails:

```text
capture first exact boundary
make only the smallest correction within prompt100 scope
rerun the failed case
rerun all 15 cases
```

Do not proceed with unexecuted cases.

## Phase 11 — Focused automated test execution

Run the focused tests created or identified by prompts100/101.

At minimum cover:

```text
group validation
atomic claim concurrency
lease recovery
all-or-none retry
response validation
auth/identity validation
exact replay
conflicting replay
API pre-commit rollback
SignalR ordering
SignalR post-commit failure
crash/replay
source exclusion/routing
```

Report real totals.

Build success is not E2E proof.

## Phase 12 — Cleanup and preservation

After successful evidence capture:

```text
stop only the loopback API process created by the runner
drop only the disposable POS1/API databases created by the runner
clear only temporary environment variables/files created by the runner
preserve versioned evidence
```

If cleanup fails, report it explicitly but do not touch unrelated processes or DBs.

Current operator databases must remain untouched.

## Required private evidence artifact

Create a new versioned private artifact. Never overwrite prior artifacts.

Suggested folder:

```text
E:\Project2026\RecoveryReports\TransactionGroupUploaderApiE2EEnvironmentV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
PROMPT101_BLOCKER_ROOT_CAUSE.md
PREFLIGHT.md
CREDENTIAL_RESOLUTION.md
RUNNER.md
DISPOSABLE_DB_SETUP.md
MIGRATION_PROOF.md
IDENTITY_ROUTING_SEED.md
API_STARTUP.md
AUTH_PROOF.md
PRODUCTION_SENDER_PROOF.md
PRODUCTION_UPLOADER_PROOF.md
HAPPY_PATH_E2E.md
FAILURE_MATRIX.md
FOCUSED_TEST_OUTPUT.txt
PROCESS_CLEANUP.md
DATABASE_CLEANUP.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Include complete actual runner/test code and any narrow product correction code in the private handoff.

Do not expose credentials, connection strings, tokens, raw identifiers, or private payloads.

## Public report

Create and push only:

```text
report/report102.md
```

The redacted public report must include:

```text
Verdict
Exact environment blocker resolved yes/no
Disposable POS1 DB created/applied yes/no
Disposable API DB created/applied yes/no
Pending migrations for both
Loopback production API yes/no
Production auth policy exercised yes/no
Production local sender used yes/no
Production uploader used yes/no
Whole-group HTTP E2E yes/no
API atomic event/delivery commit yes/no
Source exclusion yes/no
SignalR after commit yes/no
All-or-none POS1 completion yes/no
Exact replay yes/no
Conflicting replay yes/no
Crash/replay yes/no
Failure matrix passed/failed/not-executed
Focused test totals
WPF/API build totals
Disposable cleanup yes/no
Current operator DBs mutated yes/no
Private artifact yes/no
Aggregate SHA-256
```

Only the redacted coordination report may be pushed.

Private OBM source, scripts, evidence, credentials, and test data remain local.

## Final verdicts

PASS only when the real happy path and all 15 cases execute successfully:

```text
OBM_TRANSACTION_GROUP_UPLOADER_API_INGEST_E2E_READY_FOR_POS_GROUP_PULL_APPLY_ACK
```

Use one exact narrow blocker otherwise, for example:

```text
BLOCKED_SYNC_E2E_POSTGRESQL_CREDENTIAL
BLOCKED_SYNC_E2E_DISPOSABLE_DB_PROVISIONING
BLOCKED_SYNC_E2E_LOOPBACK_API_STARTUP
BLOCKED_SYNC_E2E_AUTH_HARNESS
BLOCKED_SYNC_E2E_PRODUCTION_SENDER_HARNESS
BLOCKED_SYNC_E2E_PRODUCTION_UPLOADER_HARNESS
BLOCKED_SYNC_E2E_FAILURE_INJECTION
BLOCKED_SYNC_E2E_HAPPY_PATH
BLOCKED_SYNC_E2E_FAILURE_MATRIX
```

Do not return the generic `BLOCKED_SYNC_GROUP_DISPOSABLE_ENVIRONMENT` without the first exact cause and required operator action.
