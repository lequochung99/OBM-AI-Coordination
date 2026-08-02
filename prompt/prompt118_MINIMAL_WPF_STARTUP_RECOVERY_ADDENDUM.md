# Prompt 118 Addendum — Minimal WPF startup recovery only; stop all feature and architecture work

This addendum is binding and overrides any broader or conflicting interpretation in `prompt/prompt118.md`.

## Authoritative operator correction

The current WPF is not usable. The operator cannot reach the normal OBM-POS UI. The observed startup either exits/crashes or falls into InstallationV0 and never reaches MainWindow. Treat both as a failed startup gate.

The direct operator observation overrides the limited prompt117 20-second observation. Do not claim the crash/startup problem is closed until the exact normal Visual Studio launch used by the operator physically opens MainWindow and remains stable.

The project has spent excessive time outside the requested Turn Settings work. This task must be surgical. Do not continue architecture exploration, device-routing design, database-role work, sync E2E, or any Weight implementation.

## Frozen work

Do not inspect, design, implement, or modify in this task:

```text
Service Category Weight
Booking Weight
Price Weight business/save semantics
TblTenantPosDevice writer/schema/migration
TblPosLocal/TblTenantPosDevice routing
CompanionApp or payment-terminal modeling
API destination routing
API grouped-sync happy path
canonical Provider behavior
POS1-POS10 PlatformAppV0 UI or Pairing Code behavior
API database schema, grants, roles, migrations, or data
```

Do not create a new startup coordinator, ProductRoot, installation module, state model, DbContext, provider, or fallback path.

## Exact goal

Restore the already existing normal WPF Development startup path with the smallest regression repair:

```text
normal Visual Studio WPF launch
-> canonical existing ProductRoot and obm_pos_dev_v0_pg
-> exact local installation/runtime state evaluated
-> MainWindow opens when that local installed state is valid
-> API/SignalR/sync outage is nonfatal
```

If the current process truly exits, capture and fix the exact unhandled exception first. If the process remains alive but is routed to InstallationV0, classify that as a startup-routing failure and fix only the exact wrong predicate/profile/state source.

Do not debate whether the symptom is called a crash. Reproduce and record the physical behavior.

## Minimal execution sequence

### 1. Reproduce the operator launch exactly

Use the exact startup project, executable, Visual Studio profile, environment variables, working directory, and ProductRoot selected when the operator presses Start/Debug.

Set the visible build label to:

```text
prompt118
```

Run under the debugger with break-on-thrown enabled for CLR exceptions. Capture all of these owning-boundary signals:

```text
Application.DispatcherUnhandledException
AppDomain.CurrentDomain.UnhandledException
TaskScheduler.UnobservedTaskException
process exit code
Windows/.NET application event
```

Observe for at least 60 seconds or until the exact operator failure occurs.

Return exactly one physical classification:

```text
C1_PROCESS_EXITED_UNHANDLED_EXCEPTION
C2_WINDOW_CLOSED_OR_SHUTDOWN_WITHOUT_MAINWINDOW
C3_PROCESS_ALIVE_WRONG_INSTALLATION_ROUTE
C4_OTHER_EXACTLY_PROVEN_STARTUP_FAILURE
```

Required direct evidence:

```text
exact exception type/message/inner chain when applicable
exact class/method/line
exact shutdown/close caller when applicable
exact startup predicate values
exact selected ProductRoot and its configuration source
exact selected safe DB name/provider
last successful boundary
first failing boundary
```

Do not continue from a generic `Phase2 incomplete`, `Unknown`, or `installation state invalid` label.

### 2. Compare only the direct startup regression

Compare the current startup/profile/state-selection code with the last local source/evidence that physically opened MainWindow before the recent Price Weight/sync/reset work.

Use local git history/diff and accepted artifacts without `git reset --hard`, `git clean`, or reverting unrelated edits.

Inspect only the direct call chain needed to answer:

```text
Why does the operator's normal launch not open MainWindow now?
```

Do not perform a general architecture audit.

### 3. Apply the smallest existing-boundary correction

Allowed correction examples, only when directly proven:

```text
normal profile accidentally selects an InstallationV0 test ProductRoot
startup uses a stale/wrong state source before the canonical local state
API/bootstrap exception still escapes another owning async boundary
Shutdown/Close executes after a recoverable remote failure
MainWindow eligibility incorrectly depends on API/remote resume after local activation
an authorized Development DB reset removed required Phase2 baseline/activation state and the existing canonical Phase2 service must reconcile it
```

Rules:

```text
reuse the existing normal startup path
reuse the existing ProductRoot resolver
reuse the existing Phase2 completion/reconciliation service when genuinely required
no manual completion-marker write
no protected-state copying between roots
no WpfJwt bypass
no Firebase fallback
no EnsureCreated
no DB reset/drop/recreate
no speculative seed
no opening MainWindow blindly when local state is truly incomplete
```

Keep production changes inside the exact startup/exception/state-selection call chain. Any broader change is out of scope.

### 4. Physical acceptance and stop

PASS requires all of the following on the operator-equivalent normal Visual Studio launch:

```text
visible label = prompt118
process exits/crashes = no
unhandled exception = no
canonical existing ProductRoot selected
canonical WPF DB = obm_pos_dev_v0_pg
provider = Npgsql/PostgreSQL
MainWindow opens directly for valid installed-local state
InstallationV0 does not flash or replace MainWindow on that normal installed-local launch
API intentionally unavailable does not close or reroute MainWindow
MainWindow remains alive and responsive for at least 60 seconds
close normally and launch a second time
second launch again opens MainWindow and remains stable
```

If direct evidence proves the local installation is genuinely incomplete, do not fabricate completion. Return one narrow blocker naming the exact missing existing Phase1/Phase2 boundary, but the process must remain alive with a precise recoverable state.

After acceptance, stop. Do not resume TblTenantPosDevice, routing, sync happy path, Category Weight, or Booking Weight inside this task.

## Hard status locks

Until the physical MainWindow proof passes:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=false
MANUAL_POS1_TEST_READY=false
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
```

PASS may set only:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=true
MANUAL_POS1_TEST_READY=false
```

## Evidence and report

Preserve previous artifacts. Add to the prompt118 artifact:

```text
OPERATOR_EQUIVALENT_LAUNCH.md
PHYSICAL_FAILURE_CLASSIFICATION.md
EXACT_EXCEPTION_OR_ROUTING_BOUNDARY.md
LAST_WORKING_MAINWINDOW_COMPARISON.md
MINIMAL_STARTUP_DIFF.patch
MAINWINDOW_60_SECOND_PROOF.md
SECOND_LAUNCH_PROOF.md
```

The public `report/report118.md` must include:

```text
Verdict
Operator-equivalent launch proven yes/no
Physical classification C1/C2/C3/C4
Exact failed class/method/line
Exact selected ProductRoot and source
Canonical WPF DB proof yes/no
Minimal root cause
Production files changed count and paths
Process crash/exit after fix yes/no
MainWindow opens directly yes/no
InstallationV0 shown on normal installed-local launch yes/no
API-offline MainWindow proof yes/no
60-second physical stability yes/no
Second-launch MainWindow proof yes/no
WPF build/test totals
TblTenantPosDevice changed no
API/schema/sync changed no
Category Weight changed no
Booking Weight changed no
Operator MainWindow screenshot ready true/false
Manual POS1 test ready false
Private artifact and aggregate SHA-256
```

PASS verdict:

```text
OBM_WPF_NORMAL_MAINWINDOW_STARTUP_PHYSICALLY_RESTORED_READY_FOR_OPERATOR_SCREENSHOT
```

Narrow blockers only:

```text
BLOCKED_WPF_STARTUP_UNHANDLED_EXCEPTION
BLOCKED_WPF_STARTUP_SHUTDOWN_WITHOUT_MAINWINDOW
BLOCKED_WPF_STARTUP_WRONG_INSTALLATION_ROUTE
BLOCKED_WPF_CANONICAL_PRODUCTROOT_RESOLUTION
BLOCKED_WPF_PHASE1_STATE_RECOVERY
BLOCKED_WPF_PHASE2_EXISTING_COMPLETION_BOUNDARY
BLOCKED_WPF_MAINWINDOW_PHYSICAL_PROOF
```

Do not return another generic installation/startup blocker.