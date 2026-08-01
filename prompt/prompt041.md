# Prompt 041 — Preserve explicit Development ProductRoot during InstallationV0 → Main POS handoff

## Physical operator evidence

Read completely:

```text
report/report039.md
report/report040.md
prompt/prompt040.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The operator physically ran build `prompt040` through the InstallationV0 diagnostics route. The window correctly hydrated:

```text
Phase 2 Local DB Baseline: Phase 2 v002 Complete
Target DB: obm_pos_dev_v0_pg
ProductRoot: E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
Open OBM-POS button visible
```

Clicking `Open OBM-POS` failed with:

```text
DevelopmentDatabaseRejected
Development launch requires an explicit isolated SPACEPOS_PRODUCT_ROOT.
Default ProgramData fallback is disabled for Development.
Use the approved Development profile launcher or Visual Studio debug profile.
```

This proves the InstallationV0 diagnostic UI has already resolved the approved ProductRoot, but the same-process main POS readiness/startup handoff does not receive or recognize that explicit ProductRoot.

Do not weaken the Development guard and do not enable ProgramData fallback.

## Objective

Make both supported routes work safely:

```text
A. NailSalonNet8 + OBM-POS Runtime Development profile
   -> explicit ProductRoot
   -> readiness PASS
   -> MainWindow

B. InstallationV0 diagnostics + Open OBM-POS
   -> reuse the already verified explicit ProductRoot
   -> same readiness predicate
   -> MainWindow in same process
```

Both routes must resolve exactly:

```text
E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
```

and must never fall back to ProgramData in Development.

## Root-cause audit

Audit all ProductRoot sources and consumers:

```text
Properties\launchSettings.json
SPACEPOS_PRODUCT_ROOT
SPACEPOS_INSTALLATION_MODULE
InstallationV0 module/options ProductRoot
InstallationV0Window hydrated ProductRoot
Open OBM-POS callback/handoff
App.xaml.cs startup path
InstallationV0CompletedReadinessService
RuntimeProfileStartupAssessmentService
Database locator resolution
DevelopmentDatabaseRejected guard
```

Determine and report exactly:

1. Which source produced the ProductRoot shown in InstallationV0Window.
2. Which source the main POS readiness guard checks.
3. Why the guard sees ProductRoot as missing during same-process handoff.
4. Whether the Runtime Development profile works independently or also has a propagation defect.

## Canonical ProductRoot contract

Create or reuse one explicit ProductRoot abstraction/value object for the process.

Precedence must be deterministic:

```text
1. Explicit verified handoff ProductRoot supplied by InstallationV0
2. SPACEPOS_PRODUCT_ROOT from approved launch profile
3. No Development fallback
```

Hard rules:

```text
Development + no explicit ProductRoot -> reject
Development + ProgramData default -> reject
ProductRoot outside approved isolated root policy -> reject
ProductRoot mismatch between diagnostics and main readiness -> reject
```

Do not read one ProductRoot for UI display and a different ProductRoot for main startup.

## Preferred same-process fix

Preferred design:

```text
InstallationV0Window completed-state handler
-> passes the already resolved/verified ProductRoot explicitly
-> main readiness service uses that exact value
-> MainWindow opens
-> InstallationV0Window closes
```

Do not depend on setting a user-level or machine-level environment variable.

A process-level environment-variable assignment is acceptable only if the current architecture cannot pass the value explicitly, and only when:

```text
value equals the already verified ProductRoot
scope is current process only
readback equals the supplied value
no global persistence occurs
```

Document why explicit dependency passing was or was not possible.

## Runtime Development profile

Verify `OBM-POS Runtime Development` contains:

```text
SPACEPOS_PRODUCT_ROOT=E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
DOTNET_ENVIRONMENT=Development
no SPACEPOS_INSTALLATION_MODULE
```

The report must state exact Visual Studio steps and prove the profile appears after reload.

## Readiness predicate remains unchanged

Preserve prompt040 `InstalledHealthy` requirements:

```text
Phase 1 checkpoint valid
Phase 2 v002 marker exactly once
identity spine matches
RuntimeState Activated
permissions = 7
employees = 20
canonical outbox evidence exists
DB = obm_pos_dev_v0_pg
Environment = Development
```

No seed, migration, marker write, outbox insert, Pairing Code redeem, or DB mutation during handoff/startup.

## UI behavior

When `Open OBM-POS` succeeds:

```text
MainWindow shown
Application.Current.MainWindow = MainWindow
InstallationV0Window closed
no second WPF process
no duplicate MainWindow
```

When ProductRoot validation fails:

```text
stay in InstallationV0Window
show safe reason and expected approved root classification
no ProgramData fallback
```

Remove or replace the stale Phase 1 message:

```text
Local database installation is not started.
```

When Phase 2 v002 is complete, show:

```text
Phase 1 resume verified. Pairing Code is not required.
Phase 2 database status is shown above.
```

## Physical proof expectations

Codex should not interfere with an operator UI session. If safe physical launch is not possible, leave it for operator retest.

Required operator proofs:

### Route A

```text
NailSalonNet8 startup project
OBM-POS Runtime Development profile
F5
NailSalonNet8.exe opens MainWindow
```

### Route B

```text
InstallationV0 diagnostic profile
Phase 2 v002 Complete
Open OBM-POS
same process opens MainWindow
no DevelopmentDatabaseRejected
```

Both routes must use the same ProductRoot and DB.

## Tests

Add focused tests for:

```text
explicit handoff ProductRoot accepted
Runtime Development profile ProductRoot accepted
missing ProductRoot rejected in Development
ProgramData fallback rejected in Development
mismatched handoff/profile roots rejected
same ProductRoot used by UI, readiness, locator, and startup
Open OBM-POS opens one MainWindow and closes diagnostics
no process-global/user-global environment persistence
no DB mutation during handoff
stale Phase 1 database message removed after v002 complete
prompt041 label
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap"
```

## Build label

Because InstallationV0 source changes are expected:

```text
Build label: prompt041
Window title: OBM InstallationV0 Phase 1/2 - prompt041
```

## Report 041

Create and push:

```text
report/report041.md
```

Required sections:

1. Verdict.
2. Physical DevelopmentDatabaseRejected evidence.
3. Exact ProductRoot source mismatch.
4. Canonical ProductRoot precedence/contract.
5. Same-process Open OBM-POS fix.
6. Runtime Development profile verification.
7. Development guard preserved.
8. MainWindow handoff behavior.
9. Stale Phase 1 message correction.
10. Source files changed.
11. Build/test counts.
12. No DB mutation proof.
13. Route A operator steps.
14. Route B operator steps.
15. Prompt041 label proof.
16. No secrets/no source push confirmation.
17. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_PRODUCTROOT_HANDOFF_READY_FOR_USER_RETEST
```

```text
OBM_POS_PRODUCTROOT_HANDOFF_PHYSICAL_PASS
```

```text
BLOCKED_OBM_POS_PRODUCTROOT_HANDOFF
```
