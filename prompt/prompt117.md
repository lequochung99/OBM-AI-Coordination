# Prompt 117 — Close the physical WPF InstallationV0 five-second startup crash and restore stable local-first startup

## Starting checkpoint

Prompt116 remains blocked with:

```text
BLOCKED_MAIN_DEV_POS_DEVICE_WRITER_CONTRACT
```

Updated coordination references:

```text
report/report116.md
updated report116 commit:
c33276bf0b64f454c013c0d68253ec4ad090e889

updated prompt116 private artifact aggregate SHA-256:
acc4caa30b83e1cb7b981e90b86beef919cbe97ee0a3440eac9a611cd410b949
```

The updated report116 proves:

```text
existing POS1-POS10 PlatformAppV0 UI and Pairing Code path are preserved
logical POS writer is PlatformAppV0Phase1Controller.CreatePosStation -> PlatformAppV0Store -> PlatformAppV0State.PosStations
no active canonical TblTenantPosDevice writer lifecycle is proven
no TblTenantPosDevice migration/table was created
WPF physical startup was not executed/proven
MANUAL_POS1_TEST_READY=false
```

The operator has now provided a direct physical runtime observation:

```text
WPF starts
-> enters the Installation flow
-> after approximately five seconds an error is shown
-> the WPF process crashes/exits
```

This physical startup crash is now the first blocker. Close it before resuming TblTenantPosDevice writer/schema design or any sync happy path.

## Authoritative user and architecture locks

### Local-first runtime

When the canonical local PostgreSQL database and local runtime identity/state are valid, the WPF application must start and remain usable locally.

API unavailability, expired cloud/API credential, SignalR unavailability, or a failed background sync attempt may disable/degrade cloud communication, but must not crash or terminate:

```text
WPF process
MainWindow
local checkout
local database operation
```

If installation is genuinely incomplete, the InstallationV0 UI may remain active, but it must show a recoverable error/state and must not crash the process.

### InstallationV0 remains exactly two phases

```text
Phase 1:
Pairing Code -> redeem -> WpfJwt -> bootstrap/me -> protected machine state/checkpoint -> restart/resume

Phase 2:
local PostgreSQL migration/schema -> minimal baseline seed -> completion/activation state
```

Do not merge phases or add a third startup/authentication system.

### Frozen work during this task

Do not implement or modify:

```text
TblTenantPosDevice writer lifecycle
TblTenantPosDevice entity/mapping/migration/table
TblPosLocal semantics
POS1-POS10 PlatformAppV0 UI
Pairing Code API/UI behavior
API destination-routing schema
sync happy path
Service Category Weight
Booking Weight
POS2 pull/apply/ACK
```

Do not reset/drop/recreate either canonical Development database.

Do not create a second WPF startup lane, second installation module, alternate ProductRoot, alternate DbContext, alternate provider, or fallback database.

### Authentication/provider locks

Preserve:

```text
Pairing Code -> WpfJwt -> bootstrap/me for installation/bootstrap
canonical Provider owns normal WPF-to-API token/header behavior
Firebase email/password remains deleted
```

Do not bypass WpfJwt, restore Firebase, weaken API authorization, hardcode a token, or add manual token/header logic.

## Strict scope

Execute only:

```text
1. Read and verify the complete updated prompt116 private artifact.
2. Build the latest WPF source and add the visible prompt117 runtime label required by the operator, without changing business behavior.
3. Reproduce the exact WPF startup using the canonical Visual Studio Development lane and canonical ProductRoot/state.
4. Capture the exact five-second failure boundary with direct exception/call-stack/thread/log evidence.
5. Prove why startup enters InstallationV0 rather than MainWindow.
6. Classify whether the crash is caused by installation state, timer/background callback, API/bootstrap call, token/checkpoint handling, DB activation state, dispatcher handling, or another exact boundary.
7. Correct only the smallest production-capable WPF startup defect.
8. Physically prove stable behavior for valid installed-local state and incomplete/recoverable installation state.
9. Run focused startup/local-first tests and build WPF.
```

Do not proceed into the prompt116 writer/schema task after the startup fix. End this task after stable WPF startup proof.

## Required evidence intake

Read completely:

```text
OBM_POS_NewChat_Handoff_V001_2026-08-02.md when locally available
prompt/prompt107.md
report/report107.md
prompt/prompt109.md
report/report109.md
prompt/prompt111.md
report/report111.md
prompt/prompt112.md
report/report112.md
prompt/prompt116.md
prompt/prompt116_POS_STATION_DEVICE_AGGREGATE_ADDENDUM.md
prompt/prompt116_EXISTING_POS1_10_UI_AND_WPF_STARTUP_ADDENDUM.md
report/report116.md
```

Read and verify the updated prompt116 artifact:

```text
E:\Project2026\RecoveryReports\MainDevPosDeviceSchemaWriterAndSyncHappyPathV001
aggregate SHA-256:
acc4caa30b83e1cb7b981e90b86beef919cbe97ee0a3440eac9a611cd410b949
```

At minimum inspect complete current WPF source for:

```text
App.xaml/App.xaml.cs startup and exception handlers
Program/Main entry when present
InstallationV0 startup coordinator and window/navigation logic
all DispatcherTimer/System.Threading.Timer/Task.Delay calls near startup
all five-second constants/timeouts/debounces/retries
all async void startup/event handlers
all fire-and-forget tasks launched before MainWindow
all Application.Current.Shutdown/Environment.Exit/Close calls
all unhandled exception handlers and logger sinks
Phase 1 protected state/checkpoint readers
WpfJwt expiry/identity/contract validation
bootstrap/me caller and timeout/error handling
Phase 2 completion/activation markers
canonical ProductRoot resolution
canonical WPF Development DB connection resolution
migration/pending/current-state checks
MainWindow selection/opening logic
background outbox worker startup registration
SignalR startup/connection registration
```

Inspect current local non-secret state metadata only:

```text
resolved ProductRoot path
state/checkpoint file names and existence
installation phase/status markers
protected credential presence/absence marker only
credential expiration metadata without token value
local DB safe name/provider/host classification
migration history/pending count when connection is safe
latest WPF executable path/hash/build label
```

Never expose passwords, WpfJwt/token values, complete connection strings, passfile contents, raw tenant/device identities, or business data.

Record before editing:

```text
PROMPT116_ARTIFACT_VERIFIED=true
TASK_SCOPE=WPF_STARTUP_CRASH_ONLY
TBLTENANTPOSDEVICE_CHANGES=FORBIDDEN
API_DB_MUTATION=FORBIDDEN
WPF_DB_RESET=FORBIDDEN
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
MANUAL_POS1_TEST_READY=false
```

## Phase 1 — Prove the exact physical process and latest build

Build the canonical WPF source under:

```text
E:\Project2026\4POS\NailSalonNet8
```

Add/update the visible WPF label to:

```text
prompt117
```

The label is only runtime-version evidence. Do not use it to bypass logic.

Before launch record safely:

```text
WPF_EXE_PATH=<exact path>
WPF_EXE_SHA256=<hash>
BUILD_TIMESTAMP=<safe timestamp>
VISIBLE_LABEL=prompt117
PRODUCT_ROOT=<exact local path>
ENVIRONMENT=Development
EXPECTED_DB=<safe canonical WPF Development DB name>
```

Stop only stale WPF processes from this exact source/build lane. Do not kill unrelated apps.

Launch the actual executable/debug target, not a test-only substitute.

## Phase 2 — Capture the five-second crash with 100% direct evidence

Reproduce the operator-observed sequence and capture:

```text
PROCESS_START_UTC
INSTALLATION_UI_SHOWN=yes/no
VISIBLE_LABEL_OBSERVED=prompt117 yes/no
SECONDS_TO_ERROR=<measured>
ERROR_UI_TEXT=<sanitized exact text>
PROCESS_EXITED=yes/no
PROCESS_EXIT_CODE=<value>
FIRST_CHANCE_EXCEPTION_TYPE=<type when available>
UNHANDLED_EXCEPTION_TYPE=<type>
SANITIZED_EXCEPTION_MESSAGE=<exact sanitized message>
INNER_EXCEPTION_CHAIN=<types/messages sanitized>
THROWING_METHOD=<exact class/method/line>
CALL_STACK=<complete relevant stack>
THREAD_CONTEXT=<UI dispatcher/background/task/timer>
TRIGGER=<timer/api callback/state check/etc.>
LAST_SUCCESSFUL_STARTUP_CHECK=<exact boundary>
FIRST_FAILED_STARTUP_CHECK=<exact boundary>
```

Capture Windows Event Log/.NET runtime log and application logs when available. Prefer debugger break-on-thrown plus unhandled exception evidence. Do not rely only on the visible message.

Search and prove every approximately five-second timer/timeout path before selecting the cause.

If the process does not reproduce under debugger but crashes outside debugger, run the same executable/state both ways and capture the difference. Do not report PASS from a non-representative test harness.

## Phase 3 — Prove why InstallationV0 is selected

Map the exact startup decision tree:

```text
protected Phase 1 state present/valid?
bootstrap/me checkpoint valid?
Phase 2 completion marker present/valid?
canonical DB reachable?
migration current?
runtime activation identity valid?
MainWindow eligibility predicate?
Installation window eligibility predicate?
```

Return exact predicate values and call sites, with sensitive values redacted.

Classify the current state as exactly one:

```text
S1_VALID_INSTALLED_LOCAL_STATE_SHOULD_OPEN_MAINWINDOW
S2_PHASE1_INCOMPLETE_SHOULD_STAY_INSTALLATION
S3_PHASE2_INCOMPLETE_SHOULD_STAY_INSTALLATION
S4_CORRUPT_OR_INCONSISTENT_CHECKPOINT_REQUIRES_RECOVERABLE_REPAIR_UI
S5_RUNTIME_DB_OR_ACTIVATION_STATE_DEFECT
S6_OTHER_EXACTLY_PROVEN_STATE
```

Do not force MainWindow when installation is truly incomplete. Do not force Installation when the local installed state is valid.

## Phase 4 — Correct the smallest startup defect

Allowed correction categories include only what direct evidence proves, for example:

```text
catch/translate a known API/bootstrap timeout into recoverable Installation UI state
remove Application.Shutdown/throw from a recoverable background failure
await and handle a fire-and-forget startup task correctly
marshal timer callbacks safely to the dispatcher
prevent an expired/missing remote credential from terminating valid local runtime
correct an installation-state predicate/checkpoint inconsistency
correct a canonical local DB/activation-state read defect
prevent background outbox/SignalR startup failures from becoming process-fatal
```

Requirements:

```text
no swallowed unknown exceptions without logging
no empty catch blocks
no global catch that hides corruption and opens MainWindow blindly
no retry loop faster than existing safe cadence
no second startup coordinator
no new identity/security gate
no API-online requirement for valid local MainWindow startup
```

If the exact defect requires operator action rather than source correction, keep the WPF process alive in a clear recoverable UI and report the exact action/state required. A recoverable Installation UI is acceptable; process crash is not.

## Phase 5 — Physical acceptance matrix

Prove the applicable cases physically using preserved state or safe versioned copies. Do not destroy the operator's only state without backup/versioning.

### Case A — Current operator state

```text
latest prompt117 build launches
exact current ProductRoot/state is used
process remains alive
no unhandled exception
Installation UI or MainWindow matches the proven state classification
error, when present, is recoverable and does not close the process
```

### Case B — Valid installed-local state

When a valid installed local checkpoint/DB state is available or can be proven from a safe existing Development state:

```text
MainWindow opens
local DB is canonical PostgreSQL/Npgsql
pending migrations=0 when safe to query
API offline/unreachable does not terminate MainWindow
expired/unavailable cloud credential does not terminate MainWindow
```

Do not fabricate installation completion markers merely to pass this case. If no valid installed state exists, report `NOT_AVAILABLE` with exact evidence and prove Case A plus Case C.

### Case C — Incomplete/recoverable installation state

```text
InstallationV0 UI remains open for at least the physical observation window
no process crash
no unhandled exception
retry/back/corrective action remains available according to existing UI contract
no hidden Firebase/email-password fallback
```

### Observation window

For each applicable physical launch, keep the process alive long enough to pass the original approximately five-second failure point multiple times and prove stable state. Record actual observed duration; do not infer stability from immediate launch.

## Phase 6 — Focused tests and build

Add/run focused tests for the exact defect and at minimum:

```text
startup state decision predicates
recoverable bootstrap/API timeout does not call Shutdown
background timer/task exception is observed and handled at the owning boundary
valid local state is not blocked by API/token/cloud outage
incomplete installation remains in Installation UI without process exit
corrupt/inconsistent checkpoint produces recoverable state, not crash
canonical ProductRoot/DB lane remains selected
Firebase email/password remains absent
TblTenantPosDevice source/schema remains unchanged
```

Run WPF build after correction.

Expected:

```text
all focused tests pass
0 skipped
WPF build errors=0
```

Build/test success does not override a failed physical startup proof.

## End state

PASS requires:

```text
exact five-second crash root cause proven
smallest production-capable correction applied
latest prompt117 build physically launched
current state no longer crashes
valid local installed state opens/keeps MainWindow when available
incomplete installation remains recoverable without process exit
API/token/SignalR/background failure cannot terminate valid local operation
TblTenantPosDevice writer/schema remains untouched
POS1-POS10 PlatformAppV0 UI and Pairing Code remain untouched
Category Weight unchanged
Booking Weight unchanged
MANUAL_POS1_TEST_READY=false
```

`MANUAL_POS1_TEST_READY` remains false because the TblTenantPosDevice writer/routing and sync happy path are still unresolved. PASS from this task only permits returning to that design after WPF startup is stable.

## Required private artifact

Preserve earlier artifacts unchanged. Create:

```text
E:\Project2026\RecoveryReports\WpfInstallationStartupCrashV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ARTIFACT_VERIFICATION.md
UPDATED_REPORT116_INTAKE.md
BUILD_AND_EXE_IDENTITY.md
STARTUP_DECISION_TREE.md
CURRENT_STATE_CLASSIFICATION.md
FIVE_SECOND_TIMER_AUDIT.md
PHYSICAL_CRASH_REPRO.md
EXCEPTION_AND_CALLSTACK.md
WINDOWS_EVENT_LOG.md
STARTUP_BEFORE.md
STARTUP_AFTER.md
LOCAL_FIRST_CONTRACT_PROOF.md
CURRENT_STATE_PHYSICAL_PROOF.md
VALID_INSTALLED_STATE_PROOF.md
INCOMPLETE_INSTALLATION_PROOF.md
FOCUSED_TEST_OUTPUT.txt
WPF_BUILD_OUTPUT.txt
FINAL_STATE.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Use `NOT_AVAILABLE` with exact reason for evidence files that cannot apply; do not omit them silently.

## Public report

Create and push only:

```text
report/report117.md
```

Include:

```text
Verdict
Prompt116 updated artifact SHA verified yes/no
Latest WPF label observed
WPF executable/hash proof yes/no
Current startup state classification S1-S6
Installation UI shown yes/no
Measured seconds to original error
Exact crash trigger category
Exact throwing class/method
Unhandled exception type
Process exited before correction yes/no and exit code
Process exited after correction yes/no
Current-state physical startup stable yes/no
Current-state observed duration
Valid installed-local state proof yes/no/not-available
MainWindow physical proof yes/no/not-available
Incomplete installation recoverable proof yes/no
API/cloud outage local-first proof yes/no/not-applicable
WPF canonical runtime DB proof yes/no/not-reached
WPF pending migrations count or not-reached
Focused test totals
WPF build totals
TblTenantPosDevice changed yes/no
API DB mutated yes/no
WPF DB reset performed yes/no
POS1-POS10 UI/Pairing Code changed yes/no
Category Weight changed yes/no
Booking Weight changed yes/no
Manual POS1 test ready false
Production/customer/reference DB mutated yes/no
Private artifact yes/no
Aggregate SHA-256
```

Do not expose tokens, passwords, complete connection strings, protected state contents, raw tenant/device identities, or business data.

## Verdicts

PASS:

```text
OBM_WPF_INSTALLATION_STARTUP_CRASH_CLOSED_LOCAL_FIRST_STABLE_READY_TO_RESUME_POS_DEVICE_WRITER_DESIGN
```

Narrow blockers only:

```text
BLOCKED_WPF_STARTUP_CRASH_NOT_REPRODUCED
BLOCKED_WPF_STARTUP_EXCEPTION_CAPTURE
BLOCKED_WPF_STARTUP_STATE_CLASSIFICATION
BLOCKED_WPF_INSTALLATION_RECOVERY_UI
BLOCKED_WPF_LOCAL_DB_RUNTIME_STATE
BLOCKED_WPF_STARTUP_PHYSICAL_PROOF
BLOCKED_WPF_STARTUP_FOCUSED_TESTS
```

Every blocked result must include the exact executable/build identity, current state classification or why unavailable, exception/call-stack evidence when available, process exit state, DB/reset state, and:

```text
MANUAL_POS1_TEST_READY=false
```

Do not return a generic startup blocker.
