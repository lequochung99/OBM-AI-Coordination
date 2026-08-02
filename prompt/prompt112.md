# Prompt 112 — Close the canonical ApiServer non-interactive PostgreSQL startup credential boundary and resume the single POS1 → API happy path

## Starting checkpoint

Prompt111 returned:

```text
BLOCKED_MAIN_DEV_API_STARTUP
```

Coordination references:

```text
report/report111.md
report111 commit:
b3487cea1b8b6dc593f347056eb7c98a85db61c5

prompt111 private artifact aggregate SHA-256:
e2ce5d442e56f25133abd77b16a44268ec4cae1b595682ccd5b26f219f226a5c
```

Report111 proves:

```text
all accepted artifact SHAs were verified
no Firebase/email-password path was used
no Development identity/routing rows were created
no Price Rule/domain/outbox/event/delivery writes occurred
no SignalR publish occurred
no WPF/API DB reset occurred
no production/customer/reference DB was mutated
```

The exact public blocker is:

```text
start-api-local.ps1 loaded its configured environment source,
but the LocalDevelopment ApiServer process did not receive a complete PostgreSQL credential/configuration for the canonical API Development database.
The script then entered a hidden interactive password prompt.
Execution stopped before any write.
```

This task must close that narrow startup boundary and then resume the same prompt111 happy path from the beginning.

## Authoritative decisions

### Canonical database

The ApiServer runtime target remains the canonical API Development database accepted by reports108 and 109:

```text
obm_api_dev_v0_pg
```

Do not create, rename, reset, drop, recreate, or migrate a different database.

Do not restore the obsolete noncanonical user-secret DB override removed by prompt109.

### Canonical authentication boundaries

The retired Firebase email/password path remains deleted.

The WPF installation/bootstrap flow remains:

```text
Pairing Code -> redeem -> WpfJwt -> bootstrap/me
```

Do not broaden installation-scoped WpfJwt into a runtime sync credential unless the current production source already explicitly defines that contract.

The PostgreSQL database credential is a separate infrastructure credential. Do not confuse it with WpfJwt, Firebase, POS sync authentication, or Platform administrator authentication.

### Canonical sync architecture

There is still exactly one sync flow:

```text
WPF domain Save + TblLocalOutbox in one local transaction
-> existing periodic WPF outbox worker
-> existing canonical FlushOutbox/API service path
-> existing standard API grouped sync controller/service
-> one TblEventLog/TblEventDelivery transaction
-> one successful API commit
-> existing post-commit SignalR notification
```

Do not create a second uploader, worker, endpoint, ingest pipeline, event writer, delivery transport, ACK system, or SignalR publisher.

## Strict scope

Execute only:

```text
1. Read and verify the complete prompt111 private artifact.
2. Identify the exact missing/incomplete ApiServer PostgreSQL runtime credential/configuration boundary.
3. Audit configuration precedence for start-api-local.ps1 and the LocalDevelopment ApiServer process.
4. Reuse the existing approved protected PostgreSQL runtime credential mechanism.
5. Make ApiServer local startup non-interactive and fail-closed for automation.
6. Prove ApiServer starts on loopback against obm_api_dev_v0_pg with pending migrations = 0.
7. Resume and complete the full prompt111 one-group POS1 -> API happy path.
8. Rerun focused tests, architecture guards, and WPF/API builds.
```

Do not execute:

```text
15-case failure/recovery matrix
Category Weight implementation
Booking Weight implementation
POS2 pull/apply/ACK
checkout/payment changes
Queue changes
BookingConsole changes
DB reset or migration generation
cloud/production deployment
```

## Required evidence intake

Read completely:

```text
prompt/prompt108.md
report/report108.md
prompt/prompt109.md
report/report109.md
prompt/prompt110.md
report/report110.md
prompt/prompt111.md
report/report111.md
```

Read and verify:

```text
E:\Project2026\RecoveryReports\MainApiDevResetExecutionV001
aggregate SHA-256:
e9d8298486f31f40581cb4445fa0abac25030bd586303098c05e1a9225f0d0ea

E:\Project2026\RecoveryReports\LegacyFirebaseUserSecretRemovalV001
aggregate SHA-256:
b97b2eaed1738c497502c92b057c5133bf6b20345d302b3daea44541d0012dfa

E:\Project2026\RecoveryReports\CanonicalSyncFlowConsolidationV001
aggregate SHA-256:
a7d113ef381c07095b3ccd4145de734d4011e5eb51a78d2f6c7f6095ae868ccd

prompt111 private artifact
aggregate SHA-256:
e2ce5d442e56f25133abd77b16a44268ec4cae1b595682ccd5b26f219f226a5c
```

At minimum inspect:

```text
start-api-local.ps1 complete body and every helper it calls
all .ps1/.cmd/.bat launch wrappers for ApiServer local Development
launchSettings.json profiles and environment names
appsettings.json and appsettings.LocalDevelopment.json/appsettings.Development.json when present
Program.cs configuration/provider registration
ExternalDbContext runtime connection resolution
all environment-variable names read by the script and process
all PGPASSFILE/protected credential integrations
all user-secret keys by key name only
report109 runtime configuration precedence evidence
report108 canonical DB resolution/reset evidence
prompt111 startup logs and exact hidden-prompt call site
```

Never print secret values, complete connection strings, passfile contents, tokens, or passwords.

Record before editing:

```text
PROMPT108_ARTIFACT_VERIFIED=true
PROMPT109_ARTIFACT_VERIFIED=true
PROMPT110_ARTIFACT_VERIFIED=true
PROMPT111_ARTIFACT_VERIFIED=true
CANONICAL_API_DB=obm_api_dev_v0_pg
FIREBASE_EMAIL_PASSWORD=RETIRED
USER_SECRET_DB_OVERRIDE=FORBIDDEN
INTERACTIVE_PASSWORD_PROMPT_IN_AUTOMATION=FORBIDDEN
DB_RESET=FORBIDDEN
SYNC_ARCHITECTURE_CHANGE=FORBIDDEN
```

## Phase 1 — Prove the exact startup failure

Before changing anything, capture sanitized direct evidence:

```text
START_SCRIPT=<exact path>
START_PROFILE=<exact profile/environment>
CONFIG_SOURCES_IN_PRECEDENCE_ORDER=<key-name/source names only>
CANONICAL_DB_NAME_RESOLVED=<safe database name>
HOST_RESOLVED=<loopback/approved local classification>
PORT_RESOLVED=<safe port>
USERNAME_PRESENT=<yes/no; do not print value when sensitive>
PASSWORD_OR_PASSFILE_RESOLVED=<yes/no; do not print value/path contents>
EXACT_MISSING_KEY_OR_BOUNDARY=<key/source name only>
HIDDEN_PROMPT_CALL_SITE=<script/function/line>
SANITIZED_EXCEPTION_OR_RESULT=<exact sanitized result>
POSTGRES_SQLSTATE=<value or NOT_AVAILABLE>
```

Determine whether the failure is one of:

```text
A. script loaded the wrong environment/profile
B. script did not propagate an already available protected credential to the child process
C. runtime connection builder ignored the approved protected source
D. obsolete prompt fallback remained after prompt109 cleanup
E. the approved normal runtime credential is genuinely absent
F. another narrowly proven configuration defect
```

Do not guess.

If the normal runtime credential is genuinely absent and cannot be provisioned through an existing approved protected mechanism, return:

```text
BLOCKED_MAIN_DEV_API_RUNTIME_CREDENTIAL_ABSENT
```

with the exact required key/source name and setup boundary, but no secret value.

## Phase 2 — Correct the smallest canonical credential/configuration defect

Requirements:

```text
reuse the existing protected PostgreSQL runtime credential/configuration mechanism
normal ApiServer runtime credential must be distinct in purpose from destructive maintenance/admin credential unless current source explicitly proves otherwise
no password literal in source, scripts, launchSettings, appsettings, reports, or artifacts
no complete connection string in source or report
no restored Firebase credential
no restored obsolete user-secret DB override
no new credential framework
no second configuration pipeline
no hidden prompt during automated/non-interactive startup
```

Preferred behavior for `start-api-local.ps1`:

```text
resolve canonical DB/host/provider and approved protected credential source
validate required non-secret fields
start ApiServer non-interactively
fail immediately with a named missing-source error when credential resolution fails
never wait indefinitely for hidden input in automation
```

An interactive operator mode may remain only when it is an explicitly selected separate mode and cannot be entered accidentally by the canonical automated Development startup path.

If the current approved mechanism uses PGPASSFILE, environment variables, DPAPI-protected local state, or another existing protected source, preserve that design and connect it correctly. Do not disclose its contents.

Add focused regression tests for:

```text
canonical LocalDevelopment/Development profile selects obm_api_dev_v0_pg
approved protected credential source reaches the child process/runtime connection builder
non-interactive mode never invokes hidden password prompt
missing credential fails fast with a sanitized named error
obsolete noncanonical user-secret DB override remains absent
Firebase keys remain absent
```

## Phase 3 — Prove canonical ApiServer startup

Start the real ApiServer from:

```text
E:\Project2026\1ApiServer\ApiServer01
```

Use the corrected canonical local startup boundary.

Required proof:

```text
startup is non-interactive
Environment is the intended local Development environment
provider = Npgsql/PostgreSQL
host is loopback or approved local Development
runtime DB = obm_api_dev_v0_pg
runtime DB differs from WPF DB and maintenance DB
pending migrations = 0
readiness/health succeeds
loopback-only binding
canonical grouped sync route is loaded
production authorization policy is active
no secret value is logged
```

Stop the process cleanly after the complete happy-path task.

## Phase 4 — Resume prompt111 without bypasses

After startup proof succeeds, resume all remaining prompt111 phases exactly through production boundaries.

### Minimal Development prerequisites

Create only the existing-contract prerequisites for:

```text
one generated Development tenant
one generated POS1 source identity
one generated POS2 destination identity/subscriber mapping
subscriptions/routing for exactly TblTurnPolicy and TblTurnAmountRule
minimal Price Rule Save prerequisites
```

Do not use production/customer identifiers.

Do not manually insert:

```text
TblLocalOutbox
TblEventLog
TblEventDelivery
Price Rule outbox payloads
```

### Real domain Save

Use the production Price Rule Save boundary to create one true change.

Prove:

```text
one local DbContext
one explicit local PostgreSQL transaction
domain rows + complete TblLocalOutbox group commit together
no partial local state
policy-first ordering
contiguous SequenceNumber values
ExpectedEventCount equals actual group size
```

### Existing periodic worker

Use the existing registered production periodic WPF outbox worker and canonical FlushOutbox chain.

Do not use a manual HTTP request, controller-direct invocation, second worker, or uploader harness.

Prove:

```text
one worker cycle
one atomic group claim
one grouped HTTP request
no parallel production path invoked
```

### API durable commit

For the generated TransactionGuid, prove:

```text
production sync authentication/identity validation passed
one API transaction began
TblEventLog count = ExpectedEventCount
TblEventDelivery destination count = ExpectedEventCount per valid destination
TblEventDelivery source count = 0
one durable API commit
no partial durable state
```

### SignalR

Prove:

```text
API commit completed before SignalR publish attempt
existing SignalR publisher path only
notification-only metadata, no business payload
publish succeeded
existing destination listener observed notification when current contract supports observation
```

### Local completion

Prove all group rows are marked Sent together with claims cleared and no mixed status.

Required marker:

```text
SYNC_GROUP_MAIN_DEV_HAPPY_PATH_COMMITTED
```

## Phase 5 — Tests and builds

Run focused tests for:

```text
ApiServer protected runtime credential resolution
non-interactive startup/no hidden prompt
canonical API DB selection
pending migrations = 0
production worker -> canonical FlushOutbox chain
complete-group claim/validation
one grouped request
production sync auth/identity match
API atomic EventLog/Delivery commit
source exclusion
SignalR after commit
all-or-none local completion
```

Rerun the prompt110 architecture guards.

Expected:

```text
all pass
0 skipped
parallel production paths remaining = 0
```

Build:

```text
WPF
ApiServer
```

Build/test success does not override failed physical startup or happy-path proof.

## End state

Leave:

```text
canonical API Development DB migration-current
canonical WPF Development DB migration-current
ApiServer local startup non-interactive through approved protected credential source
legacy Firebase/email-password absent
obsolete DB user-secret override absent
one successful canonical POS1 -> API transaction-group happy path recorded
Development-only E2E prerequisites documented for reuse by the later failure matrix
no production/customer/reference mutation
```

## Required private artifact

Preserve all earlier artifacts unchanged. Create:

```text
E:\Project2026\RecoveryReports\MainDevApiStartupAndSyncHappyPathV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ARTIFACT_VERIFICATION.md
PROMPT111_BLOCKER_INTAKE.md
STARTUP_CONFIG_PRECEDENCE.md
EXACT_CREDENTIAL_FAILURE.md
START_API_BEFORE.md
START_API_AFTER.md
NON_INTERACTIVE_STARTUP_PROOF.md
API_RUNTIME_DB_PROOF.md
SYNC_AUTH_CALL_CHAIN.md
DEVELOPMENT_PREREQUISITES.md
PRICE_RULE_SAVE_PROOF.md
WPF_WORKER_PROOF.md
GROUPED_HTTP_PROOF.md
API_TRANSACTION_PROOF.md
SIGNALR_AFTER_COMMIT_PROOF.md
LOCAL_COMPLETION_PROOF.md
HAPPY_PATH_COUNTS.md
FOCUSED_TEST_OUTPUT.txt
ARCHITECTURE_GUARD_OUTPUT.txt
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
report/report112.md
```

Include:

```text
Verdict
Prompt111 artifact SHA verified yes/no
Exact startup script/profile
Exact credential/configuration failure classification
Protected PostgreSQL runtime credential source name only
Hidden password prompt reachable after task yes/no
Canonical API DB runtime proof yes/no
API startup non-interactive yes/no
ApiServer loopback readiness yes/no
API pending migrations count
Production sync auth physically passed yes/no
Development prerequisites created yes/no
Production Price Rule Save used yes/no
Local domain + outbox atomic commit yes/no
Local group expected/actual count
Local group sequence/order proof yes/no
Existing periodic WPF worker used yes/no
Production worker cycle count
Grouped HTTP request count
Canonical API ingest action used yes/no
API transaction begin count
API durable commit count
TblEventLog group row count
TblEventDelivery destination row count
TblEventDelivery source row count
Source exclusion proof yes/no
SignalR attempted after commit yes/no
SignalR publish succeeded yes/no
Destination notification observed yes/no/not-applicable
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

Do not expose passwords, tokens, complete connection strings, passfile contents, raw Development identities, or business payload values.

## Verdicts

PASS:

```text
OBM_MAIN_DEV_API_STARTUP_AND_CANONICAL_SYNC_HAPPY_PATH_READY_FOR_FAILURE_MATRIX
```

Narrow blockers only:

```text
BLOCKED_MAIN_DEV_API_RUNTIME_CREDENTIAL_ABSENT
BLOCKED_MAIN_DEV_API_CREDENTIAL_PROPAGATION
BLOCKED_MAIN_DEV_API_NONINTERACTIVE_STARTUP
BLOCKED_MAIN_DEV_API_RUNTIME_DB_DRIFT
BLOCKED_MAIN_DEV_API_STARTUP
BLOCKED_MAIN_DEV_SYNC_AUTH
BLOCKED_MAIN_DEV_DESTINATION_ROUTING
BLOCKED_MAIN_DEV_PRICE_RULE_SAVE
BLOCKED_MAIN_DEV_GROUP_UPLOAD
BLOCKED_MAIN_DEV_API_COMMIT
BLOCKED_MAIN_DEV_SIGNALR
BLOCKED_MAIN_DEV_LOCAL_COMPLETION
BLOCKED_MAIN_DEV_HAPPY_PATH_TESTS
```

A blocked result must name the exact class/script/method/config source, sanitized failure, SQLSTATE when available, and all write/reset state. Do not return a generic startup or credential blocker.
