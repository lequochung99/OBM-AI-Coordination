# Prompt 119 — Restore true local-first WPF startup when retained WpfJwt is rejected

## Starting checkpoint

Report118 returned:

```text
BLOCKED_WPF_REFRESH_RESUME
```

Coordination references:

```text
report/report118.md
report118 commit:
c73fe44abbbd9d42a8c4e0962e4f56cb2d53325f

prompt118 private artifact aggregate SHA-256:
fa9ec103ee8f733f99fed9e8aedb301fd775b5c7d08c0bab9d5aa95379ee826c
```

Report118 proves:

```text
full ApiServer starts non-interactively on http://127.0.0.1:7161
Visual Studio and start-api-local.ps1 use the same full API endpoint
canonical API DB = obm_api_dev_v0_pg
health/readiness endpoints return 200
WPF protected credential record still exists
WPF protected credential decrypts successfully through DPAPI
current startup uses the same ProductRoot used at redeem
credential was not cleared or deleted
current PlatformAppV0 WpfJwt contract has no refresh-token field/path
protected hello returns WPF_HELLO_HTTP_401
MainWindow does not open
InstallationV0 is shown
```

The report's suggested conclusion that a fresh redeem is required to open the local application is rejected for normal installed-local startup.

## Authoritative operator decision — local-first

OBM-POS is local-first.

Once local Phase 2 installation, canonical PostgreSQL database, local runtime activation, and station identity are valid:

```text
MainWindow must open and local POS must remain usable even when:
- WpfJwt is expired or rejected
- the API is offline
- protected hello returns 401
- SignalR is unavailable
- sync cannot authenticate
```

Loss or expiry of an API credential may disable only:

```text
API access
cloud sync
SignalR/cloud communication
remote operations
```

It must not:

```text
route the app back to New Installation
block MainWindow
clear local activation
clear local database identity
clear the retained credential automatically
crash or close WPF
```

Do not implement a refresh-token system in this task.

Do not redeem a new Pairing Code in this task.

Do not broaden installation-scoped WpfJwt.

## Strict scope

Execute only:

```text
1. Read and verify the complete prompt118 private artifact.
2. Prove the exact startup predicate/call chain that treats WPF_HELLO_HTTP_401 as a MainWindow blocker.
3. Prove local Phase2 completion, local activation, station identity, canonical DB, and migration state independently of remote WpfJwt validity.
4. Correct the smallest existing startup/state boundary so a valid installed-local POS opens MainWindow with cloud/API status degraded when WpfJwt is rejected or API is offline.
5. Preserve the retained protected credential record; do not clear, rotate, overwrite, or redeem again.
6. Physically prove MainWindow startup with API unavailable and with protected hello returning 401.
7. Run focused tests and WPF build.
```

Do not execute or modify:

```text
Service Category Weight
Booking Weight
Price Weight save semantics
TblTenantPosDevice writer/schema/migration
API destination routing
sync happy path
CompanionApp modeling
payment-terminal modeling
POS1-POS10 PlatformAppV0 UI
Pairing Code creation/redeem behavior
ApiServer DB schema/migrations/roles
API secret/config cleanup beyond documenting the report118 result
refresh-token endpoint/DTO/storage
new token exchange service
```

Do not reset/drop/recreate WPF or API databases.

## Required evidence intake

Read completely:

```text
prompt/prompt117.md
report/report117.md
prompt/prompt118.md
prompt/prompt118_MINIMAL_WPF_STARTUP_RECOVERY_ADDENDUM.md
prompt/prompt118_API_NO_USER_SECRETS_AND_WPF_TOKEN_RESUME_ADDENDUM.md
prompt/prompt118_FULL_API_NO_USERSECRETS_AND_ENDPOINT_ALIGNMENT_ADDENDUM.md
prompt/prompt118_ENV_LOCAL_PRODUCTION_CANONICAL_SOURCE_ADDENDUM.md
report/report118.md
OBM_POS_NewChat_Handoff_V001_2026-08-02.md when locally available
```

Read and verify:

```text
E:\Project2026\RecoveryReports\WpfCanonicalInstalledLocalStartupV001
aggregate SHA-256:
fa9ec103ee8f733f99fed9e8aedb301fd775b5c7d08c0bab9d5aa95379ee826c
```

At minimum inspect complete current WPF code for:

```text
App startup and MainWindow/Installation selection
InstallationV0 Loaded/resume path
Phase1InstallationService.CallProtectedHelloAsync
all handling of HTTP 401/403 from bootstrap/me or protected hello
all methods named Clear/Delete/Reset/Invalidate/Rotate for protected credentials or installation checkpoints
Phase2 completion assessment
local runtime activation assessment
station identity assessment
canonical WPF DB resolver
migration/pending checks
Open OBM-POS enablement predicate
MainWindow eligibility predicate
cloud/API degraded-state representation
background Provider/SignalR/outbox startup behavior
```

Never expose token values, passwords, connection strings, raw tenant/device identities, or business payloads.

Record before editing:

```text
PROMPT118_ARTIFACT_VERIFIED=true
TASK_SCOPE=LOCAL_FIRST_MAINWINDOW_STARTUP_ONLY
REDEEM_NEW_PAIRING_CODE=FORBIDDEN
REFRESH_TOKEN_IMPLEMENTATION=FORBIDDEN
RETAINED_CREDENTIAL_MUTATION=FORBIDDEN
WPF_DB_RESET=FORBIDDEN
API_DB_MUTATION=FORBIDDEN
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
MANUAL_POS1_TEST_READY=false
```

## Phase 1 — Prove the exact incorrect gate

Capture direct evidence:

```text
STARTUP_PROJECT=<exact project>
STARTUP_PROFILE=<exact profile>
VISIBLE_LABEL=prompt119
PRODUCT_ROOT=<exact path>
CANONICAL_WPF_DB=<safe name>
RETAINED_CREDENTIAL_PRESENT=yes
RETAINED_CREDENTIAL_DPAPI_READ=yes
PROTECTED_HELLO_RESULT=WPF_HELLO_HTTP_401
EXACT_401_HANDLER=<class/method/line>
EXACT_MAINWINDOW_BLOCKING_PREDICATE=<class/method/line>
EXACT_INSTALLATION_ROUTE_CALLER=<class/method/line>
CREDENTIAL_CLEAR_DELETE_CALL_REACHED=no/yes with exact caller
LOCAL_PHASE2_COMPLETION=<true/false with source>
LOCAL_RUNTIME_ACTIVATION=<true/false with source>
LOCAL_STATION_IDENTITY=<valid/invalid with source>
WPF_PENDING_MIGRATIONS=<count or exact blocker>
```

Distinguish these independent states:

```text
LOCAL_INSTALLATION_VALID
REMOTE_API_CREDENTIAL_VALID
REMOTE_API_REACHABLE
SYNC_AVAILABLE
```

They must not be collapsed into one `Authorized/Unauthorized` flag.

## Phase 2 — Define the correct startup state machine

Use the existing state/coordinator types; do not create a second startup framework.

Required semantics:

### Local installation invalid

```text
Phase1 or Phase2 genuinely incomplete/corrupt
-> InstallationV0 remains open
-> precise recoverable action shown
-> no MainWindow
```

### Local installation valid, remote credential valid

```text
-> MainWindow opens
-> API/sync available
```

### Local installation valid, remote credential rejected/expired/missing

```text
-> MainWindow opens
-> cloud/API status = Reauthorization Required / Offline / Degraded
-> local checkout and local DB workflows remain enabled
-> no automatic return to InstallationV0
-> no automatic credential clear
```

### Local installation valid, API offline

```text
-> MainWindow opens
-> cloud/API status = Offline/Degraded
-> local workflows remain enabled
```

Initial installation authorization remains protected. This task changes only normal startup behavior after local installation completion is proven.

## Phase 3 — Apply the smallest correction

Allowed correction is limited to the exact existing startup/state-selection boundary, for example:

```text
separate local-installation validity from remote credential validity
translate protected-hello 401 into a nonfatal cloud-auth-degraded state after Phase2 completion
stop using Phase1 resume success as a recurring MainWindow prerequisite
prevent credential rejection from clearing/downgrading local activation
allow background reauthorization status without reopening InstallationV0
```

Forbidden:

```text
opening MainWindow when local Phase2/activation is genuinely incomplete
writing completion markers manually
copying state between ProductRoots
bypassing initial Pairing Code authorization
marking bootstrap/me AllowAnonymous
adding refresh tokens
redeeming a new code
hardcoding a token
swallowing unknown exceptions without diagnostics
```

Update visible build label to:

```text
prompt119
```

## Phase 4 — Physical acceptance matrix

Use the operator-equivalent normal Visual Studio WPF launch.

### Case A — retained credential rejected with API running

Start the full API on the canonical loopback endpoint and reproduce protected hello 401 using the retained credential.

Prove:

```text
MainWindow opens directly
InstallationV0 does not replace MainWindow
cloud/API auth status is degraded or reauthorization-required
process remains alive and responsive for at least 60 seconds
local DB access succeeds
no credential clear/delete occurs
no new redeem occurs
```

### Case B — API completely offline

Stop the API and launch WPF again.

Prove:

```text
MainWindow opens directly
process remains alive and responsive for at least 60 seconds
API/sync status is offline/degraded
local application remains usable
InstallationV0 is not shown
```

### Case C — second launch persistence

Close WPF normally and launch again while API remains offline or the retained credential remains rejected.

Prove:

```text
MainWindow opens directly again
local installation/activation remains valid
retained credential state is unchanged
no new Pairing Code redeem is required for local operation
```

Do not claim local functionality solely from process existence. Prove at least safe read-only access to canonical local DB and MainWindow responsiveness.

## Phase 5 — Focused tests and build

Run focused tests for:

```text
local-installation-valid + protected-hello 401 => MainWindow
local-installation-valid + API offline => MainWindow
remote credential rejection does not clear credential/checkpoint
remote credential rejection does not clear local activation
Phase2-incomplete still => InstallationV0
initial unpaired installation still requires Pairing Code
unknown/corrupt local state remains recoverable and does not open blindly
prompt117 HttpRequestException regression
canonical ProductRoot/DB remains selected
Firebase email/password remains absent
TblTenantPosDevice unchanged
```

Expected:

```text
all pass
0 skipped
WPF build errors=0
```

Physical proof overrides test/build success.

## End state

PASS requires:

```text
local installed state is proven valid independently of remote WpfJwt
retained credential remains present and unchanged
protected hello 401 does not block MainWindow
API offline does not block MainWindow
InstallationV0 is not shown on normal installed-local startup
MainWindow stable for 60 seconds in both 401 and offline cases
second launch opens MainWindow again
no new redeem
no refresh-token implementation
Category Weight and Booking Weight unchanged
OPERATOR_MAINWINDOW_SCREENSHOT_READY=true
MANUAL_POS1_TEST_READY=false
```

PASS verdict:

```text
OBM_WPF_LOCAL_FIRST_MAINWINDOW_RESTORED_WITH_REMOTE_AUTH_DEGRADED_READY_FOR_OPERATOR_SCREENSHOT
```

Narrow blockers only:

```text
BLOCKED_WPF_LOCAL_PHASE2_STATE_INVALID
BLOCKED_WPF_LOCAL_ACTIVATION_STATE_INVALID
BLOCKED_WPF_LOCAL_STATION_IDENTITY_INVALID
BLOCKED_WPF_MAINWINDOW_401_GATE
BLOCKED_WPF_MAINWINDOW_OFFLINE_GATE
BLOCKED_WPF_MAINWINDOW_PHYSICAL_PROOF
BLOCKED_WPF_LOCAL_FIRST_TESTS
```

Every blocker must include the exact class/method/line, predicate values, whether credential/checkpoint/local activation changed, and:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=false
MANUAL_POS1_TEST_READY=false
```

## Required private artifact

Create a new versioned artifact:

```text
E:\Project2026\RecoveryReports\WpfLocalFirstRemoteAuthDegradedV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ARTIFACT_VERIFICATION.md
REPORT118_BLOCKER_INTAKE.md
STARTUP_STATE_SEPARATION.md
EXACT_401_GATE_BEFORE.md
EXACT_401_GATE_AFTER.md
LOCAL_PHASE2_ACTIVATION_PROOF.md
CREDENTIAL_PRESERVATION_PROOF.md
API_401_MAINWINDOW_PROOF.md
API_OFFLINE_MAINWINDOW_PROOF.md
SECOND_LAUNCH_PROOF.md
LOCAL_DB_READONLY_PROOF.md
FOCUSED_TEST_OUTPUT.txt
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
report/report119.md
```

Include:

```text
Verdict
Prompt118 artifact SHA verified yes/no
Visible label
Canonical ProductRoot proof yes/no
Canonical WPF DB proof yes/no
WPF pending migrations count
Local Phase2 completion proof yes/no
Local runtime activation proof yes/no
Local station identity proof yes/no
Retained credential present before/after
Retained credential DPAPI read before/after
New redeem performed yes/no
Refresh-token implementation added yes/no
Exact prior 401 blocking class/method/line
Protected hello 401 reproduced yes/no
MainWindow opens with 401 yes/no
InstallationV0 shown with 401 yes/no
API-401 60-second stability yes/no
MainWindow opens with API offline yes/no
InstallationV0 shown with API offline yes/no
API-offline 60-second stability yes/no
Second launch MainWindow proof yes/no
Credential/checkpoint/local activation cleared or changed yes/no
Local DB read-only proof yes/no
Focused test totals
WPF build totals
TblTenantPosDevice changed no
API/schema/sync changed no
Category Weight changed no
Booking Weight changed no
Operator MainWindow screenshot ready true/false
Manual POS1 test ready false
Private artifact yes/no
Aggregate SHA-256
```

Do not expose token values, secrets, connection strings, raw identities, or business data.
