# Prompt 040 — Handoff from completed InstallationV0 to the OBM-POS main workflow

## Operator-observed issue

The operator completed Phase 1 and Phase 2 v002 successfully. Restart hydration now shows:

```text
Build label: prompt039
Phase 2 Local DB Baseline: Phase 2 v002 Complete
Target DB: obm_pos_dev_v0_pg
RuntimeState: Activated
```

However, starting the current executable still opens `InstallationV0Window` instead of entering the normal OBM-POS application workflow/MainWindow.

The operator asks how to start the actual POS.

## Immediate distinction to verify

There are two possible situations:

### A. Operator is launching the standalone InstallationV0 project

If Visual Studio startup project is:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj
```

then opening `InstallationV0Window` is expected. This project is the installation harness, not the normal POS shell.

The normal POS project is:

```text
E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj
```

### B. Operator launches NailSalonNet8 but its startup router still sends the process to InstallationV0Window

If this occurs, the startup handoff is incomplete and must be fixed in source.

Prompt040 must investigate and close both cases. Do not assume either case without proving the actual startup project/process path and startup routing.

## Authoritative completed installation state

Read completely:

```text
report/report037.md
report/report038.md
report/report039.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The durable completed state is:

```text
Phase 1 ApiAuthorized checkpoint: valid
Phase 2 v002 marker: present exactly once in dbo.Phase2TrialCompletionMarker
Identity spine: verified
TblPosRuntimeProfile: one row, RuntimeState Activated
TblEmployeePermission: 7
TblEmployee: 20
Target DB: obm_pos_dev_v0_pg
Environment: Development
```

The normal POS startup must treat this state as installed and ready, not as installation-required.

## Objective

Implement and physically prove the canonical startup decision:

```text
Start NailSalonNet8
        ↓
Read ProductRoot and database locator
        ↓
Run startup assessment
        ↓
If installation is complete and healthy
    → open normal OBM-POS MainWindow
If installation is incomplete or blocked
    → open InstallationV0Window
```

InstallationV0 must not remain the permanent application shell after v002 completion.

## Startup source audit

Audit the real startup path in the main WPF project, including as applicable:

```text
App.xaml
App.xaml.cs
OnStartup / Startup event
startup bootstrapper
DatabaseStartupAssessmentService
RuntimeProfileStartupAssessmentService
Bootstrap/Recovery router
InstallationV0Window launch sites
MainWindow launch sites
ShutdownMode
single-instance/process handoff logic
```

Report the exact current decision tree and the precise reason the operator currently sees InstallationV0Window.

## Canonical readiness predicate

Do not use only one marker or only one runtime-state field.

The main POS may open only after a read-only readiness assessment proves:

```text
1. Database locator resolves exactly to obm_pos_dev_v0_pg in Development.
2. Expected schema/migrations are present.
3. Durable Phase 1 checkpoint exists and identity is readable.
4. dbo.Phase2TrialCompletionMarker contains v002 exactly once for current TenantGuid/PosGuid.
5. TblTenant current identity matches Phase 1.
6. TblPosLocal current identity matches Phase 1 POS/slot.
7. TblPosRuntimeProfile identity matches the Phase 1 identity spine.
8. TblPosRuntimeProfile.RuntimeState = Activated.
9. Required v002 baseline invariants exist, including 7 permissions and 20 employee starter rows for this current trial.
10. No RecoveryRequired/Disabled/blocking startup condition exists.
```

When these hold, classify startup as the existing canonical equivalent of:

```text
InstalledHealthy
```

Remember: `InstalledHealthy` is an assessment result, not a `PosRuntimeState` enum value.

## Main startup behavior

### Installed/healthy

When readiness passes:

```text
create/show normal OBM-POS MainWindow
set MainWindow as Application.Current.MainWindow
close/do not create InstallationV0Window
continue existing POS initialization path
```

Do not run seed, migrations, Pairing Code redeem, or Phase 2 mutation during normal startup.

### Installation incomplete

When Phase 1/Phase 2 is genuinely incomplete:

```text
show InstallationV0Window
show exact safe reason
allow the approved explicit installation actions
```

### Installed but inconsistent

When marker exists but invariants fail:

```text
do not silently open POS
do not rerun seed automatically
show InstallationV0Window or recovery UI in verify/blocked mode
show a safe readiness result code
```

Examples:

```text
INSTALLATION_COMPLETE_INVARIANT_MISMATCH
RUNTIME_PROFILE_RECOVERY_REQUIRED
IDENTITY_SPINE_MISMATCH
SCHEMA_NOT_READY
```

## InstallationV0 completed-state UI

When InstallationV0 is launched directly for diagnostics and v002 is complete:

- preserve `Phase 2 v002 Complete` hydration;
- keep mutating install action disabled;
- provide a clear non-mutating action:

```text
Open OBM-POS
```

This button should launch or hand off to the normal POS workflow only after rerunning the same readiness assessment.

Preferred behavior inside the same process:

```text
show MainWindow
set Application.Current.MainWindow = MainWindow
close InstallationV0Window
```

If architecture requires restarting the main executable, use an explicit safe process handoff and avoid duplicate WPF processes. Do not invent a process restart if same-process navigation is supported.

## Visual Studio operator path

Prompt040 report must state exact operator steps for immediate manual testing:

```text
1. Stop debugging.
2. In Visual Studio, right-click NailSalonNet8 project.
3. Set as Startup Project.
4. Ensure Development profile/ProductRoot/database locator point to the completed POS1 lane.
5. Press F5.
```

Also prove the launched process binary path and project/output assembly so stale `InstallationV0.exe` cannot be mistaken for the main POS executable.

## ProductRoot and database-locator consistency

The completed installation ProductRoot is:

```text
E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
```

Audit how `NailSalonNet8` receives or resolves ProductRoot and the DB locator.

Do not silently switch the main app to another ProductRoot or another database such as an older Development profile.

At startup, prove sanitized values:

```text
ProductRoot classification
DatabaseName = obm_pos_dev_v0_pg
EnvironmentName = Development
TenantCode = OBMDEVV0
PosName = POS1
RuntimeState = Activated
```

No passwords/connection strings/secrets in logs or report.

## Preserve completed database state

Prompt040 must not:

```text
rerun Phase 2 seed
insert duplicate outbox
rewrite markers
change employees/permissions
change runtime history
redeem another Pairing Code
modify reference DB
```

Startup/readiness work should be read-only except normal in-memory application initialization.

## Physical acceptance

Codex may run builds/tests and may launch the main WPF only if it can do so without interfering with the operator's active UI session. Otherwise leave physical launch to the operator.

Required operator acceptance:

1. Start `NailSalonNet8` rather than the standalone InstallationV0 harness.
2. Main POS workflow opens, not `InstallationV0Window`.
3. The normal POS title/main shell is visible.
4. DB is `obm_pos_dev_v0_pg` Development.
5. Tenant/POS identity is `OBMDEVV0 / POS1`.
6. Management employee UI can load the 20 starter employees.
7. Checkout/staff UI uses existing Staff filtering.
8. No database seed/migration/outbox delta occurs merely from opening POS.
9. Closing and reopening POS repeats the same healthy route.

## Tests

Add focused tests for:

```text
completed v002 readiness opens MainWindow
incomplete Phase 1 opens InstallationV0Window
v001-only state opens InstallationV0Window
v002 marker plus invariant mismatch fails closed
RuntimeState Activated contributes to InstalledHealthy
RecoveryRequired/Disabled does not open MainWindow
main startup never mutates seed/outbox/marker
InstallationV0 direct diagnostics can hand off via Open OBM-POS
no duplicate WPF windows/processes
correct ProductRoot/database lane retained
prompt040 label where visible in InstallationV0 diagnostics
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap"
```

If the broader filter is invalid for the current test runner, run the InstallationV0 focused suite plus the exact startup/bootstrap test classes discovered in source.

## Build label

If InstallationV0 source changes, set:

```text
Build label: prompt040
Window title: OBM InstallationV0 Phase 1/2 - prompt040
```

The normal POS MainWindow does not need to display the coordination prompt label unless a current debug banner already exists. Do not add intrusive production UI solely for coordination.

## Report 040

Create and push:

```text
report/report040.md
```

Required sections:

1. Verdict.
2. Exact executable/project the operator had been launching.
3. Main WPF startup decision tree before correction.
4. Root cause of persistent InstallationV0Window.
5. InstalledHealthy/readiness predicate.
6. ProductRoot/database-locator proof.
7. MainWindow handoff implementation.
8. InstallationV0 `Open OBM-POS` diagnostic handoff behavior.
9. Fail-closed incomplete/inconsistent behavior.
10. Exact source files changed.
11. Build/test commands and counts.
12. No DB mutation/outbox/marker delta proof.
13. Physical launch evidence or exact operator steps.
14. Main POS employee/checkout smoke-test steps.
15. Active InstallationV0 label proof.
16. No secrets/no source push confirmation.
17. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_MAIN_STARTUP_HANDOFF_READY_FOR_USER_TEST
```

```text
OBM_POS_MAIN_STARTUP_PHYSICAL_PASS
```

```text
BLOCKED_OBM_POS_MAIN_STARTUP_HANDOFF
```
