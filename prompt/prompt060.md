# Prompt 060 — Close the final station-enabled/mirror predicate; no post-MainWindow repair modal

## Physical operator evidence

The operator physically launched the current prompt059 build.

The real MainWindow opens, but one modal remains:

```text
POS Station Assignment Repair
The local station mirror references a POS station that is not enabled for this local tenant: <sanitized PosGuid>.
Open Advance Setting > POS Station to repair this machine's station assignment.
```

This is now the only visible startup defect.

Prior physical and source evidence already proves:

```text
Local PostgreSQL authentication succeeded
SchemaReady=True
RuntimeProfileCount=1
RuntimeState=Activated
TenantIdentityConsistent=True
PosIdentityConsistent=True
LocalPosReady=True
MainWindow opens
API HTTP 401 is non-blocking
```

The GUID shown by the remaining modal has the same shape/value role as the canonical local `PosGuid`; do not print it raw in the report.

The remaining problem is therefore one of these exact classes:

```text
A. the canonical station row is valid, but a legacy “enabled” predicate/query is wrong;
B. the mirror/cache is read through the wrong tenant/path/source;
C. TblPosLocal has an installation-time enabled/active/status default that was not materialized;
D. the local station rows are genuinely ambiguous/inconsistent.
```

Do not assume which class applies. Prove it.

## Critical boundary

This is POS-station state, not ASP.NET Identity, not employee operational PIN, not PostgreSQL authentication, and not API authentication.

Do not create another station service, identity service, router, compatibility layer, or password/security abstraction.

## Mandatory documentation gate

Before source/test/config/documentation edits, read completely:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report057.md
report/report059.md
```

Read the station evidence:

```text
E:\Project2026\RecoveryReports\InstallationV0\StationAssignmentCleanupV001\README.md
E:\Project2026\RecoveryReports\InstallationV0\StationAssignmentCleanupV001\station-source-inventory.csv
E:\Project2026\RecoveryReports\InstallationV0\StationAssignmentCleanupV001\pre-change-station-flow.mmd
E:\Project2026\RecoveryReports\InstallationV0\StationAssignmentCleanupV001\post-change-station-flow.mmd
E:\Project2026\RecoveryReports\InstallationV0\StationAssignmentCleanupV001\modal-callsite-scan.txt
E:\Project2026\RecoveryReports\InstallationV0\StationAssignmentCleanupV001\runtime-consumer-hydration.csv
E:\Project2026\RecoveryReports\InstallationV0\StationAssignmentCleanupV001\graphify\graphify-out\graph.json
```

Before the first implementation edit record:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V001
CanonicalDocSha256=<actual current hash>
```

If canonical V001 no longer authorizes local-DB-first startup with API independence, stop before source edits.

## Objective

For a single unambiguous canonical local station:

```text
Activated TblPosRuntimeProfile
+ matching TblTenant
+ matching TblPosLocal
+ matching TenantGuid/PosGuid
+ canonical LocalPosStartupResult.LocalIdentity
```

normal startup must produce:

```text
MainWindow visible
no POS Station Assignment Repair modal
no automatic Advanced Settings navigation
no second station-readiness decision after MainWindow
```

If the local data is genuinely inconsistent, the app must return a precise local installation/recovery result before business UI is shown. It must not open MainWindow and then show a modal.

## Phase 1 — locate the exact remaining modal and caller

Search the exact current wording and every call path:

```text
POS Station Assignment Repair
local station mirror references a POS station that is not enabled
not enabled for this local tenant
Open Advance Setting > POS Station
```

Report:

```text
exact file/method/line
caller chain from App/MainWindow/business initialization
call timing relative to MainWindow.Show and Loaded
predicate
query/filter expression
all fields used by the enabled test
mirror path used
TenantGuid source
PosGuid source
whether the call is automatic or explicit repair UI
```

Use Graphify plus source/reference inspection. The exact remaining automatic caller must be identified; a source scan of only older modal strings is insufficient.

## Phase 2 — read-only physical DB and mirror audit

Use the approved protected local credential path without printing secrets.

Inspect `obm_pos_dev_v0_pg` only inside:

```text
BEGIN TRANSACTION READ ONLY;
...
ROLLBACK;
```

Audit sanitized metadata for the canonical identity only:

```text
TblPosRuntimeProfile row count
RuntimeState
TenantGuid/PosGuid equality booleans
PosName/slot presence booleans
SourceClientId shape
TblTenant matching row count and status fields
TblPosLocal matching row count
all TblPosLocal active/enabled/disabled/deleted/status columns and values
foreign-key relationship booleans
other local station rows for the same tenant, count only
```

Do not print private GUIDs, business rows, credentials, or connection strings.

Inspect every active `pos-station.json`/station mirror path read by runtime. Record only:

```text
path
exists
TenantGuid/PosGuid equality booleans
field names
last-write timestamp
whether it matches canonical local context
```

Do not print raw GUID values.

## Phase 3 — classify the exact root cause

The report must choose exactly one primary classification:

```text
LEGACY_ENABLED_PREDICATE_FALSE_NEGATIVE
MIRROR_TENANT_OR_PATH_MISMATCH
INSTALLATION_TBLPOSLOCAL_STATE_NOT_MATERIALIZED
GENUINE_LOCAL_STATION_AMBIGUITY
OTHER_PROVEN_CAUSE
```

Include the exact pre-change predicate and evidence.

### Rules

- Do not bypass a genuinely authoritative disabled state.
- Do not treat a legacy multi-POS feature flag as station disablement unless source/schema proves that is its intended canonical meaning.
- `RuntimeState=Activated` alone must not hide multiple or mismatched station rows.
- A stale mirror must not override one valid canonical local DB station.
- No POS1/GUID hard-coding.

## Phase 4 — minimal correction by classification

### A. Legacy enabled predicate false negative

If the canonical DB/profile station is valid and the remaining query uses an obsolete or unrelated enable flag:

```text
- remove that legacy predicate from normal runtime station resolution;
- use the exact canonical local station row/profile match;
- keep the legacy flag only in the feature that actually owns it, if still required;
- delete the automatic modal caller;
- retain explicit repair diagnostics only.
```

### B. Mirror tenant/path mismatch

If the DB is valid but runtime reads another mirror/path/tenant:

```text
- collapse normal runtime to one ProductRoot-local mirror path;
- hydrate/repair the mirror from LocalPosStartupResult.LocalIdentity;
- remove legacy fallback paths from normal runtime;
- retain old paths as migration/import evidence only when required;
- do not let the mirror block MainWindow.
```

### C. Installation TblPosLocal state not materialized

If the single canonical `TblPosLocal` row exists but an authoritative enabled/active/status field is missing or false because the installation materialization omitted it:

```text
- fix the Phase 2 identity-spine/baseline materialization for all future installations;
- add an idempotent, versioned local station-state repair/upgrade implementation;
- do not silently ignore the field in normal runtime;
- do not physically mutate PostgreSQL in this prompt;
- leave the physical repair action for an explicit operator-authorized follow-up after source/build/test proof.
```

Return verdict:

```text
OBM_POS_LOCAL_STATION_STATE_REPAIR_READY_FOR_OPERATOR_APPLY
```

and include the exact one-row/field repair plan, without raw GUIDs or SQL containing secrets.

### D. Genuine ambiguity

If multiple or conflicting station rows exist:

```text
- fail before MainWindow with a precise RecoveryRequired result;
- do not auto-select;
- do not show a post-MainWindow modal;
- report the safe row counts and conflicting fields.
```

Return:

```text
BLOCKED_STATION_ASSIGNMENT_SOURCE_AMBIGUOUS
```

## Phase 5 — startup/window behavior

After correction, automatic startup must have only these outcomes:

```text
canonical station valid
-> MainWindow
-> no station modal

canonical station invalid/ambiguous
-> Installation or Recovery before MainWindow
-> exact safe result code
```

The following is forbidden:

```text
MainWindow visible
-> automatic POS Station Assignment Repair modal
```

Advanced Settings > POS Station remains explicit repair/manual reassignment UI only.

## Phase 6 — tests

Add focused tests for:

```text
exact physical classification and corrected predicate
single Activated canonical station -> no repair modal
valid DB station + matching mirror -> no repair modal
valid DB station + missing mirror -> mirror repaired/no modal
valid DB station + stale legacy-path mirror -> canonical ProductRoot mirror wins/no modal
legacy multi-POS/feature flag does not disable canonical station unless explicitly authoritative
single canonical row with authoritative disabled field -> versioned repair required, not bypassed
multiple rows -> RecoveryRequired before MainWindow
wrong tenant mirror -> canonical DB context wins or recovery, never post-MainWindow modal
API HTTP 401/unavailable -> no effect
exact remaining modal text has zero automatic normal-runtime callers
Advanced Settings repair command may retain explicit repair text
prompt060 label
```

## Phase 7 — Graphify/evidence

Create the next available folder:

```text
E:\Project2026\RecoveryReports\InstallationV0\StationEnabledClosureV001
```

Never overwrite an existing version.

Preserve:

```text
README.md
SHA256SUMS.txt
graphify\...
remaining-modal-call-chain.mmd
station-enabled-predicate-before-after.md
physical-db-station-state.txt
mirror-path-inventory.csv
automatic-modal-scan.txt
```

No secrets or raw private GUIDs.

Prove that the automatic normal-runtime path no longer reaches the modal, or document the versioned repair blocker.

## Build label

Set:

```text
Build label: prompt060
Window title: OBM InstallationV0 Phase 1/2 - prompt060
```

## Required builds/tests

Run sequentially:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~MainWindow|FullyQualifiedName~Database|FullyQualifiedName~Station|FullyQualifiedName~Naming|FullyQualifiedName~Wiring" -v minimal
```

## Documentation updates

Canonical V001 architecture remains unchanged.

Preserve current task/result under the next versioned history folder before updating current docs.

Update `CURRENT_RESULT.md` with the proven classification and correction.

Update `CURRENT_TASK.md` to physical MainWindow/local CRUD/API-offline retest when source-only correction is complete, or to the explicit local station-state repair action when classification C applies.

Do not authorize ASP.NET Identity deletion yet.

## Prohibited actions

Do not:

```text
mutate PostgreSQL
run seed/migration physically
redeem Pairing Code
change API tokens/contracts
change employee PINs
change DB roles/passwords/GRANTs
hard-code station identity
launch WPF automatically
commit/push OBM source
drop ASP.NET Identity tables
```

## Report 060

Create and push:

```text
report/report060.md
```

Required sections:

1. Verdict.
2. DOCS_READ_BEFORE_CODE_GATE proof.
3. Physical remaining-modal evidence.
4. Exact remaining modal callsite/caller chain.
5. Read-only DB station-state evidence.
6. Mirror path/source evidence.
7. Root-cause classification.
8. Exact pre-change predicate.
9. Minimal correction or versioned repair plan.
10. Final startup outcomes and no-post-MainWindow-modal proof.
11. Exact files changed.
12. Builds/tests counts.
13. Graphify/evidence folder/hashes.
14. Prompt060 label proof.
15. No DB/API/PIN/process/source-push mutation proof.
16. Operator physical retest or explicit repair steps.
17. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_FINAL_STATION_ENABLED_GATE_CLOSED_READY_FOR_PHYSICAL_RETEST
```

```text
OBM_POS_LOCAL_STATION_STATE_REPAIR_READY_FOR_OPERATOR_APPLY
```

```text
BLOCKED_STATION_ASSIGNMENT_SOURCE_AMBIGUOUS
```

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```

```text
BLOCKED_STATION_ENABLED_CLOSURE_BUILD_OR_TEST
```
