# Prompt 050 — Decouple completed local POS startup from expired Phase 1 API credential

## Physical operator evidence

Read completely before changing source:

```text
report/report045.md
report/report049.md
report/report039.md
report/report044.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The operator physically rebuilt and ran build `prompt049`.

The current InstallationV0 diagnostics screen shows:

```text
Build label: prompt049
Target DB: obm_pos_dev_v0_pg (Development/Test)
ProductRoot: E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
Protected hello failed: WPF_HELLO_HTTP_401
Phase 2 Local DB Baseline: blocked until Phase 1 passes
Open OBM-POS state: Idle
Open OBM-POS button disabled
Redeem Pairing Code enabled
```

The same database was previously physically verified as:

```text
TblPosRuntimeProfile count = 1
RuntimeState = Activated
Tenant/POS identity consistent
Phase 2 v002 marker present
Permissions = 7
Employees = 20
Local DB = obm_pos_dev_v0_pg
```

This evidence means the Phase 1 protected API credential has expired or is otherwise unauthorized, but the local POS database is already installed and Activated.

## Operator decision — authoritative runtime rule

The normal POS startup decision comes from local PostgreSQL status, not from current API token validity.

Canonical rule:

```text
Local database config resolves
+ PostgreSQL authentication succeeds
+ schema is locally usable
+ TblPosRuntimeProfile.RuntimeState = Activated
+ local Tenant/POS/profile identity is internally consistent
=> MainWindow may open
```

API state is independent:

```text
API token valid / API reachable
=> Online

API token expired / HTTP 401 / API unavailable
=> OfflineDeferred or ReauthorizationRequired
=> MainWindow still opens locally
```

An expired Phase 1 `WpfJwt` must not:

```text
- mark Phase 2 as not installed;
- disable Open OBM-POS when local runtime is Activated;
- request Pairing Code before local MainWindow can open;
- change RuntimeState;
- rerun seed;
- invalidate local PostgreSQL CRUD;
- erase or ignore the durable Phase 2 marker.
```

Pairing Code may remain available as an explicit API/cloud reauthorization action, but it is not a prerequisite for ordinary local POS startup.

## Root-cause objective

Find and correct the exact UI/startup dependency that currently performs this incorrect sequence:

```text
Phase 1 protected hello / bootstrap verification
-> HTTP 401
-> Phase1Pass = false
-> skip or overwrite Phase2StartupHydration
-> show "blocked until Phase 1 passes"
-> disable Open OBM-POS
```

The required sequence is:

```text
1. Resolve ProductRoot and local DB config.
2. Hydrate local Phase 2/runtime status read-only and independently.
3. Determine local POS route from TblPosRuntimeProfile and local identity.
4. Probe API credential separately.
5. Render local installation/runtime state and API connectivity state as separate dimensions.
```

Do not merely suppress the 401 text. Correct the dependency graph.

## Two independent state dimensions

Implement or expose two independent status models.

### Local POS status

At minimum:

```text
LocalDatabaseConfigResolved
LocalDatabaseAuthenticationSucceeded
SchemaReady
RuntimeProfileCount
RuntimeState
TenantIdentityConsistent
PosIdentityConsistent
LocalPosReady
LocalPosResultCode
```

### API/cloud status

At minimum:

```text
ProtectedCredentialPresent
ProtectedCredentialReadable
AccessCredentialAccepted
ApiReachable
ApiStatus
ApiResultCode
```

Suggested API states:

```text
Online
OfflineDeferred
ReauthorizationRequired
Unknown
```

Do not introduce a refresh-token implementation in this prompt. The current API token gap remains deferred.

## InstallationV0 diagnostics behavior

### Local POS complete, API HTTP 401

Required UI:

```text
Phase 2 Local DB Baseline: Phase 2 v002 Complete
Local POS status: Ready / Activated
API status: Reauthorization Required or OfflineDeferred
Open OBM-POS: enabled
Redeem Pairing Code: available as optional API reauthorization
```

The wording must not say:

```text
Phase 2 blocked until Phase 1 passes
Local database installation is not started
```

when the durable local database is already complete.

### Local POS incomplete

Only when the local DB status is genuinely incomplete may the diagnostics UI show installation-required state and disable Open OBM-POS.

### Local POS inconsistent/recovery

When runtime state is `RecoveryRequired`, `Disabled`, schema is unusable, or identity is inconsistent:

```text
Open OBM-POS disabled
show exact local result code
no automatic seed/redeem
```

## Open OBM-POS behavior

The button must call the prompt049 canonical structured router regardless of Phase 1 API status.

For the current physical state the expected route is:

```text
RouteDecision=OpenMainPos
RuntimeState=Activated
LocalRuntimeReady=True
MainWindowConstructed=True
MainWindowShown=True
MainWindowVisible=True
ApiStatus=ReauthorizationRequired or OfflineDeferred
```

Success result remains:

```text
OPEN_POS_MAINWINDOW_SHOWN
```

The diagnostics window closes only after MainWindow is visible.

API/bootstrap/sync failures after MainWindow is shown remain non-blocking.

## Direct runtime profile behavior

Verify the primary route independently:

```text
NailSalonNet8 + OBM-POS Runtime Development
-> local DB-first assessment
-> RuntimeState Activated
-> MainWindow opens
-> expired/missing WpfJwt does not open InstallationV0
-> API status becomes OfflineDeferred/ReauthorizationRequired
```

Normal runtime must not call protected hello as a prerequisite before showing MainWindow.

## Preserve installation semantics

Phase 1 remains required for an initial clean installation to materialize the authorized Tenant/POS identity.

Phase 1 proof may remain required for:

```text
- initial Pairing Code redeem;
- initial identity materialization;
- installation diagnostics/audit;
- API/cloud reauthorization.
```

But after durable local installation is complete and runtime state is Activated, current API token expiry is not equivalent to installation incompleteness.

Do not delete Phase 1 checkpoint or protected credential in this prompt.

## Phase 2 hydration source of truth

Use the existing durable local sources read-only, including as applicable:

```text
dbo.Phase2TrialCompletionMarker
dbo.TblPosRuntimeProfile
dbo.TblTenant
dbo.TblPosLocal
```

`Phase2StartupHydrationService` must be callable even when protected API verification returns 401.

Hydration ordering must not overwrite a locally complete status with a Phase1-failed placeholder afterward.

Audit every assignment to:

```text
Phase2 status text
Install/Verify button enabled state
Open OBM-POS enabled state
Phase1 pass flag
startup/hydration result
```

Identify and remove last-writer-wins behavior that reverts Phase2 complete to blocked.

## No-mutation requirements

This prompt must not:

```text
- redeem a Pairing Code automatically;
- refresh or rotate tokens;
- mutate PostgreSQL;
- rerun Phase 2 seed;
- rewrite markers;
- update runtime state/history;
- update employees, permissions, PINs, or outbox;
- change PostgreSQL roles/passwords;
- set User/Machine environment variables.
```

Startup and diagnostics checks must be read-only.

## Source audit

Inspect and correct the exact paths, including as applicable:

```text
InstallationV0Window initialization/resume
Phase1InstallationService resume/protected hello
Phase2StartupHydrationService
InstallationV0Module Open OBM-POS callback
PosStartupRouteResult
App.OpenInstalledPosFromInstallationV0Async
App.StartNormalApplicationAsync
RuntimeProfileStartupAssessmentService
button enable/disable calculations
status text render methods
```

## Tests

Add focused tests for at least:

```text
Phase2 complete + RuntimeState Activated + protected hello 401
-> Phase2 still displays Complete

Phase2 complete + protected hello 401
-> Open OBM-POS enabled

Phase2 complete + protected hello 401
-> canonical router returns OpenMainPos

MainWindow visible + API 401
-> route success remains success

API 401
-> API status ReauthorizationRequired/OfflineDeferred

API 401
-> no Pairing Code auto-redeem

API 401
-> no seed/marker/outbox/runtime-state mutation

local DB incomplete + API valid
-> installation route remains required

local DB recovery state + API valid/invalid
-> recovery route

Phase2 hydration executes even when Phase1 API verification fails

late Phase1 failure render cannot overwrite local Phase2 complete status

direct Runtime Development with expired/missing API token
-> MainWindow route

prompt050 label
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap|FullyQualifiedName~OpenObmPos|FullyQualifiedName~RuntimeState|FullyQualifiedName~Offline" -v minimal
```

## Build label

Set:

```text
Build label: prompt050
Window title: OBM InstallationV0 Phase 1/2 - prompt050
```

## Physical execution policy

Do not physically redeem a new Pairing Code.

Do not mutate DB.

Do not automatically launch WPF if it may interfere with the operator session.

Leave final tests to the operator:

```text
Route A — prompt050 InstallationV0 with expired WpfJwt
          -> Phase2 Complete remains visible
          -> Open OBM-POS enabled
          -> MainWindow opens locally

Route B — Runtime Development with API unavailable/401
          -> MainWindow opens directly
          -> local mode OfflineDeferred/ReauthorizationRequired
```

## Report 050

Create and push:

```text
report/report050.md
```

Required sections:

1. Verdict.
2. Physical prompt049 HTTP 401 evidence.
3. Exact incorrect Phase1-to-Phase2 UI dependency.
4. Local POS status model.
5. API/cloud status model.
6. Corrected hydration ordering and source of truth.
7. Open OBM-POS enablement rule.
8. Direct runtime route behavior.
9. Initial installation versus installed-runtime distinction.
10. No-mutation proof.
11. Exact source files changed.
12. Build/test commands and counts.
13. Prompt050 label proof.
14. Exact operator retest steps.
15. Deferred refresh-token work.
16. No secrets/no DB mutation/no source push proof.
17. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_API_INDEPENDENT_LOCAL_STARTUP_READY_FOR_USER_RETEST
```

```text
BLOCKED_OBM_POS_API_INDEPENDENT_LOCAL_STARTUP
```
