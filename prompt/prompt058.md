# Prompt 058 — Remove duplicate station-identity gating and carry the local POS identity into MainWindow

## Physical operator evidence

The operator rebuilt and physically ran build `prompt057` through the InstallationV0 profile.

Visible diagnostics prove:

```text
Build label: prompt057
Phase 2 Local DB Baseline: Phase 2 v002 Complete
Local POS status: Ready / Activated / LOCAL_POS_READY_ACTIVATED
LocalDatabaseConfigResolved=True
LocalDatabaseAuthenticationSucceeded=True
SchemaReady=True
RuntimeProfileCount=1
RuntimeState=Activated
TenantIdentityConsistent=True
PosIdentityConsistent=True
LocalPosReady=True
API status: Reauthorization Required / WPF_HELLO_HTTP_401
```

After clicking `Open OBM-POS` once, the structured result is:

```text
Open OBM-POS state: Failed
StageId=InstallationV0OpenObmPos
ResultCode=POS_RUNTIME_ROUTE_STATIONIDENTITYMISSING
```

No MainWindow became visible.

This is authoritative physical evidence.

It proves:

```text
- local PostgreSQL is usable;
- the runtime profile is Activated;
- Tenant/POS identity checks already passed;
- LocalPosStartupService returned local readiness;
- the shared handoff callback ran;
- one later station-identity predicate or missing payload still blocks MainWindow;
- API HTTP 401 is unrelated.
```

Do not redeem a Pairing Code.

## Mandatory documentation gate

Before any source edit, read completely:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report056.md
report/report057.md
```

Current expected documentation state from report057:

```text
CanonicalDocVersion=V001
CanonicalDocSha256=7044A02F29FE349FE531DE6800BA739E6B29EA473B9A867881B283DB8743BC72
CURRENT_TASK.md Hash=D2AE96BE77FCE9B1028810355CCE9FE2E043E37D83611381FC8FE970D086118A
CURRENT_RESULT.md Hash=D3342AB33DAA8EF8F8B8345988BDB43DAAFA2887806EFD2366FCB116525EFF0D
```

Record:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V001
CanonicalDocSha256=<actual current hash>
```

If the current canonical contract no longer authorizes the simple local-DB runtime rule, stop before source edits with:

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```

## Canonical rule

```text
Local PostgreSQL usable
+ local Tenant/POS context resolvable
-> MainWindow
-> API session afterward
```

The following do not block MainWindow:

```text
WPF_HELLO_HTTP_401
expired API token
Pairing Code absent
protected hello failure
API unavailable
```

Employee LoginNumber/PIN is not a startup credential.

## Exact objective

Fix only the physical `POS_RUNTIME_ROUTE_STATIONIDENTITYMISSING` defect.

For the current state:

```text
LocalPosReady=True
RuntimeState=Activated
RuntimeProfileCount=1
TenantIdentityConsistent=True
PosIdentityConsistent=True
```

the required result is:

```text
ResultCode=OPEN_POS_MAINWINDOW_SHOWN
MainWindowConstructed=True
MainWindowShown=True
MainWindowVisible=True
LocalRuntimeReady=True
```

Do not add another route layer, fallback identity source, compatibility shim, API proof, or hidden flag.

## First task — locate the exact predicate and value loss

Trace every active definition/reference that can produce:

```text
POS_RUNTIME_ROUTE_STATIONIDENTITYMISSING
StationIdentityMissing
STATION_IDENTITY_MISSING
stationIdentity == null
StationGuid
PosGuid
TenantGuid
PosIdentity
StationIdentity
```

Audit at least:

```text
App.xaml.cs
LocalPosStartupService
LocalPosStartupResult
LocalPosStartupDecision
MainWindowOpenResult
RouteFromAssessment or current equivalent
StartNormalApplicationAsync
ShowMainWindowForActivatedRuntimeAsync
OpenInstalledPosFromInstallationV0Async
RetryStartupAssessmentAsync
InstallationV0Module
InstallationV0Window
MainWindow constructor and startup dependencies
runtime profile / station identity providers
```

Report the exact pre-change predicate, the exact object/property that was null or empty, and where the value was expected to be populated.

Do not infer from names alone.

## Required distinction

Determine which of these two defects exists:

### A. Duplicate gate

```text
LocalPosStartupService already proved Tenant/POS identity
but a later route checks station identity again
```

Required fix:

```text
remove the duplicate route check;
trust the Ready result and its local context;
```

### B. Required local identity payload was dropped

```text
LocalPosStartupService returned Ready
but the required local Tenant/POS/station identity object was not carried
into MainWindow construction or App runtime context
```

Required fix:

```text
populate one canonical local identity payload from the same local DB/runtime profile;
carry it through the existing result/context;
use it for MainWindow;
do not query API or Pairing Code;
do not create a second identity service.
```

The report must state whether the defect was A, B, or both.

## Identity source policy

The canonical local identity source must be the existing local runtime data already proven by the diagnostics, such as the current runtime profile/local POS records.

Allowed local fields include the existing equivalents of:

```text
TenantGuid
PosGuid
PosName
PosSlot
SourceClientId
RuntimeState
```

Do not print private raw GUIDs in the coordination report. Use sanitized presence/consistency booleans.

Do not use API claims, WpfJwt, protected hello, Pairing Code, environment variables, or employee PIN as a replacement identity source.

## One-decision rule

After this prompt:

```text
LocalPosStartupService
```

must remain the only owner of local runtime readiness and local Tenant/POS consistency.

The MainWindow transition may verify that a required payload object is structurally present, but it must not independently reassess installation/API proof or generate a contradictory route after `LocalPosStartupResult.Ready`.

Avoid logic like:

```csharp
if (localResult.IsReady)
{
    // second unrelated identity gate here
}
```

Prefer:

```csharp
if (!localResult.IsReady)
{
    return installationOrRecovery;
}

return await ShowMainWindowAsync(localResult.LocalIdentity);
```

If MainWindow does not actually require a station identity payload, remove that parameter/gate rather than fabricating one.

## Direct runtime and handoff must remain identical

Both paths must use the same service instance and the same identity payload:

```text
Direct runtime
-> ILocalPosStartupService
-> Ready + local identity
-> shared MainWindow transition

InstallationV0 Open OBM-POS
-> same ILocalPosStartupService
-> Ready + same local identity shape
-> same shared MainWindow transition
```

No `new LocalPosStartupService()` fallback may reappear.

## Read-only physical identity proof

Source inspection is mandatory.

When needed, perform a strictly read-only PostgreSQL check against `obm_pos_dev_v0_pg` using the configured runtime path:

```text
BEGIN TRANSACTION READ ONLY;
inspect only sanitized presence/count/consistency metadata;
ROLLBACK;
```

Allowed evidence:

```text
runtime profile count
runtime state
TenantGuid present boolean
PosGuid present boolean
matching local POS row count
```

Do not print GUID values, connection strings, password/passfile contents, employee/customer rows, or business data.

Do not mutate DB.

## Required tests

Add focused tests proving at least:

```text
Ready + Activated + consistent Tenant/POS identity
-> OPEN_POS_MAINWINDOW_SHOWN

Ready result with required local identity payload
-> MainWindow transition receives the payload

InstallationV0 handoff and direct runtime use the same identity shape/path

API HTTP 401 + Ready local runtime
-> MainWindow opens

API unavailable + Ready local runtime
-> MainWindow opens

no active branch emits POS_RUNTIME_ROUTE_STATIONIDENTITYMISSING
when LocalPosReady=True and PosIdentityConsistent=True

real local identity absent/inconsistent
-> InstallationRequired or RecoveryRequired

no API/Pairing/PIN source is used to populate local station identity

prompt058 label
```

If the old result code is no longer semantically valid, remove it from active source and the naming guard should prevent its return.

## Build and verification

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~MainWindow|FullyQualifiedName~Offline|FullyQualifiedName~Database|FullyQualifiedName~Wiring|FullyQualifiedName~Identity" -v minimal
```

Run an active-source scan for:

```text
POS_RUNTIME_ROUTE_STATIONIDENTITYMISSING
```

Expected:

```text
ZERO_ACTIVE_MATCHES
```

Historical reports/evidence may retain the text.

## Build label

Set:

```text
Build label: prompt058
Window title: OBM InstallationV0 Phase 1/2 - prompt058
```

## Physical execution policy

Do not launch WPF automatically.

Do not redeem Pairing Code.

Leave physical retest to the operator:

```text
A. InstallationV0 prompt058
   -> Local POS Ready/Activated
   -> API remains HTTP 401
   -> Open OBM-POS
   -> MainWindow visible

B. OBM-POS Runtime Development
   -> same local identity path
   -> MainWindow opens directly
```

## Mutation prohibitions

Do not:

```text
mutate PostgreSQL
run migration/seed
change DB roles/passwords/GRANTs
change API tokens/contracts
redeem Pairing Code
change employee PINs
set User/Machine environment variables
drop ASP.NET Identity tables
commit/push OBM source
```

## Documentation/history

Keep canonical architecture at V001.

Preserve the current `CURRENT_TASK.md` and `CURRENT_RESULT.md` under the next available versioned history folder before updating them.

After success, set the next task to physical launch/local CRUD/API-offline validation only.

## Report 058

Create and push:

```text
report/report058.md
```

Required sections:

1. Verdict.
2. DOCS_READ_BEFORE_CODE_GATE proof and hashes.
3. Physical prompt057 evidence.
4. Exact `STATIONIDENTITYMISSING` predicate/root cause.
5. Classification: duplicate gate, dropped payload, or both.
6. Canonical local identity source.
7. Exact source/result/context wiring correction.
8. Direct runtime path.
9. InstallationV0 handoff path.
10. Proof API 401 remains non-blocking.
11. Proof no second readiness/identity owner remains.
12. Deleted result codes/branches/files if any.
13. Read-only DB evidence if used.
14. Exact files changed.
15. Build/test commands and counts.
16. Active-source station-identity result scan.
17. Prompt058 label proof.
18. Updated CURRENT_TASK/CURRENT_RESULT history and hashes.
19. No DB/API/PIN/process/source-push mutation proof.
20. Exact operator physical retest steps.
21. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_LOCAL_STATION_IDENTITY_WIRING_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```

```text
BLOCKED_LOCAL_STATION_IDENTITY_SOURCE_MISSING
```

```text
BLOCKED_LOCAL_STATION_IDENTITY_BUILD_OR_TEST
```
