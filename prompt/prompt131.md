# Prompt131 — Remove legacy SpacePos.Provisioning.Schema dependency from active WPF installation and resume V1 physical install

## Starting evidence

Read completely:

```text
report/report129.md
report/report130.md
docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL_V004.md
```

Operator physical evidence from the latest `prompt130` build:

```text
PostgreSQL administrator credential accepted
CREATE DATABASE permission error no longer occurs
active install now fails with:
BLOCKED_WPF_V1_DATABASE_CREATION: SpacePos.Provisioning.Schema tool not found. Incomplete database may be dropped.
```

The active Development target remains:

```text
obm_pos_dev_v1_pg
```

The operator entered `postgres` in the one visible username/password pair to pass CREATE DATABASE. Do not accept `postgres` as the normal runtime identity. The canonical runtime PostgreSQL role remains the existing least-privilege local role, currently `hung` in this Development lane.

## Exact goal

Complete the existing installation implementation with the smallest correction:

```text
provisioning credential
-> create missing DB only
-> assign/verify runtime owner and grants
-> connect using runtime credential
-> apply the existing attached Npgsql/EF migration chain
-> seed baseline
-> DatabaseReady
-> ApplicationReady
-> MainWindow offline
```

Remove the active runtime dependency on the missing external `SpacePos.Provisioning.Schema` executable/tool when its responsibility is already covered by the existing canonical migration/bootstrap services.

Do not create a replacement external executable or a second schema subsystem.

## Frozen work

Do not modify:

```text
Category Weight
Booking Weight
Price Weight
Pairing/WpfJwt/header behavior
API database
TblTenantPosDevice
sync architecture
CompanionApp/BookingConsole
Firebase/.env cleanup
```

Do not reset, copy, or mutate `obm_pos_dev_v0_pg`.

Do not print or persist plaintext passwords.

## Phase 1 — Prove the exact legacy tool call chain

Search the complete WPF source, project files, scripts, publish/build outputs, git history, and accepted artifacts for:

```text
SpacePos.Provisioning.Schema
Provisioning.Schema
schema tool not found
Incomplete database may be dropped
process/executable launch
ProcessStartInfo
schema bootstrap executable path
```

Return:

```text
LEGACY_TOOL_CALLER_CLASS
LEGACY_TOOL_CALLER_METHOD
EXPECTED_TOOL_PATH
EXPECTED_TOOL_PROJECT_OR_ARTIFACT
LAST_WORKING_ROLE_OF_TOOL
CURRENT_REASON_NOT_FOUND
```

Classify exactly one:

```text
S1_OBSOLETE_WRAPPER_AROUND_EXISTING_EF_MIGRATIONS
S2_PROJECT_EXISTS_BUT_BUILD_COPY_WIRING_MISSING
S3_TOOL_CONTAINS_UNIQUE_SCHEMA_LOGIC_NOT_IN_CURRENT_MIGRATIONS
S4_STALE_UNREACHABLE_CODE_INVOKED_BY_WRONG_BRANCH
S5_OTHER_EXACTLY_PROVEN
```

Prefer reuse of the canonical migration owner already used elsewhere in NailSalonNet8.

## Phase 2 — Separate provisioning and runtime credentials minimally

Do not persist `postgres` as the runtime connection identity.

Use the existing UI and settings model with the smallest extension needed:

```text
Runtime host
Runtime port
Runtime username = hung in this Development proof
Runtime password = protected DPAPI storage
Target DB name = obm_pos_dev_v1_pg
Provisioning/admin username = postgres by safe Development default or existing setting
Provisioning/admin password = operator-entered PasswordBox, memory-only
```

The provisioning credential may only be used for:

```text
connect to maintenance DB
CREATE DATABASE when target is absent
set/verify owner and grants for the runtime role
```

After DB creation:

```text
clear admin password from UI/in-memory owner as soon as practical
connect to target DB with runtime credential
run migration/seed/runtime operations as runtime role
```

Do not save the administrator password into `database-password.dpapi`, `database-settings.json`, logs, reports, or source. Only the runtime password may use the existing protected runtime store.

If the current implementation already has a separate provisioning credential boundary, reuse it instead of adding duplicate controls.

## Phase 3 — Replace only the active legacy schema boundary

### If S1 or S4

Replace the legacy process/tool invocation with the existing in-process canonical migration/bootstrap call, for example the existing DbContext migration owner/service. Do not introduce a raw ad-hoc migration runner if an existing owner exists.

Required flow:

```text
create/connect target DB
-> canonical DbContext.Database.MigrateAsync or existing migration service
-> pending migrations = 0
```

### If S2

First determine whether restoring build/copy wiring is truly smaller and still canonical under V004. Do not retain the external tool merely because it existed historically if it only wraps the same EF migration chain.

### If S3

Do not recreate the whole executable. Move only the proven unique required schema behavior into the existing canonical migration/bootstrap owner, preferably as one attached migration or existing service extension. Stop for operator review if this becomes a broad schema rewrite.

## Phase 4 — Incomplete V1 DB safety

Before the next physical run, determine whether `obm_pos_dev_v1_pg` currently exists after the failed attempt.

Branch safely:

```text
DB absent
-> WPF may create it normally

DB exists and contains only an incomplete fresh-install schema with zero business/user data
-> resume idempotently through the same migration/install service
-> do not automatically drop/recreate

DB exists with business/user data or ambiguous state
-> stop with BLOCKED_WPF_V1_EXISTING_DATA_REQUIRES_OPERATOR_REVIEW
```

Automatic drop is forbidden. The text `Incomplete database may be dropped` must not trigger a drop without explicit operator approval.

No normal startup/install retry may call:

```text
EnsureDeleted
DROP DATABASE
DROP SCHEMA
TRUNCATE
```

## Phase 5 — Resume physical installation

Use visible label:

```text
prompt131
```

Physical steps:

```text
1. Launch the actual WPF InstallationV0 build.
2. Runtime username remains hung.
3. Operator enters runtime password and separate postgres admin password locally.
4. Click Install Local Database Baseline once.
5. WPF creates or safely resumes obm_pos_dev_v1_pg.
6. WPF applies canonical migrations in-process.
7. pending migrations = 0.
8. Baseline seed commits.
9. DatabaseReady is recorded.
10. ApplicationReady is recorded.
11. API remains offline.
12. MainWindow opens directly.
13. Keep MainWindow responsive for 60 seconds.
14. Close and restart twice; both launches open MainWindow directly.
```

Do not redeem Pairing Code during this task.

## Required acceptance evidence

Report exact safe fields:

```text
Legacy tool classification S1-S5
Legacy active invocation count before/after
New external schema tool count = 0
Provisioning role used = postgres marker only, no password
Runtime role used = hung marker only, no password
Target DB existed before retry yes/no
Target DB created or safely resumed
Target DB owner/runtime grants proven
Migration owner/service/method
Applied migration count
Pending migration count
TblPosRuntimeProfile row count/state
TblPosRuntimeStateHistory transition counts
Installation TblLocalOutbox rows count
MainWindow 60-second proof
Restart proof 1
Restart proof 2
InstallationV0 flash/reopen count
Destructive action counts
```

## Tests

Run focused tests for:

```text
legacy tool absence no longer blocks installation
no external process dependency for normal schema migration
provisioning credential is not persisted as runtime credential
runtime role owns/can use target DB
existing incomplete fresh DB resumes without drop
existing data blocks destructive reset
migration from zero reaches pending=0
ApplicationReady opens MainWindow with API offline
```

Build InstallationV0 and the WPF test project.

## PASS gate

PASS requires:

```text
active SpacePos.Provisioning.Schema dependency removed or canonically retired
no replacement external schema tool
postgres used only for provisioning
hung used for runtime/migration/seed
obm_pos_dev_v1_pg created or safely resumed by WPF
pending migrations = 0
DatabaseReady and ApplicationReady physically written
MainWindow opens offline for 60 seconds
restart twice opens MainWindow directly
no DB drop/reset/truncate
```

PASS verdict:

```text
OBM_WPF_V1_LEGACY_SCHEMA_TOOL_REMOVED_MIGRATED_APPLICATIONREADY_MAINWINDOW_OFFLINE_PHYSICALLY_PROVEN
```

Narrow blockers only:

```text
BLOCKED_WPF_LEGACY_SCHEMA_TOOL_ROLE_UNRESOLVED
BLOCKED_WPF_RUNTIME_PROVISIONING_CREDENTIAL_SEPARATION
BLOCKED_WPF_V1_EXISTING_DATA_REQUIRES_OPERATOR_REVIEW
BLOCKED_WPF_CANONICAL_MIGRATION_PATH
BLOCKED_WPF_APPLICATIONREADY_PHYSICAL_PROOF
```

## Artifact and report

Create a new versioned artifact:

```text
E:\Project2026\RecoveryReports\WpfV1LegacySchemaToolRemovalV001
```

Push:

```text
report/report131.md
```
