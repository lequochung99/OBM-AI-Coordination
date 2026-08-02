# Prompt 118 — Close uncontrolled re-entry into InstallationV0 and restore canonical installed-local MainWindow startup

## Starting checkpoint

Prompt117 returned:

```text
OBM_WPF_INSTALLATION_STARTUP_CRASH_CLOSED_LOCAL_FIRST_STABLE_READY_TO_RESUME_POS_DEVICE_WRITER_DESIGN
```

Coordination references:

```text
report/report117.md
report117 commit:
d23642565c374b87a7cb5e7ed6b6d9fafca73013

prompt117 private artifact aggregate SHA-256:
257afc02890a0b560e0c50d60d8f51a715b8203400f31d6f13e7e08653973395
```

Report117 proves the original process-terminating defect was closed:

```text
Phase1InstallationService.CallProtectedHelloAsync raised HttpRequestException
that exception escaped the async WPF Loaded resume path
before correction the process exited with an unhandled exception
with label prompt117 after correction the InstallationV0 window remained alive/responding for 20 seconds
```

However, report117 did **not** prove a valid installed-local startup or MainWindow. It classified the current state as:

```text
S3_PHASE2_INCOMPLETE_SHOULD_STAY_INSTALLATION
```

The operator has now provided direct physical screenshot evidence from the latest prompt117 build. Normal WPF startup shows:

```text
window title: OBM InstallationV0 Phase 1/2 - prompt117
Phase 2 Local DB Baseline: awaiting Phase 1 resume
Local POS status: Unknown
API status: Unknown
Target DB: obm_pos_dev_v0_pg (Development/Test)
Open OBM-POS: disabled
ProductRoot: E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
```

The operator states that the application has fallen into the Installation flow without control. The immediate crash is contained, but startup routing is still not accepted.

This task must determine whether the normal WPF launch is using the wrong ProductRoot/profile, whether a valid installed-local state is being ignored because API/Phase1 resume is unavailable, whether the canonical DB is genuinely missing Phase2 activation/baseline state, or whether another exact state-drift defect exists. Then repair only that boundary and physically prove the correct normal startup behavior.

## Authoritative interpretation

Prompt117 is accepted only for **crash containment**. It is not final acceptance of the startup decision.

Do not accept `S3_PHASE2_INCOMPLETE_SHOULD_STAY_INSTALLATION` merely because the currently selected ProductRoot is named `WpfInstallationV0Phase1` or because a remote/bootstrap call fails. Re-prove the canonical normal-development ProductRoot, launch profile, protected local state, DB state, Phase2 completion state, and MainWindow predicate from direct evidence.

## Authoritative local-first startup contract

After installation has completed and the protected local activation/runtime state is valid:

```text
canonical local PostgreSQL DB valid
+ local station/runtime identity valid
+ local Phase2 completion/activation state valid
=> normal WPF startup opens MainWindow
```

The following may degrade API/sync/cloud functionality but must not route an already installed app back into new installation or block MainWindow:

```text
API offline
bootstrap/me unavailable
WpfJwt expired/unavailable after installation completion
SignalR unavailable
background sync unavailable
```

InstallationV0 is selected only when local installation is genuinely fresh/incomplete, or when protected local state is corrupt/inconsistent and requires a recoverable repair UI.

Phase1 remote resume must not be re-required on every normal startup after Phase2/local activation is complete.

## Frozen architecture and feature work

Do not implement or modify in this task:

```text
TblTenantPosDevice writer lifecycle
TblTenantPosDevice entity/mapping/migration/table
API destination routing
POS1-POS10 PlatformAppV0 UI
Pairing Code API/UI behavior
canonical Provider authentication behavior
sync happy path
Service Category Weight
Booking Weight
POS2 pull/apply/ACK
```

Do not reset/drop/recreate WPF or API databases.

Do not create:

```text
second ProductRoot
second startup coordinator
second installation module
alternate DbContext/provider
fallback database
manual completion-marker writer
manual token/header path
Firebase/email-password fallback
```

## Strict scope

Execute only:

```text
1. Read and verify the complete prompt117 private artifact.
2. Reproduce the exact operator screenshot state using the latest normal WPF Visual Studio Development launch.
3. Prove the actual startup project, executable, launch profile, environment, ProductRoot source, and precedence.
4. Audit every existing ProductRoot/profile candidate and identify the one canonical normal-development root without creating another root.
5. Prove the complete Phase1/Phase2/local-activation/DB state behind the startup decision.
6. Determine whether current startup routing is profile/root drift, invalid state precedence, genuine incomplete Phase2, completion-marker drift, protected-state corruption, or another exact defect.
7. Correct only the smallest production-capable startup/state-reconciliation defect.
8. When local installation is truly complete, physically prove normal startup opens MainWindow and remains local-first with API unavailable.
9. When installation is truly incomplete, prove the UI reports the exact missing phase/action and remains recoverable; do not falsely mark completion.
10. Run focused tests and build WPF.
```

Do not resume the prompt116 POS-device writer task inside prompt118.

## Required evidence intake

Read completely:

```text
OBM_POS_NewChat_Handoff_V001_2026-08-02.md when locally available
prompt/prompt107.md
report/report107.md
prompt/prompt109.md
report/report109.md
prompt/prompt116.md
prompt/prompt116_POS_STATION_DEVICE_AGGREGATE_ADDENDUM.md
prompt/prompt116_EXISTING_POS1_10_UI_AND_WPF_STARTUP_ADDENDUM.md
report/report116.md
prompt/prompt117.md
report/report117.md
```

Read and verify:

```text
E:\Project2026\RecoveryReports\WpfInstallationStartupCrashV001 or the exact prompt117 artifact path
aggregate SHA-256:
257afc02890a0b560e0c50d60d8f51a715b8203400f31d6f13e7e08653973395

E:\Project2026\RecoveryReports\MainWpfDevResetExecutionV002
aggregate SHA-256:
47f68c634a5984611f3cb8b39ba3999f6005a558ad1e0d64bf998f7f4c2a0c58
```

Recover the exact prompt117 artifact path from its `PRIVATE_HANDOFF.md`; do not guess if the suggested folder name differs.

At minimum inspect complete current source/configuration for:

```text
App.xaml/App.xaml.cs startup decision
normal Visual Studio startup project and executable
launchSettings/debug-profile/environment settings
all ProductRoot environment/configuration keys and precedence
ProductRoot canonical resolver and guardrails
InstallationV0 coordinator/window selection
MainWindow eligibility predicate
Phase1 protected state/checkpoint reader
Phase1 completion and resume markers
WpfJwt expiration/contract handling after installation completion
Phase2 assessment/baseline executor/completion marker
runtime activation state and local station identity
canonical WPF DB resolver
migration-history/pending-migration checks
minimal baseline seed/required marker checks
Open OBM-POS button enablement predicate
all code that can clear, rotate, downgrade, or invalidate installation state
```

Inspect non-secret local evidence for all existing candidate roots, including the currently observed:

```text
E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
```

For each candidate record only:

```text
path
configuration source/caller
intended lane: normal Development / installation test / stale / unknown
protected state/checkpoint file names and existence
Phase1 completion marker
Phase2 completion/activation marker
bound safe DB name
last-write timestamps
whether current launch can select it
```

Do not print token values, passwords, complete connection strings, passfile contents, raw tenant/device identities, or private data.

Record before editing:

```text
PROMPT117_ARTIFACT_VERIFIED=true
TASK_SCOPE=WPF_STARTUP_ROUTING_AND_LOCAL_STATE_ONLY
CURRENT_SCREEN=INSTALLATION_PROMPT117
CURRENT_PRODUCT_ROOT=E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
EXPECTED_DB=obm_pos_dev_v0_pg
TBLTENANTPOSDEVICE_CHANGES=FORBIDDEN
API_DB_MUTATION=FORBIDDEN
WPF_DB_RESET=FORBIDDEN
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
MANUAL_POS1_TEST_READY=false
```

## Phase 1 — Prove the exact normal launch lane

Build the latest canonical WPF source under:

```text
E:\Project2026\4POS\NailSalonNet8
```

Update the visible build label to:

```text
prompt118
```

Record:

```text
STARTUP_PROJECT=<exact project>
STARTUP_EXECUTABLE=<exact path>
EXECUTABLE_SHA256=<hash>
VISUAL_STUDIO_PROFILE=<exact profile>
ENVIRONMENT=<exact value>
PRODUCT_ROOT_KEY=<exact key>
PRODUCT_ROOT_SOURCE=<launch profile/env/protected config/etc.>
PRODUCT_ROOT_PRECEDENCE=<ordered sources>
RESOLVED_PRODUCT_ROOT=<exact path>
EXPECTED_NORMAL_PRODUCT_ROOT=<exact proven path>
NORMAL_LAUNCH_USING_INSTALLATION_TEST_ROOT=yes/no
```

Do not infer the root purpose from its name alone. Prove it from call sites, configuration, accepted artifacts, and runtime state.

If normal Visual Studio startup is unintentionally pinned to an installation-test root, correct the existing profile/configuration source to the already established canonical normal-development root. Do not copy protected state blindly and do not create a new root.

Add a regression guard so a normal Development profile cannot silently select a root classified as installation-test-only.

## Phase 2 — Build the complete startup state truth table

For the selected root and canonical WPF DB, record exact predicate values and owning methods:

```text
PHASE1_PROTECTED_STATE_PRESENT
PHASE1_STATE_DECRYPTABLE
PHASE1_CONTRACT_VALID
PHASE1_COMPLETION_MARKER
BOOTSTRAP_CREDENTIAL_PRESENT
BOOTSTRAP_CREDENTIAL_EXPIRED
REMOTE_BOOTSTRAP_REACHABLE
PHASE2_DB_PRESENT
PHASE2_PROVIDER_NPGSQL
PHASE2_DB_NAME_MATCH
PHASE2_MIGRATION_HISTORY_READABLE
PHASE2_PENDING_MIGRATIONS
PHASE2_REQUIRED_BASELINE_PRESENT
PHASE2_COMPLETION_MARKER
LOCAL_RUNTIME_ACTIVATION_PRESENT
LOCAL_STATION_IDENTITY_VALID
MAINWINDOW_ELIGIBLE
INSTALLATION_ELIGIBLE
OPEN_OBM_POS_BUTTON_ENABLED
```

For each predicate include:

```text
class/method
source of truth
actual value
why that value should or should not gate local startup
```

Reconcile report107's accepted migration-current WPF DB evidence with prompt117's `schema migration required` assessment. Identify whether the difference is:

```text
different DB
wrong ProductRoot binding
migration chain mismatch
missing minimal baseline/activation marker rather than schema
stale assessment cache
wrong role/credential
another exact defect
```

Do not use a generic `Phase2 incomplete` classification.

## Phase 3 — Classify the exact routing defect

Choose exactly one primary classification:

```text
R1_NORMAL_PROFILE_SELECTED_WRONG_PRODUCT_ROOT
R2_VALID_INSTALLED_LOCAL_STATE_REMOTE_PHASE1_RESUME_WRONGLY_GATES_MAINWINDOW
R3_DB_MIGRATION_CURRENT_BUT_BASELINE_OR_ACTIVATION_MARKER_DRIFT
R4_PHASE2_GENUINELY_INCOMPLETE_AFTER_AUTHORIZED_DEV_DB_RESET
R5_PHASE1_STATE_GENUINELY_INCOMPLETE_OR_CORRUPT
R6_MAINWINDOW_ELIGIBILITY_PREDICATE_DEFECT
R7_OTHER_EXACTLY_PROVEN_STARTUP_STATE_DEFECT
```

State whether the current operator environment is truly installed or truly incomplete. Provide direct evidence; do not guess from UI labels.

## Phase 4 — Correct the smallest canonical defect

Allowed outcomes depend on the proven classification.

### If R1 — wrong ProductRoot/profile

```text
repair only the existing normal Development profile/resolver
select the established canonical root
preserve installation-test profiles as explicit test-only profiles when still needed
prevent implicit fallback to the Phase1 test root
```

Do not merge/copy protected files between roots unless an existing canonical state-migration service explicitly owns that operation.

### If R2 or R6 — local installed state incorrectly gated by remote/API state

Enforce:

```text
valid protected local activation + valid canonical DB => MainWindow
remote bootstrap/API validation runs as nonfatal background/degraded-cloud state
expired/unavailable remote credential does not downgrade completed installation
```

Do not weaken initial installation authorization. This rule applies only after local Phase2/activation completion is physically proven.

### If R3 — physical DB and completion metadata drift

Use the existing canonical Phase2 reconciliation/completion boundary only after proving all required DB schema, baseline, and activation facts. Do not write a completion marker directly and do not declare complete from migration count alone.

### If R4 — Phase2 genuinely incomplete

Use only the existing canonical `Install Local Database Baseline`/Phase2 service to complete:

```text
migration/schema when required
minimal baseline seed
completion/activation state
```

Requirements:

```text
one canonical transaction/checkpoint contract
no DB reset
no manual SQL/table creation
no speculative business seed
no TblTenantPosDevice work
no requirement that API remain online after local activation is established
```

If valid Phase1 state is required and unavailable, stop with an exact blocker instead of fabricating it.

### If R5 — Phase1 genuinely incomplete/corrupt

Keep InstallationV0 open with exact recoverable state. Do not open MainWindow and do not automatically redeem/re-pair. Return the precise operator action or protected-state repair boundary required.

## Phase 5 — Physical normal-startup acceptance

Launch the actual normal WPF Development executable/profile with label `prompt118`.

### Required case A — current state

```text
correct canonical ProductRoot selected
state classification displayed/logged correctly
process remains alive
no unhandled exception
no uncontrolled installation loop
```

### Required case B — installed-local state, when physically valid or correctly completed in this task

Prove:

```text
normal launch opens MainWindow directly
InstallationV0 does not flash/open first
Open OBM-POS manual bridge is not required for ordinary startup
canonical DB = obm_pos_dev_v0_pg
provider = Npgsql
pending migrations = 0
required local baseline/activation state valid
API endpoint intentionally unavailable or stopped
MainWindow remains alive and responsive for at least 60 seconds
local API/sync status may show degraded/offline without closing or rerouting
close and restart once
second launch again opens MainWindow directly
```

Do not claim this case from a test harness or fabricated marker.

### Required case C — incomplete state, when genuinely applicable

```text
InstallationV0 remains open
exact missing phase/action is shown
no generic Local POS/API Unknown when a more precise local state is known
no process crash
no automatic state downgrade loop
retry/corrective action remains available
```

## Phase 6 — Focused tests and build

Run focused tests for:

```text
normal profile -> canonical ProductRoot selection
normal profile cannot silently use installation-test root
startup state truth table
Phase2-complete local state opens MainWindow without API
expired/unavailable WpfJwt after local completion does not reopen installation
Phase2-incomplete state remains InstallationV0
DB/current-marker reconciliation
Open OBM-POS enablement predicate
restart stability
prompt117 HttpRequestException regression
Firebase/email-password remains absent
TblTenantPosDevice unchanged
```

Expected:

```text
all pass
0 skipped
WPF build errors=0
```

Physical behavior overrides build/test results.

## End state

PASS requires:

```text
normal WPF Development startup uses the one canonical ProductRoot/profile
current state is classified from physical local facts, not remote availability
when installed-local state is valid, MainWindow opens directly and survives API outage
when installation is incomplete, InstallationV0 is precise and recoverable
no uncontrolled re-entry into installation
no crash
TblTenantPosDevice remains unchanged
Category Weight and Booking Weight remain unchanged
MANUAL_POS1_TEST_READY=false
OPERATOR_MAINWINDOW_SCREENSHOT_READY=true when MainWindow physical proof passes
```

This task does not authorize manual sync testing. It only restores correct WPF startup. After PASS, the operator may provide the MainWindow/POS UI screenshot requested for the later POS-device writer/routing design.

## Required private artifact

Preserve prior artifacts unchanged. Create:

```text
E:\Project2026\RecoveryReports\WpfCanonicalInstalledLocalStartupV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ARTIFACT_VERIFICATION.md
OPERATOR_SCREENSHOT_INTAKE.md
NORMAL_LAUNCH_PROFILE.md
PRODUCT_ROOT_PRECEDENCE.md
PRODUCT_ROOT_INVENTORY.md
STARTUP_STATE_TRUTH_TABLE.md
REPORT107_REPORT117_STATE_RECONCILIATION.md
STARTUP_ROUTING_CLASSIFICATION.md
MAINWINDOW_ELIGIBILITY_BEFORE.md
MAINWINDOW_ELIGIBILITY_AFTER.md
PHASE2_RECONCILIATION_OR_COMPLETION_PROOF.md
API_OFFLINE_LOCAL_FIRST_PROOF.md
PHYSICAL_STARTUP_PROOF.md
RESTART_PROOF.md
FOCUSED_TEST_OUTPUT.txt
FINAL_STATE.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

If Phase2 is not changed, record `NOT_APPLICABLE` with exact reason in `PHASE2_RECONCILIATION_OR_COMPLETION_PROOF.md`.

## Public report

Create and push only:

```text
report/report118.md
```

Include:

```text
Verdict
Prompt117 artifact SHA verified yes/no
Latest WPF label observed
Normal startup project/executable/profile
Resolved ProductRoot
Resolved ProductRoot classification
Normal profile used installation-test root before/after yes/no
Canonical WPF DB proof yes/no
WPF pending migrations count
Required baseline present yes/no
Phase1 completion state
Phase2 completion state
Local runtime activation state
Remote API/bootstrap reachable yes/no
Current startup routing classification R1-R7
Startup source/config corrected yes/no
Phase2 reconciliation/completion performed yes/no
Manual completion marker write used yes/no
MainWindow physically opened yes/no
InstallationV0 opened before MainWindow yes/no
MainWindow API-offline survival duration
Second-launch MainWindow proof yes/no
Incomplete-installation recoverable proof yes/no/not-applicable
Uncontrolled Installation re-entry remaining yes/no
Focused test totals
WPF build totals
TblTenantPosDevice changed yes/no
WPF DB reset performed yes/no
API DB mutated yes/no
Category Weight changed yes/no
Booking Weight changed yes/no
Manual POS1 test ready false
Operator MainWindow screenshot ready true/false
Production/customer/reference DB mutated yes/no
Private artifact yes/no
Aggregate SHA-256
```

Do not expose protected state contents, tokens, passwords, complete connection strings, raw identities, or business data.

## Verdicts

PASS when normal installed-local startup is physically correct:

```text
OBM_WPF_CANONICAL_INSTALLED_LOCAL_STARTUP_MAINWINDOW_STABLE_READY_FOR_OPERATOR_SCREENSHOT
```

Narrow blockers only:

```text
BLOCKED_WPF_NORMAL_PROFILE_PRODUCT_ROOT
BLOCKED_WPF_CANONICAL_PRODUCT_ROOT_STATE
BLOCKED_WPF_PHASE1_PROTECTED_STATE
BLOCKED_WPF_PHASE2_BASELINE_OR_ACTIVATION
BLOCKED_WPF_MAINWINDOW_ELIGIBILITY
BLOCKED_WPF_LOCAL_FIRST_API_GATE
BLOCKED_WPF_STARTUP_ROUTING_PHYSICAL_PROOF
BLOCKED_WPF_STARTUP_ROUTING_TESTS
```

Every blocked result must include:

```text
exact profile/ProductRoot
exact failed predicate and owning method
physical DB/baseline/activation state
whether API availability incorrectly influenced the decision
whether any state/DB was mutated
MANUAL_POS1_TEST_READY=false
OPERATOR_MAINWINDOW_SCREENSHOT_READY=false
```

Do not return a generic installation blocker.
