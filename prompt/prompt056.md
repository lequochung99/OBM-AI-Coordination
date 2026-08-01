# Prompt 056 — Canonical-gated deletion and renaming of misleading InstallationV0/runtime services

## Authoritative prerequisite

Prompt055 completed the first canonical documentation and repository instruction gate:

```text
report/report055.md
Verdict: OBM_POS_CANONICAL_V001_AND_CODE_GATE_READY
```

This prompt performs the approved next task already recorded in:

```text
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
```

The purpose is semantic/source cleanup only:

```text
- delete unused legacy/compatibility services and aliases;
- rename active services, interfaces, result models, enum members, physical files,
  DI registrations, tests, comments, and current documentation references;
- preserve the canonical behavior exactly;
- make future AI/source search unable to mistake API/bootstrap/PIN concepts for
  normal local POS startup authentication.
```

Do not add a new router, compatibility layer, context object, wrapper, security flag, password system, or startup condition.

## Phase 0 — mandatory read-before-code gate

Before editing any implementation/test/project/config file, read completely:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report054.md
report/report055.md
```

Read the evidence files needed for deletion decisions:

```text
E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\installation-flow-subgraph.json
E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\edge-inventory.csv
E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\symbol-action-inventory.csv
E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\redundant-link-candidates.md
E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\dynamic-edge-checklist.md
```

Before the first source edit, compute and record:

```text
CanonicalDocVersion=V001
CanonicalDocSha256=A70BE08FE143A34775A1F71844B7A6B672DDD28C24CB041B841E0604585B3915
AgentsSha256=20CB5060FC42ADDA2DE5EEBF90F4630BFBFCAA9ED195615E0B5B8DA035905114
CurrentTaskSha256=2871956A986F46E96F2C00BF0FAD7B0DE6DCEA02EEC0871159BDB52AED33A652
CurrentResultSha256=7E3093B8E9F4E59B55705D564C71BCEF73F363D5044A792927E366086E3B95F0
```

Required gate result:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
```

If any current hash differs, read the changed documents, report the new hashes and determine whether the task is still authorized. If the current canonical contract no longer authorizes this cleanup, stop before source edits with:

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```

## Canonical behavior that must not change

Normal runtime:

```text
Local PostgreSQL usable
-> open MainWindow
-> initialize API session afterward
```

API state:

```text
Token valid
-> Online

Token expired / HTTP 401 / API unavailable
-> MainWindow remains open
-> local CRUD remains available
-> API is Offline or Reauthorization Required
```

Credentials and PIN:

```text
Database = obm_pos_dev_v0_pg
Normal runtime PostgreSQL username = hung
PostgreSQL password = protected local infrastructure credential
TblEmployee.LoginNumber = employee operational PIN
```

The operational PIN is not an application password, PostgreSQL credential, API credential, installation proof, or startup credential.

Phase 1/Phase 2 detailed proof remains installation/verification only and must not become ordinary runtime authentication.

## No canonical architecture change

Canonical V001 already specifies the target architecture and names. This prompt must make source conform to V001.

Do not modify `INSTALLATION_RUNTIME_CANONICAL.md` unless source evidence proves the V001 target is impossible or unsafe. In that case, stop before implementation and request a separately versioned canonical V002 documentation task.

Do not silently change architecture and then rewrite V001 afterward.

## Phase 1 — exact symbol and external-contract audit

Before rename/delete, inventory all definitions and references using:

```text
rg/source search
C# symbol/reference inspection
DI registrations
factories
XAML/resource references
reflection/Activator/Type.GetType
configuration/JSON type names
public API/binary compatibility
serialization contracts
PowerShell/build scripts
Graphify graph evidence
```

For every candidate, record:

```text
definition file
physical file name
active production callers
installation-only callers
recovery/updater callers
DI registrations
factories
reflection/XAML/config references
tests-only references
external/public contract evidence
final action
```

Valid final actions:

```text
DELETE
RENAME
MOVE_AND_RENAME
MERGE
KEEP_INSTALLATION_ONLY
KEEP_RECOVERY_ONLY
KEEP_DEVELOPMENT_SAFETY_ONLY
BLOCKED_EXTERNAL_CONTRACT
```

A tests-only reference is not sufficient reason to retain a compatibility shim. Update the test and delete the shim.

## Phase 2 — required normal-startup cleanup

### A. Local POS startup service and result names

Required final active names and matching physical files:

```text
Services\Startup\LocalPosStartupService.cs
Services\Startup\LocalPosStartupResult.cs
Services\Startup\LocalPosStartupDecision.cs
```

Remove/rename these old names from active source:

```text
ApplicationStartupCoordinator
DatabaseStartupAssessmentService
DatabaseStartupAssessment
DatabaseStartupMode
InstalledHealthy
BootstrapRepairRequired
POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED
RepairBootstrap
NeedsBootstrapRepair
```

Required handling:

- `DatabaseStartupAssessmentService` compatibility subclass: update callers/tests to `LocalPosStartupService`, then delete the subclass and old physical file.
- `DatabaseStartupAssessment`: rename to `LocalPosStartupResult` unless a proven external serialized/public contract blocks it.
- `DatabaseStartupMode`: rename to `LocalPosStartupDecision` and use practical values such as `Ready`, `InstallationRequired`, `RecoveryRequired`, `Failed`.
- `InstalledHealthy`: rename to `Ready`.
- `ApplicationStartupCoordinator`: delete if the Phase 1 graph/source audit confirms no required recovery/updater caller. If a required non-runtime caller remains, rename it to that exact domain; it must not retain generic startup ownership.
- Generic BootstrapRepair result/route names must disappear from active normal runtime.

Do not retain aliases solely for source compatibility within this repository.

### B. API session names

Required final active names and physical files:

```text
ConnectService\IApiSessionInitializer.cs
ConnectService\ApiSessionInitializer.cs
```

Remove active names/files:

```text
IAppJwtBootstrapper
AppJwtBootstrapper
ConnectService\IAppJwtBootstrapper.cs
ConnectService\AppJwtBootstrapper.cs
```

Update:

```text
DI registration
constructor parameters
fields
method/local names
logs/comments
focused tests
```

The API initializer must still execute only after MainWindow is visible and must never change the successful local startup result.

### C. Installation verification name

Required final name and physical file:

```text
InstallationV0\Application\InstallationV0VerificationService.cs
```

Rename/delete the old active name:

```text
InstallationV0CompletedReadinessService
```

Update all installation/audit callers and tests.

This service must remain installation/verification-only. No normal runtime caller is allowed.

### D. Ambiguous Development/recovery names

Audit:

```text
RuntimeProfileStartupAssessmentService
LaunchProvenanceContext
EffectiveProductRootContext
```

Rules:

- Delete if unused after cleanup.
- If required only for Development safety, rename so both `Development` and `Safety` are explicit.
- If required only for recovery/updater workflows, rename to the exact recovery/updater responsibility.
- Do not let these types independently decide normal MainWindow eligibility.
- Do not create another compatibility alias.

### E. Route result duplication

Audit whether `PosStartupRouteResult` duplicates the renamed `LocalPosStartupResult`.

- If it only carries MainWindow visibility evidence for InstallationV0 handoff, keep it with a precise name such as `MainWindowOpenResult`.
- If it duplicates local startup decision ownership, merge/delete the duplicate.
- There must be one owner of local DB readiness and one owner of window transition evidence, not two competing route engines.

Any rename must match the physical filename.

## Phase 3 — physical file, project, DI, and test cleanup

After symbol changes:

```text
- rename/move physical files so public type and filename match;
- update explicit Compile/project references when present;
- update namespaces/usings;
- update DI registrations and resolutions;
- update tests to final names;
- delete obsolete tests that only validate removed shims;
- delete obsolete comments and result-code formatters;
- remove stale aliases/adapters/factories;
- remove active references from current docs/comments;
- do not edit historical evidence/report files merely to erase old names.
```

Search for stale backup source files such as:

```text
App.xaml.cs.codex-bak-*
*.bak
*.old
```

Do not let backup files participate in build, Graphify active-source reasoning, or naming guards. Preserve or move them only under an explicit evidence/archive location if they contain unique user work; otherwise report them as operator cleanup candidates. Do not delete uncertain user backups automatically.

## Phase 4 — naming and architecture regression guard

Add a focused automated guard under the WPF tests or a repository validation utility.

Scan active source and current canonical docs, excluding:

```text
bin
obj
.git
RecoveryReports
docs\refactoryInstallation\history
coordination reports
migrations/snapshots where historical names are immutable evidence
explicit archive/evidence folders
```

Fail when active source/current docs contain forbidden names:

```text
ApplicationStartupCoordinator
DatabaseStartupAssessmentService
DatabaseStartupAssessment
DatabaseStartupMode
AppJwtBootstrapper
IAppJwtBootstrapper
InstallationV0CompletedReadinessService
BootstrapRepairRequired
POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED
RepairBootstrap
NeedsBootstrapRepair
```

Also fail if current canonical docs contain:

```text
employee password
manager password
```

Use:

```text
employee operational PIN
manager/non-Staff operational PIN
```

The guard must not flag preserved historical reports/evidence.

## Phase 5 — Graphify/source proof after cleanup

Create the next available evidence folder:

```text
E:\Project2026\RecoveryReports\InstallationV0\ServiceNamingCleanupV001
```

Use V002/V003 if needed; never overwrite.

Without installing/upgrading Graphify, run the installed structural extractor after cleanup and preserve:

```text
README.md
SHA256SUMS.txt
graphify\...
final-symbol-inventory.csv
forbidden-name-scan.txt
direct-runtime-path.mmd
installation-handoff-path.mmd
```

Prove:

```text
- direct runtime reaches LocalPosStartupService -> MainWindow;
- API session edge is post-MainWindow;
- InstallationV0 handoff reaches the same LocalPosStartupService;
- InstallationV0VerificationService has no normal-runtime caller;
- removed compatibility names have zero active-source definitions/callers;
- dynamic/reflection/XAML/config checks do not reveal hidden old-name use.
```

Graphify absence is not expected, but source proof remains mandatory if Graphify execution fails.

## Phase 6 — documentation pointers and version preservation

Because architecture and canonical target do not change, keep:

```text
CanonicalDocVersion=V001
```

Before replacing `CURRENT_TASK.md` or `CURRENT_RESULT.md`, preserve their prompt055 versions under the next available history folder, for example:

```text
docs\refactoryInstallation\history\V001\CURRENT_TASK_V001.md
docs\refactoryInstallation\history\V001\CURRENT_RESULT_V001.md
```

Do not overwrite an existing history folder/file.

After successful cleanup:

- `CURRENT_RESULT.md` records naming cleanup completion, final names, builds/tests, and pending physical retest.
- `CURRENT_TASK.md` sets the next task to physical MainWindow/local CRUD/API-offline retest; it must not authorize ASP.NET Identity deletion yet.
- `README.md` continues to point to canonical V001.
- Update current docs only with final source names; do not alter the runtime architecture.
- Recompute and report hashes.

## Behavior and mutation prohibitions

This prompt must not:

```text
mutate PostgreSQL
run migration/seed
change DB role/password/GRANT/REVOKE
redeem Pairing Code
change API tokens/contracts
implement refresh tokens
change employee PIN values or validation rules
drop ASP.NET Identity tables
migrate outbox data
set User/Machine environment variables
launch WPF automatically
commit/push OBM source
```

Source cleanup, documentation pointer updates, external evidence creation, builds, and tests are allowed.

## Required builds and tests

Run sequentially:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~MainWindow|FullyQualifiedName~Offline|FullyQualifiedName~Database|FullyQualifiedName~Naming|FullyQualifiedName~Documentation" -v minimal
```

Also run exact forbidden-name scans and report zero active matches or justified blocked matches.

## Build label

If InstallationV0 source changes, set:

```text
Build label: prompt056
Window title: OBM InstallationV0 Phase 1/2 - prompt056
```

## Physical execution policy

Do not launch WPF automatically.

Final physical retest remains with the operator after build/test PASS:

```text
A. OBM-POS Runtime Development
   -> connect local DB as hung
   -> MainWindow opens directly

B. InstallationV0 -> Open OBM-POS
   -> same LocalPosStartupService
   -> MainWindow opens

C. API HTTP 401/unavailable
   -> MainWindow remains open
   -> local CRUD works
```

## Report 056

Create and push:

```text
report/report056.md
```

Required sections:

1. Verdict.
2. `DOCS_READ_BEFORE_CODE_GATE` proof with V001 paths and hashes.
3. External-contract/reference audit.
4. Old-name to final-name/action table.
5. Deleted files/types/interfaces/enum members/result codes/DI registrations/tests.
6. Renamed/moved files/types/interfaces/result models.
7. Compatibility shims retained, with proven external reason; expected normally none.
8. Final direct runtime path.
9. Final InstallationV0 handoff path.
10. Proof API initializer is post-MainWindow.
11. Proof installation verification has no normal-runtime caller.
12. Runtime PostgreSQL username source remains `hung` without secret disclosure.
13. Naming/architecture regression guard.
14. Graphify/source post-cleanup evidence folder and hashes.
15. Canonical V001 unchanged or explicit blocker requiring V002.
16. History preservation for previous CURRENT_TASK/CURRENT_RESULT.
17. Updated CURRENT_TASK/CURRENT_RESULT paths and hashes.
18. Exact source/docs/tests/project files changed/deleted/moved.
19. Build/test commands and counts.
20. Prompt056 label proof.
21. No DB/API/PIN/process/source-push mutation proof.
22. Exact operator physical retest steps.
23. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_SERVICE_NAMES_AND_LEGACY_SHIMS_CLEAN_READY_FOR_USER_RETEST
```

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```

```text
BLOCKED_SERVICE_NAMING_CLEANUP_EXTERNAL_CONTRACT
```

```text
BLOCKED_SERVICE_NAMING_CLEANUP_BUILD_OR_TEST
```
