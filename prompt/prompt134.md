# Prompt134 — Forensic reuse-first restoration of the V005 pairing-before-database installation flow

## Authority

Read completely before editing:

```text
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL_V005.md
report/report131.md
report/report132.md
report/report133.md if it exists when this prompt is executed
```

V005 is the current authority. The required order is:

```text
Test local PostgreSQL access
-> redeem Pairing Code
-> persist canonical Tenant/POS installation plan and credential
-> create/resume local DB
-> apply migrations
-> DatabaseReady
-> bootstrap POS1/POS2/replacement according to the API plan
-> ApplicationReady
-> MainWindow
```

## Operator direction

This flow was already implemented and working recently. Do not rebuild InstallationV0 or invent a new installer. First locate the previously working implementation and restore only the wiring or small deltas that were lost.

This is a forensic-reuse task first and a coding task second.

## Frozen work

Do not modify:

```text
Category Weight
Booking Weight
Price Weight
TblTenantPosDevice
destination routing
ongoing sync architecture
SignalR architecture
CompanionApp/payment-terminal architecture
Firebase/.env cleanup
refresh-token architecture
```

Do not drop, truncate, recreate, or copy any local PostgreSQL database.

Preserve `obm_pos_dev_v1_pg` as current installation evidence. It may be empty or partially initialized and must be resumed idempotently.

## Phase 0 — Find the previously working code before editing

Search all locally available evidence from the last two development days, including:

```text
git log --all
git reflog
uncommitted/staged changes if any
local branches and detached commits
Visual Studio backup/autorecovery files
RecoveryReports artifacts
InstallationV0 historical folders still present
prompt/report artifacts and diffs
source files modified by prompt117 through prompt133
```

Search specifically for prior working call chains involving:

```text
Pairing Code redeem before DB creation
Phase1InstallationService.TryResumeAsync
PlatformV0 pairing redeem response
protected checkpoint persistence
TenantGuid / PosGuid / PosStationId / InstallationAttemptId
CleanLocalDatabaseService.CreateCleanDatabaseAsync
schema/migration bootstrap
PostgreSqlPhase2ReferenceSeedExecutor
Phase2StartupHydrationService
PostgresPosRuntimeProfileRepository
LocalPosStartupService
MainWindow launch
```

Produce a forensic table before editing:

| Concern | Current active owner | Last-known working owner/version | Exact difference | Reuse decision |
|---|---|---|---|---|

Required evidence fields:

```text
LAST_WORKING_COMMIT_OR_ARTIFACT
LAST_WORKING_DATE_TIME
LAST_WORKING_PAIRING_CALL_CHAIN
LAST_WORKING_CHECKPOINT_FIELDS
LAST_WORKING_DB_CREATE_CALL_CHAIN
LAST_WORKING_MIGRATION_CALL_CHAIN
LAST_WORKING_BOOTSTRAP_CALL_CHAIN
CURRENT_BREAK_POINTS
MINIMAL_FILES_REQUIRED
```

Do not claim that no old implementation exists until git history, reflog, artifacts, and current source have all been checked.

## Phase 1 — Preserve and reuse existing owners

Reuse existing production owners whenever they exist:

```text
InstallationV0Window
existing PostgreSQL preflight UI and models
existing provisioning/runtime credential boundary
existing pairing UI and API redeem endpoint
existing protected checkpoint/token store
CleanLocalDatabaseService
canonical EF/Npgsql migration path
existing baseline/bootstrap service
PostgresPosRuntimeProfileRepository
LocalPosStartupService
existing MainWindow launch path
```

Forbidden:

```text
second installation window
second pairing flow
second checkpoint format
second DB settings store
second migration runner
second bootstrap endpoint
second startup coordinator
manual SQL schema creation
```

If restoring the previous code would require a new subsystem, stop with:

```text
BLOCKED_PROMPT134_REUSE_NOT_FOUND_REQUIRES_OPERATOR_REVIEW
```

## Phase 2 — Restore the exact V005 order with minimal wiring

### A. PostgreSQL preflight only

WPF collects and tests:

```text
host/port
provisioning user/password
runtime user/password
local DB name
```

At this point:

```text
no DB create
no migration
no seed
```

### B. Pairing before DB create

Use the existing PlatformApp/API chain:

```text
PlatformApp Tenant/POS1-POS10 selection
-> Pairing Code issuance
-> WPF redeem
-> API returns existing WpfJwt plus canonical installation identity/plan
-> WPF persists protected checkpoint
```

Audit the existing response/checkpoint fields first. Reuse equivalent fields instead of adding duplicates.

Minimum plan semantics required by V005:

```text
TenantGuid
PosGuid
PosStationId / POS slot
InstallationAttemptId
AttemptVersion or PlanVersion
InstallationMode
BootstrapMode
```

If some fields are already derivable from existing source contracts, document the mapping. Add only genuinely missing fields.

Local DB host/user/password/name remain WPF-local and must not be transported by PlatformApp or pairing payloads.

### C. Create or resume DB only after protected pairing checkpoint exists

For `obm_pos_dev_v1_pg`:

```text
if absent -> create once with provisioning credential
if existing and empty -> resume migration
if partially migrated -> resume migration
if business/user data exists with inconsistent state -> stop for Recovery review
```

Never drop or recreate automatically.

Use `postgres` only for create/owner/grant. Use `hung` for migration, bootstrap, and normal runtime. Do not persist the provisioning password.

### D. Apply migrations in-process

Use the attached canonical EF/Npgsql path already restored by prompt131.

No active dependency on:

```text
SpacePos.Provisioning.Schema
EnsureCreated
manual CREATE TABLE
```

Pending migrations must become zero.

### E. Runtime state

After schema is current:

```text
upsert TblPosRuntimeProfile = DatabaseReady
append DatabaseReady history
commit
```

Then apply the bootstrap branch selected by the API plan:

```text
New Tenant / POS1 -> existing approved minimal baseline
POS2-POS10 -> existing canonical tenant bootstrap/snapshot boundary
Reinstall/replacement -> reuse logical Tenant/POS identity and bootstrap canonical state
```

Do not invent a second sync/bootstrap pipeline. Installation bootstrap must create zero `TblLocalOutbox` rows.

Final transaction:

```text
apply Tenant/POS identity and approved baseline/snapshot
upsert same profile = ApplicationReady
append ApplicationReady history
commit
```

Only then open MainWindow.

## Phase 3 — Resume and compatibility requirements

Current ProductRoot may contain configuration/checkpoints written by previous prompts. Preserve source compatibility:

```text
legacy valid database-settings.json -> atomic upgrade through existing writer
existing valid protected pairing checkpoint -> resume same installation attempt
DB exists but schema empty -> resume migration
DatabaseReady -> resume bootstrap only
ApplicationReady -> MainWindow directly
```

API/token failure before ApplicationReady must preserve:

```text
local DB
Tenant/POS identity checkpoint
installation attempt identity
```

After ApplicationReady, API/token failure must only mark cloud Offline/Degraded.

## Phase 4 — Physical acceptance

Use visible label:

```text
prompt134
```

Physical proof must use the actual WPF UI and existing ProductRoot.

### Case 1 — PostgreSQL preflight

Prove valid provisioning/runtime credentials and safe DB name without creating or mutating the DB before pairing.

### Case 2 — Pairing plan

Create/redeem one Development Pairing Code for the intended Tenant/POS slot. Prove the protected checkpoint contains the canonical identity/plan and survives restart.

Do not print token or secret values.

### Case 3 — Existing V1 DB resume

Classify `obm_pos_dev_v1_pg` safely. If empty or partial, resume it without drop/recreate:

```text
migrations current
pending migrations = 0
TblPosRuntimeProfile exists
TblPosRuntimeStateHistory exists
DatabaseReady transition exactly once
approved bootstrap applied
ApplicationReady transition exactly once
installation outbox rows = 0
```

### Case 4 — MainWindow

Prove:

```text
MainWindow opens only after ApplicationReady
InstallationV0 closes
MainWindow responsive >= 60 seconds
```

Then stop API and launch WPF twice:

```text
restart 1 -> MainWindow directly
restart 2 -> MainWindow directly
InstallationV0 does not flash
local DB remains intact
cloud may be Offline/Degraded
```

## Scope expectation

Because the flow existed recently, production changes should be small and concentrated. Report exact changed files and explain why each was necessary.

If more than a narrow set of existing owners must be rewritten, stop before broad refactor with:

```text
BLOCKED_PROMPT134_SCOPE_EXPANSION_REQUIRES_OPERATOR_REVIEW
```

## PASS verdict

PASS only when all physical cases succeed:

```text
OBM_WPF_V005_EXISTING_PAIRING_FIRST_INSTALLATION_RESTORED_APPLICATIONREADY_MAINWINDOW_OFFLINE_PROVEN
```

Narrow blockers:

```text
BLOCKED_V005_PREVIOUS_IMPLEMENTATION_NOT_FOUND
BLOCKED_V005_PAIRING_PLAN_CONTRACT
BLOCKED_V005_CHECKPOINT_RESUME
BLOCKED_V005_EXISTING_EMPTY_DB_RESUME
BLOCKED_V005_MIGRATION
BLOCKED_V005_POS_BOOTSTRAP
BLOCKED_V005_APPLICATIONREADY
BLOCKED_V005_MAINWINDOW_PHYSICAL_PROOF
```

## Report and artifact

Create:

```text
report/report134.md
E:\Project2026\RecoveryReports\WpfV005ReuseFirstInstallationV001
```

Include:

```text
FORENSIC_SEARCH.md
LAST_WORKING_IMPLEMENTATION.md
CURRENT_VS_LAST_WORKING_DIFF.md
REUSED_OWNERS.md
MINIMAL_SOURCE_CHANGES.md
PAIRING_PLAN_PROOF.md
CHECKPOINT_RESUME_PROOF.md
EXISTING_DB_CLASSIFICATION.md
MIGRATION_RESUME_PROOF.md
BOOTSTRAP_PROOF.md
RUNTIME_PROFILE_PROOF.md
MAINWINDOW_PHYSICAL_PROOF.md
NO_DESTRUCTIVE_ACTIONS.md
BUILD_TEST_TOTALS.md
```

Do not expose passwords, tokens, full connection strings, or private-key values.