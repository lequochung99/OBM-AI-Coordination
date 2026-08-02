# Prompt 105 — Close the WPF Development safety guard, execute R1 reset, and apply migrations from zero

## Starting state

Prompt103 proved:

```text
Main Visual Studio WPF Development DB resolved: yes
Main Visual Studio API Development DB resolved: yes
Reset strategy R1 selected
```

Prompt104 then returned:

```text
BLOCKED_MAIN_DEV_WPF_RESET
WPF Development DB safety guard: no
API Development DB safety guard: no
R1 WPF reset executed: no
R1 API reset executed: no
```

No destructive action occurred.

The operator explicitly confirms again:

```text
The canonical local WPF Development database data is disposable.
The resolved WPF Development database may be dropped and recreated.
The goal is complete, correct production-capable code and migrations; preservation of current Development rows is not required.
```

This task handles only the WPF Development database. Do not touch the API database in this task.

## Authoritative scope

Execute:

```text
1. Read the exact WPF DB resolution and failed safety predicate from the prompt103 and prompt104 private artifacts.
2. Close the WPF safety guard using actual Development evidence.
3. Stop only the WPF Development process/worker using that DB.
4. Drop and recreate the resolved canonical local WPF Development DB.
5. Apply the accepted WPF PostgreSQL migration chain from zero.
6. Prove pending migrations = 0 and physical schema is correct.
7. Leave the WPF Development DB ready for the later API reset and E2E.
```

Do not:

```text
touch/reset/migrate the API Development DB
start the POS1-to-API E2E
run the 15-case failure matrix
implement Category Weight or Booking Weight yet
change checkout/payment behavior
mutate production/customer/reference databases
```

## Required private evidence

Read every relevant file from the prompt103 artifact identified by:

```text
dd5985776be1f431c34f3971be1e38a6b890f7649d60434c335905652f441495
```

Read every relevant file from the prompt104 artifact identified by:

```text
f434e6613c55cda706be7397b866527790fb144f6622d94d1d94b57c65b7f73b
```

At minimum read:

```text
DEV_DB_RESOLUTION.md
RESET_STRATEGY.md
PROCESS_STOP_START.md
MIGRATION_PROOF.md
PRIVATE_HANDOFF.md
```

Also inspect the actual current WPF Development connection resolution source and configuration.

Record:

```text
TASK_SCOPE=WPF_DEV_RESET_AND_MIGRATION_ONLY
API_DB_MUTATION=FORBIDDEN
CURRENT_WPF_DEV_DATA_PRESERVATION=NOT_REQUIRED
PRODUCTION_CAPABLE_MIGRATION_REQUIRED=TRUE
```

## Phase 1 — Identify the exact failed safety predicate

Do not return only:

```text
WPF Development DB safety guard: no
```

List every guard predicate separately, for example:

```text
EnvironmentIsDevelopment
ProviderIsNpgsql
HostIsLocalOrApprovedDevelopment
DatabaseNameMatchesPrompt103Resolution
DatabaseNameIsNotProtected
ConnectionSourceIsCanonicalVisualStudioLane
CredentialResolvedThroughApprovedProtectedMechanism
RuntimeProcessUsesSameDatabase
```

For each predicate return locally:

```text
PASS or FAIL
safe evidence
exact source/config path
exact smallest correction for FAIL
```

The first actual failed predicate is the blocker.

Do not invent a new protected-name rule that rejects the already resolved canonical Development DB merely because it contains `dev` or `v0`.

Do not weaken protection for production/customer/reference names.

## Phase 2 — Reuse the prompt103 resolved WPF target

The prompt103 artifact already resolved the main Visual Studio WPF Development DB.

Reuse that exact database name and connection source; do not search for or switch to another database.

Prove safely:

```text
WPF environment = Development
provider = Npgsql
host = local/approved Development PostgreSQL
database name equals prompt103 resolved target
connection came from the canonical WPF Development lane
```

Verify runtime ownership through at least one direct physical method:

```text
pg_stat_activity while the WPF Development process is connected
or
startup/runtime diagnostic showing current_database() from the actual WPF context
```

Do not expose the password or full connection string.

## Phase 3 — Protected-name refusal

Hard reject any target matching production/customer/reference databases, including:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
Royal/customer/production databases
any remote production host
```

The resolved canonical local WPF Development DB is explicitly authorized for reset once all predicates above pass.

If the exact resolved DB name is unexpectedly protected by current code, inspect whether the protection is correct or an overbroad false positive.

A narrow correction to the Development-only reset guard is allowed only when:

```text
Environment is Development
host is local/approved Development PostgreSQL
exact database name equals the prompt103 resolved Development target
production/reference names remain rejected
```

Add focused tests for any guard correction.

## Phase 4 — Stop WPF process safely

Stop only:

```text
canonical WPF Development debug process
its in-process/outbox background worker
any proven WPF Development process holding the selected DB
```

Do not kill unrelated applications.

Prove no active session from the WPF process remains before drop.

Do not stop PostgreSQL globally.

## Phase 5 — Execute R1 on WPF Development DB

Use the existing approved protected PostgreSQL credential mechanism.

Execute:

```text
terminate remaining sessions only for the selected WPF Development DB
DROP DATABASE selected_WPF_Development_DB
CREATE DATABASE same_name WITH ENCODING 'UTF8'
```

Use safe quoting and database-name validation.

Do not use a different name.

Do not create tables manually.

Record safe markers:

```text
WPF_DEV_DB_EXISTED_BEFORE=true
WPF_DEV_DB_DROPPED=true
WPF_DEV_DB_RECREATED=true
WPF_DEV_DB_ENCODING=UTF8
```

## Phase 6 — Apply canonical WPF migrations from zero

Use the accepted prompt097 migration mechanism and actual WPF source currently used by Visual Studio.

Apply the full migration chain from empty DB.

Required proof:

```text
provider = Npgsql
migration chain applied successfully
__EFMigrationsHistory contains the expected WPF chain exactly once
pending migrations = 0
no EnsureCreated
no manual schema SQL outside attached migrations
```

A migration defect discovered here must be fixed in source rather than patched manually in the DB.

After a narrow migration correction:

```text
drop/recreate the WPF Development DB again
apply from zero again
prove pending migrations = 0
```

## Phase 7 — Physical schema proof

Prove at minimum:

```text
TblTurnPolicy exists
TblTurnAmountRule exists
TblLocalOutbox exists
TblLocalOutbox.ExpectedEventCount NOT NULL
TblLocalOutbox.EntityGuid NOT NULL
TblLocalOutbox.TransactionGuid present
TblLocalOutbox.SequenceNumber present
claim fields present
retry fields present
SentAt present
transaction-group checks/indexes present
```

Prove the Price Rule aggregate schema required by prompt099 exists and is writable.

Do not seed business rows in this task except the absolute schema/migration mechanism requirements.

## Phase 8 — Build and focused tests

Run:

```text
WPF build
WPF migration/design-time focused tests
WPF protected-name/safety-guard focused tests
WPF migration-from-zero test against the recreated Development DB or an equivalent exact repeat
```

Report actual totals.

Build success alone is not PASS.

## End state

Leave the canonical WPF Development DB present and empty/clean with:

```text
migrations current
pending migrations = 0
no failure-injection setting
WPF process stopped unless a verification startup is required
```

Do not drop it after proof.

Do not touch the API Development DB.

## Required private artifact

Create a new versioned artifact:

```text
E:\Project2026\RecoveryReports\MainWpfDevResetMigrationV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
FAILED_GUARD_PREDICATE.md
WPF_DEV_DB_RESOLUTION.md
PROTECTED_NAME_PROOF.md
PROCESS_STOP_PROOF.md
RESET_COMMAND_PROOF.md
MIGRATION_FROM_ZERO_PROOF.md
MIGRATION_HISTORY.md
SCHEMA_PROOF.md
FOCUSED_TEST_OUTPUT.txt
FINAL_WPF_DEV_DB_STATE.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Do not expose credentials, full connection strings, passfile contents, tokens, or private identifiers.

## Public report

Create and push only:

```text
report/report105.md
```

Include:

```text
Verdict
Exact failed WPF guard predicate identified yes/no
WPF Development target reused from prompt103 yes/no
Environment Development proof yes/no
Npgsql proof yes/no
Local/approved host proof yes/no
Protected-name refusal proof yes/no
Runtime DB physical proof yes/no
WPF process stopped yes/no
WPF R1 drop/recreate executed yes/no
WPF migration chain applied yes/no
WPF pending migrations count
WPF physical grouped schema proof yes/no
Focused test totals
WPF build totals
API DB mutated yes/no
Production/customer/reference DB mutated yes/no
Private artifact yes/no
Aggregate SHA-256
```

## Repository rules

OBM source remains local/private.

Push only:

```text
report/report105.md
```

Preserve unrelated dirty source changes.

## Verdicts

PASS:

```text
OBM_MAIN_WPF_DEV_RESET_AND_MIGRATION_READY_FOR_API_RESET
```

Narrow blockers only:

```text
BLOCKED_WPF_DEV_GUARD_PREDICATE
BLOCKED_WPF_DEV_RUNTIME_DB_MISMATCH
BLOCKED_WPF_DEV_PROCESS_STOP
BLOCKED_WPF_DEV_DROP_RECREATE
BLOCKED_WPF_DEV_MIGRATION_APPLY
BLOCKED_WPF_DEV_SCHEMA_PROOF
```

Do not return `BLOCKED_MAIN_DEV_WPF_RESET` without the exact failed predicate and direct evidence.
