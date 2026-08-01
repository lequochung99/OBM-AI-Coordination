# Prompt 045 — Simplify OBM-POS startup to local-DB-first runtime; scope installation and Development guards

## Read first

Read completely before changing source:

```text
report/report044.md
report/report043.md
report/report040.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The authoritative audit verdict is:

```text
OBM_POS_RUNTIME_CONNECTIVITY_AND_PIN_MODEL_AUDIT_COMPLETE
```

## Operator decision — canonical runtime model

OBM-POS has only two canonical runtime connections:

```text
1. WPF -> local PostgreSQL
   Standard host/port/database/username/password authentication.
   After PostgreSQL accepts the runtime role and the local schema/state is usable,
   ordinary local CRUD must not depend on API availability, employee PIN,
   Pairing Code, launch-profile provenance, ProductRoot provenance, or a hidden
   application-defined security gate.

2. WPF -> API
   API authentication is separate from local DB operation.
   Current token lifecycle is incomplete and will be addressed in a later prompt.
   API absence/expiry must not prevent MainWindow from opening locally.
```

Employee `LoginNumber` is an **operational PIN**, not an application password and not a runtime startup credential.

This prompt implements **S1 only** from report044:

```text
S1 — Local DB startup independent of API and PIN provenance
```

Do not implement refresh tokens, PIN-length changes, historical outbox migration, or a new authentication framework in this prompt.

## Primary objective

Replace the current V0 normal-runtime predicate with the smallest safe local readiness predicate:

```text
Start OBM-POS
    -> resolve one ProductRoot/config lane
    -> resolve local DB configuration
    -> Development-only protected-lane safety check
    -> authenticate to PostgreSQL using the configured runtime username/password
    -> verify schema/migrations are ready
    -> verify one usable local runtime profile and local Tenant/POS identity consistency
    -> verify runtime state is Activated
    -> open MainWindow
    -> initialize API/sync independently and non-blockingly
```

Normal installed runtime must not require exact Phase1/Phase2 trial proof details on every startup.

## Critical distinction: readiness is not authentication

Keep these as genuine local runtime requirements:

```text
- database config exists;
- PostgreSQL connection succeeds;
- current role can perform required runtime operations;
- schema minimum is present;
- exactly one selected/current runtime profile is usable;
- RuntimeState = Activated;
- selected TblTenant / TblPosLocal / TblPosRuntimeProfile identity is internally consistent;
- database is not a protected/forbidden Development target.
```

Do **not** use these as ordinary local runtime authentication gates:

```text
- Phase1 ApiAuthorized checkpoint availability or API token validity;
- protected WpfJwt availability/expiration;
- API reachability;
- exact Phase2 employee count = 20;
- exact permission count = 7;
- exact outbox evidence/count;
- exact trial marker provenance on every startup;
- InstallationV0 diagnostic launch provenance;
- LaunchProvenanceContext as authorization;
- employee operational PIN configuration;
- Pairing Code state.
```

A compact installed-baseline/version check may remain only if it is a genuine database compatibility/readiness check. It must not require reference-trial row counts or API proof. Explain and test the exact minimum retained check.

## Source audit before modification

Audit the current startup chain and report the exact pre-change decision tree:

```text
App.xaml / App.xaml.cs
ApplicationStartupCoordinator
RuntimeProfileStartupAssessmentService
InstallationV0CompletedReadinessService
DevelopmentProfileLaunchPolicy
EffectiveProductRootContext
LaunchProvenanceContext
RuntimeBootstrapLocator
RuntimeDatabaseCredentialProvider
DatabaseStartupAssessmentService / schema checks
MainWindow launch path
InstallationV0Window launch path
AppJwtBootstrapper
AppHost / SignalR / sync startup
```

Identify every place that currently requires:

```text
Phase1 checkpoint
Phase2 v002 marker
permission count
employee count
outbox evidence
launch provenance
API token state
```

for `InstalledHealthy` or MainWindow routing.

## Canonical normal-runtime assessment

Create or refactor one clearly named local readiness result, for example:

```text
LocalPosRuntimeReady
LocalDatabaseReadyForPos
InstalledLocalRuntimeHealthy
```

Do not overload `InstalledHealthy` with API/bootstrap proof.

The assessment should expose safe evidence such as:

```text
DatabaseConfigResolved
DatabaseAuthenticationSucceeded
SchemaReady
RuntimeProfileCount
RuntimeState
TenantIdentityConsistent
PosIdentityConsistent
DevelopmentLaneApproved
LocalRuntimeReady
```

No passwords, connection strings, raw PINs, tokens, or private identity dumps.

### Runtime DB role

Use the existing runtime DB credential path:

```text
RuntimeBootstrapLocator
RuntimeDatabaseCredentialProvider
Npgsql connection
```

Do not:

```text
create roles
alter roles
GRANT/REVOKE
change the database password
store credentials in PostgreSQL
add a hidden application authorization flag
```

This prompt must not mutate PostgreSQL as part of startup.

## Development safety scope

Preserve one small Development-only guard that prevents destructive mistakes:

```text
- explicit isolated ProductRoot/config lane;
- approved Development database name;
- forbidden production/Royal/protected database names rejected;
- no ProgramData fallback in ambiguous Development debug.
```

But once the explicit ProductRoot and local DB lane are resolved and approved, do not continue requiring runtime launch provenance.

Required simplification:

```text
DevelopmentProfileLaunchPolicy
-> validates root/database lane for Development safety
-> does not authenticate the installed POS

LaunchProvenanceContext
-> only diagnostics/debug handoff metadata
-> not part of production/local CRUD readiness

SPACEPOS_INSTALLATION_MODULE
-> only chooses InstallationV0 diagnostics mode
-> not durable runtime state
```

For the verified same-process `Open OBM-POS` handoff:

```text
- accept the already verified explicit ProductRoot;
- run the same minimal local DB readiness assessment;
- do not require a second profile-provenance ceremony;
- open exactly one MainWindow and close diagnostics on success.
```

Do not weaken protected-database checks.

## InstallationV0 and upgrade verification scope

Keep full proof checks where they belong:

```text
InstallationV0 diagnostics
installation completion verification
upgrade verification
recovery/audit tools
```

These may continue to verify:

```text
Phase1 checkpoint
Phase2 marker
identity spine
seed row counts
permission/employee/outbox evidence
marker-last transaction proof
```

Normal startup should not repeat all of those as authentication before every MainWindow.

If a completed marker exists but local DB/schema/runtime state is inconsistent, fail closed to recovery/diagnostics with a precise local reason. Do not rerun seed automatically.

## API independence and offline startup

MainWindow must open after local DB readiness succeeds, regardless of:

```text
API server unreachable
DNS failure
timeout
WpfJwt/bootstrap credential expired
station access token missing/expired
SignalR unavailable
sync worker failure
```

After MainWindow opens:

```text
- start API/auth/sync workers independently;
- valid token -> Online;
- token missing/expired or API unavailable -> Offline/Deferred;
- keep local CRUD available;
- keep TblLocalOutbox pending;
- retry/reconnect behavior may remain current for now.
```

At minimum, remove startup-blocking/modal behavior from ordinary API bootstrap failure. Replace the current `POS API login error` startup modal with non-blocking status/logging suitable for a later full offline-mode prompt.

Do not implement refresh-token issuance or rotation in prompt045.

## Employee operational PIN boundary

Normal startup must not require operational PINs to be configured.

Current seeded employee rows may contain deterministic `UNCONFIGURED` placeholder values. That must not block MainWindow or local DB CRUD.

Do not change:

```text
PIN values
PIN schema
PIN length policy
PIN validators
PIN uniqueness
manager-area behavior
```

in this prompt.

Use corrected terminology in any changed comments/messages:

```text
employee operational PIN
manager/non-Staff PIN
```

Do not introduce `employee password` terminology.

## Expected startup decisions

### Local DB ready, API online

```text
MainWindow opens
API/sync online
```

### Local DB ready, API offline

```text
MainWindow opens
API/sync status Offline/Deferred
no InstallationV0Window
no Pairing Code request
no RuntimeState recovery transition
```

### Local DB ready, access token expired

```text
MainWindow opens
API/sync status Offline/Reauthorization Required or Deferred
no local CRUD block
```

### Local DB missing/unreachable/schema invalid

```text
Installation/recovery UI opens
safe exact local DB reason shown
```

### Development points to protected DB/root

```text
Development guard blocks before local runtime
precise protected-lane reason
```

## No-mutation startup proof

Prove opening and closing normal POS does not change:

```text
TblEmployeePermission
TblEmployee
TblLocalOutbox
Phase2TrialCompletionMarker
TblPosRuntimeProfile identity/state
TblPosRuntimeStateHistory
```

Normal startup must not:

```text
run seed
rewrite markers
redeem Pairing Code
update employee PIN placeholders
insert outbox
change runtime state merely because API is offline
```

Use read-only pre/post evidence or focused tests. Do not physically mutate DB for proof.

## Source files likely involved

Inspect and change only what is necessary, likely including:

```text
E:\Project2026\4POS\NailSalonNet8\App.xaml.cs
E:\Project2026\4POS\NailSalonNet8\Services\Startup\ApplicationStartupCoordinator.cs
E:\Project2026\4POS\NailSalonNet8\Services\Startup\RuntimeProfileStartupAssessmentService.cs
E:\Project2026\4POS\NailSalonNet8\Services\Startup\InstallationV0CompletedReadinessService.cs
E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\DevelopmentProfileLaunchPolicy.cs
E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\LaunchProvenanceContext.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs
E:\Project2026\4POS\NailSalonNet8.Tests\...
```

Do not change API contracts or PostgreSQL schema in this prompt.

## Tests

Add focused tests for at least:

```text
local DB healthy + API unavailable -> MainWindow route
local DB healthy + bootstrap token expired -> MainWindow route
local DB healthy + operational PIN placeholders -> MainWindow route
local DB healthy does not require exact employee/outbox counts
local DB healthy does not require launch provenance outside Development diagnostics
Phase1 checkpoint absent after completed local installation does not block ordinary local runtime
Phase2 detailed proof remains available in InstallationV0 diagnostics
DB missing -> installation/recovery route
schema not ready -> installation/recovery route
RuntimeState not Activated -> blocked/recovery route
identity inconsistency -> blocked/recovery route
Development protected DB/root -> rejected
same-process Open OBM-POS -> minimal local readiness -> one MainWindow
API startup exception is non-blocking and does not show the old modal error
normal startup performs no seed/marker/outbox mutation
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap|FullyQualifiedName~DevelopmentProfile"
```

## Build label

If InstallationV0 diagnostics source changes, set:

```text
Build label: prompt045
Window title: OBM InstallationV0 Phase 1/2 - prompt045
```

Do not add the coordination label to production MainWindow unless an existing Development banner already supports it.

## Physical execution policy

Do not automatically launch WPF if it may interfere with the operator session.

Do not mutate PostgreSQL.

Leave final physical startup tests to the operator after builds/tests pass:

```text
Route A: OBM-POS Runtime Development -> MainWindow
Route B: InstallationV0 complete -> Open OBM-POS -> MainWindow
Route C: API stopped/unreachable -> MainWindow local mode
```

## Report 045

Create and push:

```text
report/report045.md
```

Required sections:

1. Verdict.
2. Pre-change over-engineered startup decision tree.
3. Exact gates removed from normal runtime.
4. Exact gates retained for local DB readiness.
5. Development-only guard scope.
6. InstallationV0 diagnostics/upgrade proof scope.
7. New local-DB-first startup decision tree.
8. API-offline and expired-token behavior.
9. Employee operational PIN boundary.
10. Open OBM-POS same-process handoff behavior.
11. No-mutation startup proof.
12. Exact source files changed.
13. Build/test commands and counts.
14. Read-only DB evidence.
15. Exact operator physical retest steps.
16. Risks/deferred work: refresh token, reconnect formalization, PIN 4/6 normalization.
17. No secrets/no DB mutation/no source push proof.
18. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_LOCAL_DB_FIRST_RUNTIME_READY_FOR_USER_RETEST
```

```text
BLOCKED_OBM_POS_LOCAL_DB_FIRST_RUNTIME_SIMPLIFICATION
```
