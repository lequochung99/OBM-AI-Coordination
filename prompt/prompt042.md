# Prompt 042 — Close the persistent Development ProductRoot handoff failure

## Physical operator evidence

Read completely before changing source:

```text
report/report040.md
report/report041.md
prompt/prompt041.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The operator physically retested after prompt041 and still received the identical dialog:

```text
DevelopmentDatabaseRejected
Development launch requires an explicit isolated SPACEPOS_PRODUCT_ROOT.
Default ProgramData fallback is disabled for Development.
Use the approved Development profile launcher or Visual Studio debug profile.
```

The approved ProductRoot remains:

```text
E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
```

The completed DB lane remains:

```text
DatabaseName = obm_pos_dev_v0_pg
EnvironmentName = Development
Phase 2 v002 = Complete
RuntimeState = Activated
```

Do not weaken the Development guard. Do not enable ProgramData fallback. Do not persist a User- or Machine-level environment variable.

## First gate — prove whether the operator ran stale or current binaries

Before changing logic, inspect and report sanitized physical/build evidence:

```text
active InstallationV0 build label expected = prompt041
running process executable path
loaded NailSalonNet8 assembly path
loaded InstallationV0 assembly path
file timestamps or hashes
Visual Studio launch profile name if discoverable
SPACEPOS_INSTALLATION_MODULE presence classification
process-level SPACEPOS_PRODUCT_ROOT presence classification
```

Do not print secrets or connection strings.

If the active UI/build is not prompt041 or loaded assemblies are stale, classify:

```text
STALE_PROMPT041_BINARY_CONFIRMED
```

Then still ensure the source fix below is correct, but clearly distinguish stale-binary evidence from a real prompt041 logic failure.

## Find the exact rejection call site

Search the entire WPF source for:

```text
DevelopmentDatabaseRejected
Development launch requires an explicit isolated SPACEPOS_PRODUCT_ROOT
SPACEPOS_PRODUCT_ROOT
DevelopmentProfileLaunchPolicy
ProgramData fallback is disabled
```

Inventory every direct read of:

```csharp
Environment.GetEnvironmentVariable("SPACEPOS_PRODUCT_ROOT")
```

and every cached/static ProductRoot decision.

Identify the exact call path that generated the physical dialog after clicking `Open OBM-POS` or launching the runtime profile.

Report:

```text
rejection StageId
resolver/call-site class and method
whether the decision was made before or after the prompt041 handoff callback
whether a service cached the old missing-root result
whether the handoff used a different resolver than the Development guard
```

Do not rely on the generic dialog text alone.

## Canonical effective ProductRoot contract

Create one authoritative ProductRoot resolver/context used by every Development startup guard and readiness service.

Required precedence:

```text
1. Explicit ProductRoot verified and handed off by InstallationV0 in the current process.
2. Process environment SPACEPOS_PRODUCT_ROOT supplied by the approved launch profile.
3. No Development fallback.
```

The resolver must return both:

```text
EffectiveProductRoot
EffectiveProductRootSource = InstallationHandoff | LaunchProfileEnvironment | Missing | Rejected
```

The exact approved V0 root must be validated by normalized full path comparison.

Reject:

```text
null/empty root
relative path
ProgramData fallback
source/repository root
another _dev lane
protected/production root
handoff/profile disagreement
nonexistent or non-writable root when required by existing contract
```

## Required bridge for the same-process handoff

Prompt041 used an in-memory override, but the physical failure proves at least one downstream guard still did not observe it.

After validating the explicit InstallationV0 handoff root, perform both of these process-local actions before constructing or rerunning startup/readiness services:

```text
A. Set the canonical in-memory EffectiveProductRoot context.
B. Set SPACEPOS_PRODUCT_ROOT for Process scope only.
```

Equivalent C# shape:

```csharp
Environment.SetEnvironmentVariable(
    "SPACEPOS_PRODUCT_ROOT",
    verifiedProductRoot,
    EnvironmentVariableTarget.Process);
```

This is allowed only after the root passes the strict Development approval policy.

Do not set User or Machine environment variables.

The process-level environment bridge exists so legacy components that still read the environment during this transition observe the same explicit verified root. New/changed code should use the canonical resolver rather than reading the environment directly.

## Cache/order correction

The explicit ProductRoot must be applied **before** any of these are instantiated or evaluated for the handoff:

```text
DevelopmentProfileLaunchPolicy
DatabaseStartupAssessmentService
RuntimeProfileStartupAssessmentService
InstallationV0CompletedReadinessService
DB locator/provider factory
MainWindow startup bootstrapper
```

If any service caches a root or a rejection result:

```text
invalidate/recreate the readiness scope after applying the override
```

Do not reuse a previously computed `DevelopmentDatabaseRejected` assessment.

The corrected same-process sequence must be:

```text
InstallationV0Window
→ obtain service-resolved ProductRoot
→ normalize and validate exact approved root
→ set effective ProductRoot context
→ set process-scoped SPACEPOS_PRODUCT_ROOT
→ create a fresh startup/readiness scope
→ rerun InstalledHealthy assessment
→ open exactly one MainWindow
→ set Application.Current.MainWindow
→ close InstallationV0Window
```

## Runtime profile route

The normal Visual Studio profile:

```text
OBM-POS Runtime Development
```

must set:

```text
SPACEPOS_PRODUCT_ROOT=E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
DOTNET_ENVIRONMENT=Development
```

and must not set:

```text
SPACEPOS_INSTALLATION_MODULE=InstallationV0
```

Launching this profile must open MainWindow directly when readiness passes.

## Diagnostics improvement

Replace the generic-only physical failure with a safe detailed result when the guard rejects:

```text
ResultCode
StageId
EffectiveProductRootSource
RootPresent true/false
RootApproved true/false
ProfileName classification if available
CachedAssessmentReused true/false
```

The UI may display the approved Development root path because it is not a secret, but do not display connection strings or credentials.

Required result codes include:

```text
DEVELOPMENT_PRODUCT_ROOT_MISSING
DEVELOPMENT_PRODUCT_ROOT_NOT_APPROVED
DEVELOPMENT_PRODUCT_ROOT_HANDOFF_PROFILE_MISMATCH
DEVELOPMENT_PRODUCT_ROOT_CACHED_ASSESSMENT_STALE
```

## Preserve all completed installation invariants

Do not mutate PostgreSQL.

Preserve:

```text
TblEmployeePermission = 7
TblEmployee = 20
TblLocalOutbox = 62
Phase2TrialCompletionMarker v002 = 1
TblPosRuntimeProfile RuntimeState = Activated
identity spine verified
```

No seed, migration, pairing redeem, marker rewrite, outbox insert, employee update, or runtime-history transition during startup handoff.

## InstallationV0 UI

Set:

```text
Build label: prompt042
Window title: OBM InstallationV0 Phase 1/2 - prompt042
```

When v002 is complete:

```text
Open OBM-POS
```

must remain visible and non-mutating.

On success:

```text
MainWindow opens
InstallationV0Window closes
one WPF process/window route only
```

## Required focused tests

Add tests proving:

```text
explicit handoff root has precedence over missing launch-profile environment
verified handoff writes Process-scope SPACEPOS_PRODUCT_ROOT
User/Machine environment is never written
runtime profile environment root works without handoff
handoff/profile same normalized root passes
handoff/profile different roots fail closed
ProgramData fallback remains rejected
stale cached rejection is not reused after valid handoff
fresh readiness scope sees the handoff root
all Development guards use the canonical resolver or process bridge
Open OBM-POS opens MainWindow exactly once
runtime profile opens MainWindow directly
startup/readiness remains read-only
prompt042 label
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap"
```

Do not automatically launch WPF if it may interfere with the operator session. Leave final physical retest to the operator unless a safe isolated smoke is available.

## Report 042

Create and push:

```text
report/report042.md
```

Required sections:

1. Verdict.
2. Physical prompt041 failure evidence.
3. Stale/current binary proof.
4. Exact rejection call site and StageId.
5. Root resolver/caching root cause.
6. Canonical EffectiveProductRoot resolver design.
7. Process-scope environment bridge proof.
8. Service construction/cache ordering correction.
9. Runtime Development profile proof.
10. Same-process Open OBM-POS sequence.
11. Development guard remains fail-closed.
12. Source files changed.
13. Build/test counts.
14. Read-only DB no-delta proof.
15. Prompt042 label proof.
16. Exact operator retest steps for both routes.
17. No secrets/no source push confirmation.
18. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_EFFECTIVE_PRODUCTROOT_HANDOFF_READY_FOR_USER_RETEST
```

```text
STALE_PROMPT041_BINARY_CONFIRMED_SOURCE_READY_FOR_RETEST
```

```text
BLOCKED_OBM_POS_EFFECTIVE_PRODUCTROOT_HANDOFF
```
