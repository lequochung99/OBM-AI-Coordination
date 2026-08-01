# Prompt 043 — Close DevelopmentStartupGuard launch-provenance rejection after ProductRoot acceptance

## Physical evidence

Read completely:

```text
report/report040.md
report/report041.md
report/report042.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The operator physically retested the current build and proved it is not stale:

```text
Build label: prompt042
Phase 2 v002 Complete
ProductRoot shown in UI:
E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
```

Clicking `Open OBM-POS` still fails with:

```text
ResultCode=DevelopmentDatabaseRejected
StageId=DevelopmentStartupGuard
EffectiveProductRootSource=LaunchProfileEnvironment
RootPresent=True
RootApproved=True
ProfileName=DevelopmentProfile
CachedAssessmentReused=False
```

This evidence is authoritative.

It proves:

```text
stale binary = false
ProductRoot missing = false
ProductRoot unapproved = false
cached assessment reuse = false
```

Therefore prompt042 successfully bridged an approved ProductRoot, but `DevelopmentStartupGuard` still rejects on another predicate. The current generic dialog text is misleading because the root is present and approved.

Do not weaken the Development guard. Do not enable ProgramData fallback. Do not set User/Machine environment variables. Do not mutate PostgreSQL.

## Objective

Make both routes pass with the same strict Development safety contract:

```text
A. OBM-POS Runtime Development launch profile
   -> explicit approved ProductRoot
   -> approved runtime launch provenance
   -> InstalledHealthy
   -> MainWindow

B. InstallationV0 diagnostics -> Open OBM-POS
   -> completed v002 readiness already proven
   -> explicit approved ProductRoot handoff
   -> approved same-process runtime handoff provenance
   -> InstalledHealthy
   -> exactly one MainWindow
```

The installation diagnostic launch profile itself must not be treated as the final runtime profile, but a verified completed InstallationV0 handoff must be able to transition the same process into an approved runtime mode.

## First task — enumerate the complete guard predicate

Audit every branch and return site in:

```text
DevelopmentProfileLaunchPolicy.EvaluateStartupGuard
App.xaml.cs pre-configuration guard
App.xaml.cs post-configuration guard
runtime startup/readiness callers
```

For each predicate, record sanitized values and result:

```text
EnvironmentName
DatabaseName
Database locator source/classification
EffectiveProductRootSource
RootPresent
RootApproved
ProgramDataFallbackUsed
ProtectedDatabaseRejected
LaunchProfileName / profile classification
SPACEPOS_INSTALLATION_MODULE present/value classification
InstallationDiagnosticsRequested
RuntimeDevelopmentProfileRecognized
VerifiedInstallationHandoffPresent
RuntimeHandoffAuthorized
HandoffProductRootMatches
HandoffDatabaseMatches
HandoffEnvironmentMatches
CachedAssessmentReused
```

Do not stop at the first root check. Identify the exact boolean that returned `DevelopmentDatabaseRejected` even though `RootPresent=True` and `RootApproved=True`.

Replace misleading generic rejection with a precise safe result code for each failing predicate.

## Likely fault classes — verify rather than assume

Investigate all of these:

1. `SPACEPOS_INSTALLATION_MODULE=InstallationV0` remains present after clicking Open OBM-POS, so the guard still classifies the process as diagnostics-only.
2. The guard requires a Runtime Development launch-profile provenance marker even when a verified same-process InstallationV0 handoff exists.
3. `EffectiveProductRootContext` retains `LaunchProfileEnvironment` rather than transitioning to `InstallationHandoff` because the same normalized root already exists.
4. A profile-name/classification comparison rejects `DevelopmentProfile` even though root/database/environment are approved.
5. Pre-configuration startup mode remains latched as installation mode after the in-process handoff.
6. Another guard input such as database-locator provenance is failing while the UI still shows the old root-specific message.

Report the exact root cause and exact rejection branch.

## Canonical launch provenance model

Implement an explicit process-local launch provenance state with at least these modes:

```text
InstallationDiagnostics
RuntimeDevelopmentProfile
VerifiedInstallationHandoff
MissingOrRejected
```

A verified InstallationV0 handoff is allowed only after all of these are already true:

```text
Phase 2 v002 marker exists exactly once
identity spine verified
RuntimeState = Activated
ProductRoot normalized and approved
DatabaseName = obm_pos_dev_v0_pg
EnvironmentName = Development
ProductRoot/database match the hydrated completed lane
```

This provenance must be created internally by the successful `Open OBM-POS` callback. It must not be accepted from an arbitrary external environment variable or command-line string.

The provenance context should be process-local, in-memory, and one-way for this process transition.

## Required same-process mode transition

After the operator clicks `Open OBM-POS` and the completed-state checks pass, perform the transition in this order:

```text
1. Validate exact normalized ProductRoot.
2. Validate database/environment/completed marker/identity spine.
3. Set EffectiveProductRoot source = InstallationHandoff, even when the same path was already supplied by the diagnostics launch profile.
4. Set approved runtime handoff provenance = VerifiedInstallationHandoff.
5. Clear or neutralize process-scoped SPACEPOS_INSTALLATION_MODULE for subsequent runtime routing, but only after successful internal handoff validation.
6. Clear any latched in-memory diagnostics-only startup flag used by the runtime router.
7. Preserve process-scoped SPACEPOS_PRODUCT_ROOT with the approved path.
8. Dispose/recreate all guard, configuration, locator, and readiness objects that captured installation-mode inputs.
9. Re-run DevelopmentStartupGuard and InstalledHealthy assessment.
10. Open exactly one MainWindow, assign Application.Current.MainWindow, then close InstallationV0Window.
```

Do not write User/Machine variables.

If clearing `SPACEPOS_INSTALLATION_MODULE` is not the correct architecture, replace it with an explicit runtime-mode parameter/context passed to every relevant router. Do not leave a mixed diagnostics/runtime state.

## DevelopmentStartupGuard acceptance matrix

The guard should accept exactly these Development cases:

### Runtime Development profile

```text
approved ProductRoot
approved Development DB
Environment=Development
provenance=RuntimeDevelopmentProfile
no installation-module request
```

### Verified same-process handoff

```text
approved ProductRoot
approved Development DB
Environment=Development
provenance=VerifiedInstallationHandoff
completed InstallationV0 readiness proof
installation diagnostics mode transitioned off for runtime routing
```

Reject:

```text
missing root
ProgramData fallback
unapproved/stale/production root
protected/unapproved DB
arbitrary process launched without approved profile or verified handoff
installation diagnostics process attempting runtime without completed proof
profile/handoff root mismatch
profile/handoff DB mismatch
```

## Diagnostics requirement

On failure, show safe fields including:

```text
ResultCode
StageId
RejectedPredicate
LaunchProvenance
EffectiveProductRootSource
RootPresent
RootApproved
DatabaseApproved
EnvironmentApproved
InstallationModulePresent
DiagnosticsModeLatched
RuntimeHandoffAuthorized
ProfileName
CachedAssessmentReused
```

The user-facing message must match the actual rejected predicate. Do not say the root is missing when `RootPresent=True` and `RootApproved=True`.

## Preserve completed database state

No DB mutation in this prompt.

Read-only verification should continue to show:

```text
TblEmployeePermission = 7
TblEmployee = 20
TblLocalOutbox = 62
Phase2 v002 marker = 1
RuntimeState = Activated
```

Do not rerun Phase 2, rewrite marker/outbox, redeem a Pairing Code, or modify runtime history.

## Tests

Add focused tests for:

```text
root present + root approved + diagnostics provenance is rejected before handoff
verified InstallationV0 handoff transitions provenance to InstallationHandoff/VerifiedInstallationHandoff
same root from launch environment is reclassified as InstallationHandoff after verified click
installation-module diagnostics flag does not remain latched for runtime route after verified handoff
arbitrary clearing of installation flag without verified handoff remains rejected
Runtime Development profile direct route remains accepted
verified handoff route accepted without requiring Runtime Development profile name
ProgramData/unapproved/protected routes remain rejected
specific rejection message matches actual predicate
fresh guard/readiness instances are used
exactly one MainWindow opens
no DB mutation during handoff
prompt043 label
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap|FullyQualifiedName~DevelopmentProfile"
```

If the combined filter is unsupported, run the discovered relevant classes separately and report exact counts.

Do not run final physical WPF acceptance automatically unless it can be done without interfering with the operator session.

## Build label

Set:

```text
Build label: prompt043
Window title: OBM InstallationV0 Phase 1/2 - prompt043
```

## Report 043

Create and push:

```text
report/report043.md
```

Required sections:

1. Verdict.
2. Prompt042 physical diagnostics proof.
3. Exact rejected predicate and source line/method.
4. Complete DevelopmentStartupGuard predicate matrix.
5. SPACEPOS_INSTALLATION_MODULE / diagnostics-mode audit.
6. EffectiveProductRoot source-transition audit.
7. Exact root cause.
8. Canonical launch provenance design.
9. Same-process mode-transition implementation.
10. Direct Runtime Development route behavior.
11. Verified Installation handoff behavior.
12. Precise fail-closed result codes/messages.
13. Source files changed.
14. Build/test commands and counts.
15. Read-only DB no-delta proof.
16. Prompt043 label proof.
17. Exact operator retest steps for both routes.
18. No secrets/no source push confirmation.
19. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_VERIFIED_RUNTIME_HANDOFF_PROVENANCE_READY_FOR_USER_RETEST
```

```text
OBM_POS_RUNTIME_DEVELOPMENT_PROFILE_READY_FOR_USER_RETEST
```

```text
BLOCKED_OBM_POS_DEVELOPMENT_STARTUP_GUARD
```
