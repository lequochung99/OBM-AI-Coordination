# Prompt 059 — Remove the legacy post-MainWindow POS-station assignment gate and hydrate canonical local station context

## Physical operator evidence

The operator physically launched the current OBM-POS build and reached the real MainWindow.

Immediately after MainWindow became visible, two legacy-style modal dialogs appeared:

```text
POS Station Required
Saved POS station was not found or is POS-disabled for this tenant: <sanitized-guid>.
Open Advance Setting > POS Station and choose this machine's station.
```

and:

```text
POS Station Identity Mismatch
This computer's saved station assignment does not match runtime identity.
Open Advance Setting > POS Station and assign this computer again.
```

This is authoritative physical evidence.

The current local installation had already been proven before MainWindow:

```text
LocalDatabaseAuthenticationSucceeded=True
SchemaReady=True
RuntimeProfileCount=1
RuntimeState=Activated
TenantIdentityConsistent=True
PosIdentityConsistent=True
LocalPosReady=True
```

Therefore the post-MainWindow dialogs are not proof that the local installation lacks a station identity. They prove that old MainWindow/runtime consumers still read a different saved-assignment source or repeat a legacy station-selection gate after the canonical local startup service already validated the POS station.

## Critical terminology correction

Do not confuse these dialogs with ASP.NET Identity.

```text
POS station identity / station assignment
!= ASP.NET Identity
!= employee password
!= employee operational PIN
!= API authentication
```

The POS station concept is still required: every physical machine must know which local POS station it represents.

What is legacy/redundant is the separate post-MainWindow manual-assignment path and duplicate gate when the canonical local runtime already supplies a valid station.

## Mandatory documentation gate

Before editing source/tests/config/docs, read completely:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report056.md
report/report057.md
```

Also read the post-cleanup graph evidence:

```text
E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\direct-runtime-path.mmd
E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\installation-handoff-path.mmd
E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\final-symbol-inventory.csv
E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001\graphify\graphify-out\graph.json
```

Required report evidence before the first source edit:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V001
CanonicalDocSha256=<actual current hash>
```

If canonical V001 no longer authorizes local-DB-first startup and canonical station hydration, stop before source edits.

## Canonical station identity model

Audit the exact current schema and source, but preserve these established semantics:

```text
TblPosRuntimeProfile.PosGuid = canonical local runtime POS station identity
TblPosLocal.PosGuid          = canonical local station row identity
PosGuid                      != device identity
PosGuid                      != TenantGuid
PosName                      = display name
SourceClientId               = POS:{PosGuid:D}
pos-station.json             = local machine mirror/cache when still required
```

The canonical local database/profile is authoritative for an installed Activated runtime.

A stale or missing local JSON/cache must not override a single valid canonical local DB station row/profile.

Do not hard-code POS1 or any GUID.

## Exact objective

Make the real MainWindow consume the same canonical station context that `LocalPosStartupService` already validated.

Required flow:

```text
LocalPosStartupService
-> LocalPosStartupResult.Ready
-> canonical local station context carried forward
-> hydrate MainWindow/runtime station consumers
-> show MainWindow
-> no station-required/mismatch modal
```

There must be one station assignment authority for normal installed runtime.

## Phase 1 — locate both modal call sites and all station-assignment sources

Search exact text and related symbols:

```text
POS Station Required
Saved POS station was not found
POS-disabled
POS Station Identity Mismatch
saved station assignment
Advance Setting > POS Station
CurrentPosGuid
pos-station.json
CompanyInfo.PosGuid
CompanyInfo.PosName
CompanyInfo.SourceClientId
TblPosLocal
TblPosRuntimeProfile
IsActive
IsEnabled
PosGuid
PosName
SourceClientId
```

For each modal, report:

```text
exact file/method/line
call timing relative to MainWindow.Show
predicate that triggers it
identity source used
saved-assignment source used
whether it reads DB, JSON, static state, settings table, environment, or API
whether it mutates anything
all callers
```

Use Graphify/source reference inspection to map every path from MainWindow construction/Loaded to these dialogs.

## Phase 2 — audit and classify the competing station sources

Inventory every active source of current-machine station assignment:

```text
TblPosRuntimeProfile
TblPosLocal
pos-station.json under every possible path
ProductRoot machine state
%LOCALAPPDATA% legacy path
CompanyInfo/static properties
setup/settings tables
launch profile/environment
API/Platform response/checkpoint
Advanced Settings POS Station UI
```

For each source classify:

```text
CANONICAL_DB_AUTHORITY
LOCAL_MIRROR_CACHE
INSTALLATION_PROOF_ONLY
REPAIR_UI_ONLY
LEGACY_REDUNDANT
UNKNOWN
```

Record exact precedence before and after correction.

## Phase 3 — required correction

### A. Carry canonical station context from local startup

Extend or use the existing local-only result/context so successful startup carries the sanitized station fields needed by runtime:

```text
TenantGuid
PosGuid
PosName
PosSlot when physically available
SourceClientId
RuntimeState
```

Do not introduce another station identity service if `LocalPosStartupResult` can carry the existing verified context.

Do not place secrets, tokens, employee PINs, or DB passwords in the result.

### B. Hydrate legacy runtime consumers once

Before or during MainWindow construction, set the existing runtime fields that old business code still needs, for example after source proof:

```text
CompanyInfo.PosGuid
CompanyInfo.PosName
CompanyInfo.SourceClientId
other current-station static/runtime fields actually read by business modules
```

All values must come from the canonical local DB/profile context.

Do not read API claims or WpfJwt for station identity.

### C. Remove the duplicate normal-runtime gate

When canonical local station context is valid:

```text
- do not show POS Station Required;
- do not show POS Station Identity Mismatch;
- do not force Advanced Settings manual selection;
- do not re-run an independent station-validity decision after MainWindow opens.
```

Delete the duplicate gate/callers when no repair-only use remains.

If the dialogs are still useful for explicit repair mode, move them behind an explicit repair/diagnostics action and rename them to `POS Station Assignment Repair`, not generic `Identity` language.

### D. Local mirror/cache behavior

Audit `pos-station.json` and all machine-assignment files.

For an installed Activated runtime with one canonical valid station:

```text
canonical DB/profile wins
```

If a mirror is required by legacy business code, repair/write it from canonical context using the existing approved local-machine-state mechanism. Do not let a stale mirror block MainWindow.

Do not silently overwrite an ambiguous mirror when multiple local station rows or identity inconsistency exists; route to explicit recovery instead.

No PostgreSQL mutation is allowed in this prompt.

### E. Advanced Settings behavior

`Advanced Settings > POS Station` may remain as:

```text
repair/diagnostic/manual reassignment UI
```

It must not be required during ordinary startup after canonical installation.

If manual reassignment remains possible, it must update the same canonical local assignment contract and must not create a second competing source of truth.

Do not test manual reassignment physically in this prompt.

## Phase 4 — message and GUID correctness

The physical dialog text appears to label a station GUID as a tenant value. Audit this exactly.

Ensure future diagnostics distinguish:

```text
TenantGuid
PosGuid
DeviceGuid
InstallationGuid
```

Do not print raw private GUIDs in reports. Use sanitized shape or equality booleans.

Normal successful startup should not display identity diagnostics.

## Phase 5 — tests

Add focused tests for at least:

```text
Activated local runtime + matching TblPosRuntimeProfile/TblPosLocal
-> canonical station context returned

canonical station context
-> CompanyInfo/current-station runtime fields hydrated

canonical valid station + missing legacy pos-station.json
-> MainWindow allowed
-> no station-required modal

canonical valid station + stale mismatching legacy pos-station.json
-> canonical DB wins or mirror is safely repaired
-> no mismatch modal

multiple/ambiguous local station rows
-> explicit recovery result
-> no silent POS1 fallback

runtime PosGuid != TblPosLocal PosGuid
-> recovery/fail closed before business UI

PosGuid remains distinct from TenantGuid and DeviceGuid

SourceClientId = POS:{PosGuid:D}

API HTTP 401/unavailable
-> does not affect station assignment

Advanced Settings station UI is not called during normal startup

exact legacy modal strings have zero active normal-runtime call sites

prompt059 label
```

## Phase 6 — Graphify and naming guard

Create the next available evidence folder:

```text
E:\Project2026\RecoveryReports\InstallationV0\StationAssignmentCleanupV001
```

Use V002/V003 if necessary; never overwrite.

Preserve:

```text
README.md
SHA256SUMS.txt
graphify\...
station-source-inventory.csv
pre-change-station-flow.mmd
post-change-station-flow.mmd
modal-callsite-scan.txt
runtime-consumer-hydration.csv
```

Prove:

```text
LocalPosStartupService -> canonical station context -> MainWindow/runtime consumers
no pre/post-MainWindow API dependency
no active normal-runtime modal call for station-required/mismatch
Advanced Settings POS Station is repair-only
```

Update the existing naming/architecture guard so active normal-runtime source fails if the exact legacy modal text or a new duplicate station-assignment gate is reintroduced, while repair-only diagnostics may use approved explicit terminology.

## Build label

Set:

```text
Build label: prompt059
Window title: OBM InstallationV0 Phase 1/2 - prompt059
```

## Prohibited actions

Do not:

```text
mutate PostgreSQL
run seed/migration
change DB roles/passwords
redeem Pairing Code
change API tokens/contracts
change employee PINs
hard-code POS1 or GUIDs
auto-select arbitrary station when local identity is ambiguous
drop ASP.NET Identity tables
commit/push OBM source
launch WPF automatically
```

Source/config/tests/docs/evidence changes needed for this correction are allowed.

## Required builds/tests

Run sequentially:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~MainWindow|FullyQualifiedName~Database|FullyQualifiedName~Station|FullyQualifiedName~Identity|FullyQualifiedName~Naming|FullyQualifiedName~Wiring" -v minimal
```

## Documentation updates

Canonical V001 architecture does not change.

Preserve the current `CURRENT_TASK.md` and `CURRENT_RESULT.md` under the next versioned history folder before updating them.

Update current docs to state:

```text
- canonical station authority is local DB/profile;
- local JSON/static fields are mirror/consumer state only;
- Advanced Settings station selection is repair-only;
- next task is physical MainWindow/local CRUD/API-offline retest.
```

Do not authorize ASP.NET Identity deletion yet.

## Report 059

Create and push:

```text
report/report059.md
```

Required sections:

1. Verdict.
2. DOCS_READ_BEFORE_CODE_GATE proof.
3. Physical two-dialog evidence.
4. Explicit statement: POS station identity is not ASP.NET Identity.
5. Exact modal call sites and trigger predicates.
6. Pre-change station-source inventory and precedence.
7. Canonical station authority and final precedence.
8. Duplicate gate removed or repair-only classification.
9. Canonical context fields carried through LocalPosStartupResult.
10. Runtime consumers hydrated.
11. pos-station.json/machine-mirror handling.
12. Advanced Settings repair-only proof.
13. TenantGuid/PosGuid/DeviceGuid terminology correction.
14. Exact files changed/deleted/moved.
15. Tests/build counts.
16. Graphify/evidence folder and hashes.
17. Prompt059 label proof.
18. No DB/API/PIN/process/source-push mutation proof.
19. Exact operator physical retest steps.
20. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_CANONICAL_STATION_ASSIGNMENT_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```

```text
BLOCKED_STATION_ASSIGNMENT_SOURCE_AMBIGUOUS
```

```text
BLOCKED_STATION_ASSIGNMENT_BUILD_OR_TEST
```
