# Prompt 057 — Wire the canonical local runtime end to end so OBM-POS can launch for real

## Operator decision

The documentation authority, naming cleanup, and legacy-shim deletion are complete through prompt056.

The operator now authorizes the next implementation step:

```text
Wire the final canonical runtime path so OBM-POS can launch as a real application.
```

This is not another architecture redesign.

Do not add another router, assessment layer, bootstrap abstraction, compatibility shim, context object, hidden security flag, application-password system, or API prerequisite.

Use the final names and contract already established by canonical V001.

## Phase 0 — mandatory documentation gate

Before editing implementation, test, project, launch-profile, configuration, or documentation files, read completely:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report055.md
report/report056.md
```

Also inspect the post-cleanup evidence:

```text
E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\README.md
E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\direct-runtime-path.mmd
E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\installation-handoff-path.mmd
E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\final-symbol-inventory.csv
E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\forbidden-name-scan.txt
E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\SHA256SUMS.txt
```

Before the first source edit, compute and record the current hashes. Expected prompt056 values are:

```text
CanonicalDocVersion=V001
CanonicalDocSha256=7044A02F29FE349FE531DE6800BA739E6B29EA473B9A867881B283DB8743BC72
AgentsSha256=90D09B8058381663A4EF317A5707256FA01B1DCDE5FE9F5BD3D4FA8F3CC10E8B
CurrentTaskSha256=093CA0C7D9040D52B94D065944A35DDE53A19C135090A4B42F993B9C5CBBC184
CurrentResultSha256=16D6B1DE20FED54988C51DEDEEED2F73086CDD62BD3487B8F1E30C355572A973
```

Required report evidence:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V001
CanonicalDocSha256=<actual current hash>
```

If the documents changed, read the new contents and verify that this wiring task remains authorized. If not, stop before source edits with:

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```

## Canonical behavior — non-negotiable

Normal runtime:

```text
Local PostgreSQL usable
-> show MainWindow
-> initialize API session afterward
```

API failure:

```text
HTTP 401 / expired token / API unavailable
-> MainWindow remains open
-> local PostgreSQL CRUD remains available
-> API is Offline or Reauthorization Required
```

Credentials:

```text
Database = obm_pos_dev_v0_pg
Normal runtime PostgreSQL username = hung
Password source = protected local PostgreSQL credential/passfile
```

Employee `TblEmployee.LoginNumber` is an operational PIN. It is not a startup credential and must not participate in this task.

Phase 1, Phase 2, protected hello, Pairing Code, seed counts, and identity-spine proof remain installation/verification concerns. They must not block an already usable local runtime.

## Objective

Produce one working runtime wiring path and one handoff path:

```text
Direct runtime:
App startup
-> resolve canonical local configuration
-> LocalPosStartupService
-> LocalPosStartupResult.Ready
-> MainWindowOpenResult success
-> MainWindow visible
-> ApiSessionInitializer asynchronously

Installation handoff:
InstallationV0 Open OBM-POS
-> same LocalPosStartupService
-> same MainWindow open method
-> MainWindow visible
-> ApiSessionInitializer asynchronously
```

There must be one owner of local runtime readiness and one owner of the WPF window transition.

## Phase 1 — audit the actual current wiring before changing it

Inspect at minimum:

```text
App.xaml
App.xaml.cs
NailSalonNet8.csproj
Properties\launchSettings.json
Services\Startup\LocalPosStartupService.cs
Services\Startup\LocalPosStartupResult.cs
ConnectService\IApiSessionInitializer.cs
ConnectService\ApiSessionInitializer.cs
InstallationV0\Application\MainWindowOpenResult.cs
InstallationV0\InstallationV0Module.cs
InstallationV0\Presentation\InstallationV0Window.cs
Services\Startup\InstallationV0VerificationService.cs
MainWindow.xaml
MainWindow.xaml.cs
all DI registration methods used by App
all ProductRoot/local DB configuration providers
all Npgsql connection construction paths used by normal runtime
```

Return the exact pre-change wiring table:

| Responsibility | Active type/method | DI lifetime/registration | Direct runtime caller | Installation handoff caller | Problem |
| --- | --- | --- | --- | --- | --- |

Do not rewrite code that is already correct.

## Phase 2 — service-provider and DI wiring

Ensure the real WPF service provider resolves exactly one active implementation for each runtime responsibility:

```text
LocalPosStartupService
IApiSessionInitializer -> ApiSessionInitializer
MainWindow
MainWindow dependencies/view models/services
InstallationV0Module when the installation profile is selected
InstallationV0VerificationService only for installation/diagnostics
```

Requirements:

- no duplicate registrations with different implementations;
- no registration under removed compatibility interfaces;
- no service-locator fallback that silently creates a second provider;
- no separate provider for direct runtime versus InstallationV0 handoff unless a framework requirement is proven;
- no missing MainWindow constructor dependency;
- choose lifetimes based on actual state ownership and document the reason;
- validate the provider during tests so missing/ambiguous registrations fail before physical launch.

Add a focused DI resolution test that builds the same production registration graph and resolves:

```text
LocalPosStartupService
IApiSessionInitializer
MainWindow
InstallationV0Module
```

without opening a window or connecting to PostgreSQL.

## Phase 3 — canonical local configuration wiring

Trace the exact active configuration chain:

```text
OBM-POS Runtime Development launch profile
-> ProductRoot/config source
-> database host/port/name/username
-> protected password/passfile source
-> Npgsql connection construction
-> LocalPosStartupService
```

Required sanitized result:

```text
DatabaseName=obm_pos_dev_v0_pg
DatabaseUsername=hung
PasswordSource=protected local credential/passfile
```

Do not print the password, passfile contents, or full connection string.

Rules:

- normal runtime must not silently fall back to `postgres`;
- `postgres` remains provisioning/migration/backup-only;
- do not set User/Machine environment variables;
- do not add a second configuration format;
- do not require API/token configuration to construct the local DB connection;
- keep Development safety checks limited to preventing the wrong ProductRoot/protected DB lane, not authenticating the application.

If the configured runtime role is not `hung`, correct only the source/config selection needed for normal runtime. Do not create users, alter roles, change passwords, or issue GRANT/REVOKE.

## Phase 4 — LocalPosStartupService wiring

`LocalPosStartupService` must own only the minimum local readiness decision:

```text
- local configuration can be resolved;
- PostgreSQL authentication succeeds;
- essential schema needed by MainWindow startup exists;
- local Tenant/POS context can be resolved consistently;
- installed local runtime state is suitable for normal operation.
```

It must not call or require:

```text
protected hello
WpfJwt/API token validation
Pairing Code
Phase1InstallationService
Phase2 exact employee/permission/outbox counts
employee operational PIN
InstallationV0VerificationService
SignalR/API reachability
```

Required practical result model:

```text
LocalPosStartupDecision.Ready
LocalPosStartupDecision.InstallationRequired
LocalPosStartupDecision.RecoveryRequired
LocalPosStartupDecision.Failed
```

For the current physical database and correct protected credential, the expected result is:

```text
Decision=Ready
DatabaseName=obm_pos_dev_v0_pg
DatabaseUsername=hung
LocalRuntimeReady=True
```

Use safe diagnostic metadata only; never include secrets.

## Phase 5 — direct runtime entrypoint

There must be one obvious direct runtime path in `App`:

```csharp
var localResult = await localPosStartupService.StartAsync(...);

if (localResult.Decision == LocalPosStartupDecision.Ready)
{
    var windowResult = await ShowMainWindowAsync(...);
    if (windowResult.MainWindowVisible)
    {
        StartApiSessionWithoutBlockingLocalPos();
    }
    return;
}

ShowInstallationOrRecovery(localResult);
```

The exact method names may match current source, but the ownership and ordering must be equivalent.

Remove or bypass any remaining branch that can route a locally ready database to InstallationV0 or recovery because of API/token state.

No API call may execute before the local MainWindow decision.

## Phase 6 — MainWindow transition wiring

Use one shared WPF dispatcher method for both direct runtime and InstallationV0 handoff.

Required lifecycle:

```text
1. run on Application dispatcher;
2. reuse/activate an existing visible MainWindow when appropriate;
3. preserve the previous MainWindow and ShutdownMode;
4. set ShutdownMode.OnExplicitShutdown during transition;
5. resolve exactly one MainWindow from the production provider;
6. set Application.Current.MainWindow;
7. call Show();
8. call Activate();
9. verify IsVisible=True;
10. set ShutdownMode.OnMainWindowClose;
11. return MainWindowOpenResult success;
12. close InstallationV0 only after visibility success.
```

Structured success must contain at least:

```text
ResultCode=OPEN_POS_MAINWINDOW_SHOWN
MainWindowConstructed=True
MainWindowShown=True
MainWindowVisible=True
LocalRuntimeReady=True
```

On constructor/show failure, restore the prior window/shutdown state and return the exact safe failure. Do not synthesize bool-wrapper errors.

## Phase 7 — InstallationV0 Open OBM-POS wiring

The button path must be:

```text
InstallationV0Window
-> InstallationV0Module
-> App handoff callback
-> same LocalPosStartupService
-> same MainWindow transition
```

It must not re-run:

```text
protected hello as a startup prerequisite
Pairing Code validation
Phase 1 token proof
Phase 2 exact-count verification
InstallationV0VerificationService as normal runtime auth
```

The InstallationV0 diagnostics may display API/installation status, but `Open OBM-POS` eligibility for an installed local runtime comes from `LocalPosStartupResult.Ready`.

## Phase 8 — post-MainWindow API session wiring

`IApiSessionInitializer.StartAsync()` must run only after MainWindow visibility is proven.

Required behavior:

```text
API token accepted -> Online
HTTP 401 / expired -> ReauthorizationRequired
API unreachable -> Offline
```

All three outcomes preserve MainWindow and local operation.

Requirements:

- exceptions are caught/logged safely;
- no modal startup error closes or hides MainWindow;
- no result rewrite changes `OPEN_POS_MAINWINDOW_SHOWN` after success;
- no automatic Pairing Code redemption;
- no token refresh implementation in this prompt;
- no secret/token logging.

## Phase 9 — recovery-only service boundary

Audit remaining names/classes such as:

```text
RuntimeBootstrapRepairService
RuntimeProfileShadowStartupEvaluator
RuntimeRecoveryIdentityValidator
RuntimeControlServices
RuntimeRolloutExecutionBridge
```

Do not perform broad cleanup in this task.

Only prove and enforce:

```text
- they do not participate in the normal Ready -> MainWindow path;
- recovery/updater services cannot override a successful LocalPosStartupResult.Ready;
- generic bootstrap wording is not exposed as a normal runtime result.
```

If `RuntimeBootstrapRepairService` is genuinely recovery-only, keep it for now and record its callers. If it is still on the normal startup path, detach it from that path using the smallest safe edit.

## Phase 10 — focused tests

Add/update tests for at least:

```text
production DI graph resolves final runtime services
no duplicate final runtime registrations
normal runtime uses LocalPosStartupService
InstallationV0 handoff uses the same LocalPosStartupService
local DB Ready -> MainWindow open path
API HTTP 401 does not change local Ready/MainWindow success
API unavailable does not change local Ready/MainWindow success
IApiSessionInitializer is invoked after MainWindow visibility proof
InstallationV0VerificationService has no normal-runtime caller/registration path
runtime configuration selects database obm_pos_dev_v0_pg and username hung
normal runtime has no fallback to postgres
no active pre-MainWindow API/protected-hello call
MainWindow transition restores previous state on failure
prompt057 build label
canonical naming guard still passes
```

Do not require a real PostgreSQL password in unit tests. Use abstractions/fakes around connection opening where needed, without adding a new runtime architecture layer.

## Phase 11 — build and static wiring verification

Run sequentially:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~MainWindow|FullyQualifiedName~Offline|FullyQualifiedName~Database|FullyQualifiedName~Naming|FullyQualifiedName~Documentation|FullyQualifiedName~DependencyInjection|FullyQualifiedName~Wiring" -v minimal
```

Run source checks proving:

```text
- one LocalPosStartupService production registration;
- one IApiSessionInitializer production registration;
- no removed compatibility names;
- no pre-MainWindow protected-hello/API token gate;
- direct runtime and InstallationV0 handoff converge on the same methods;
- runtime profile selects hung, not postgres.
```

## Phase 12 — controlled live startup smoke

The operator wants OBM-POS to be capable of running for real. After build/tests PASS, perform a controlled direct-runtime smoke only when it is safe.

Preconditions:

```text
- no existing NailSalonNet8/OBM-POS process is running;
- local PostgreSQL service is already running;
- current ProductRoot and protected runtime credential exist;
- no database migration/seed is required;
- the smoke does not require printing secrets;
- Codex can identify and terminate only the process it starts.
```

If all preconditions pass:

1. Start the official `OBM-POS Runtime Development` profile or equivalent exact launch command using its existing configuration.
2. Do not click business UI or perform CRUD.
3. Wait a bounded period, maximum 30 seconds, for a visible MainWindow/process window handle.
4. Record sanitized evidence only:

```text
ProcessStarted=True
MainWindowHandlePresent=True/False
MainWindowTitlePresent=True/False
ProcessStillRunning=True/False
StartupResultCode=<safe code when available>
API state=<Online|Offline|ReauthorizationRequired|Unknown>
```

5. Terminate only the process created by this smoke after evidence capture.
6. Do not stop PostgreSQL/API or any process that existed before the smoke.

A normal startup may write ordinary application logs or runtime metadata. Do not intentionally create/update business records, employee data, customers, invoices, payments, outbox business events, seed markers, or runtime state.

If any precondition is unsafe or the existing app is already running, skip the live smoke and report:

```text
LIVE_STARTUP_SMOKE=DEFERRED_TO_OPERATOR
```

Do not treat a safely deferred smoke as an implementation failure when build/tests/static wiring proof pass.

The InstallationV0 button handoff remains an operator physical test; do not automate clicks.

## Phase 13 — documentation task/result update

Canonical architecture remains V001. Do not create V002.

Before replacing current task/result, preserve prompt056 versions under the next available versioned history folder, expected:

```text
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V002\CURRENT_TASK_V002.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V002\CURRENT_RESULT_V002.md
```

Do not overwrite existing history.

After successful wiring:

- `CURRENT_RESULT.md` records final wiring, build/tests, live-smoke result or safe deferral, and pending operator physical tests.
- `CURRENT_TASK.md` authorizes only the operator physical runtime/InstallationV0/local CRUD/API-offline verification.
- Do not authorize ASP.NET Identity deletion until physical tests pass.
- Recompute and report current hashes.

## Build label

Set:

```text
Build label: prompt057
Window title: OBM InstallationV0 Phase 1/2 - prompt057
```

## Mutation restrictions

Do not:

```text
run migrations or seed
change PostgreSQL users/passwords/roles/GRANT/REVOKE
redeem Pairing Code
change API tokens/contracts
implement refresh tokens
change employee operational PIN values/rules
create customer/invoice/payment/business records
drop ASP.NET Identity tables
set User/Machine environment variables
commit/push OBM source
```

Source edits, tests, builds, safe configuration wiring, external evidence files, and a controlled non-business live startup smoke are allowed.

## Report 057

Create and push:

```text
report/report057.md
```

Required sections:

1. Verdict.
2. `DOCS_READ_BEFORE_CODE_GATE` evidence and hashes.
3. Pre-change wiring table.
4. Final production DI registration table and lifetimes.
5. Final local configuration chain and sanitized DB/user proof.
6. Final `LocalPosStartupService` decision ownership.
7. Final direct runtime path.
8. Final shared MainWindow transition.
9. Final InstallationV0 Open OBM-POS handoff path.
10. Proof API initializer is post-MainWindow and non-blocking.
11. Recovery/updater service boundary proof.
12. Exact files changed.
13. Tests added/updated.
14. Build/test commands and counts.
15. Static wiring verification results.
16. Controlled live startup smoke result or exact safe deferral reason.
17. Prompt057 label proof.
18. Canonical V001 unchanged and current hash.
19. History preservation and updated CURRENT_TASK/CURRENT_RESULT hashes.
20. No forbidden mutation/secret/source-push proof.
21. Exact operator physical retest steps.
22. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_CANONICAL_RUNTIME_WIRING_READY_FOR_PHYSICAL_LAUNCH
```

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```

```text
BLOCKED_OBM_POS_RUNTIME_WIRING_BUILD_OR_TEST
```

```text
BLOCKED_OBM_POS_RUNTIME_WIRING_CONFIGURATION
```
