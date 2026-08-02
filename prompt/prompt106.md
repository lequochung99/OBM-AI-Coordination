# Prompt 106 — Execute the approved WPF Development R1 reset through the canonical maintenance boundary

## Starting verdict

Prompt105 returned:

```text
BLOCKED_WPF_DEV_DROP_RECREATE
```

The public report proves these checks now pass:

```text
Exact failed WPF guard predicate identified: yes
WPF Development target reused from prompt103: yes
Environment Development proof: yes
Npgsql proof: yes
Local/approved host proof: yes
Protected-name refusal proof: yes
```

It also reports:

```text
Runtime DB physical proof: no
No matching WPF process found
R1 reset not executed
Migration not applied
Focused tests: 6 passed, 1 failed
```

The operator explicitly authorizes the already resolved canonical local WPF Development database to be reset. Its Development data does not need preservation.

## Architectural lock

This task changes no sync architecture.

The only canonical sync flow remains:

```text
WPF domain writes
-> TblLocalOutbox
-> existing periodic WPF outbox bot
-> existing standard API sync flow
-> successful API commit
-> existing SignalR notification
```

Do not create or modify any uploader, API sync endpoint, ingest pipeline, periodic bot, or SignalR path.

Do not touch the API Development database.

## Scope

Execute only:

```text
1. Read the prompt105 private artifact and identify the exact failed test/command.
2. Produce physical proof of the canonical WPF Development target with a short-lived read-only connection.
3. Treat absence of a running WPF process as a successful process-stop condition.
4. Use the project’s approved PostgreSQL maintenance/reset boundary against only the exact authorized Development target.
5. Recreate that WPF Development database cleanly in UTF8.
6. Apply the accepted WPF migration chain from zero.
7. Prove pending migrations = 0 and grouped schema is physically correct.
8. Rerun the failed focused test and the complete focused set.
```

## Required evidence

Read all relevant files in the prompt105 private artifact identified by:

```text
9bc1500fe25a71788a710874f063b69aadef6cabf5b8036cb2a2d650e2d02df0
```

At minimum:

```text
FAILED_GUARD_PREDICATE.md
WPF_DEV_DB_RESOLUTION.md
PROTECTED_NAME_PROOF.md
PROCESS_STOP_PROOF.md
RESET_COMMAND_PROOF.md
FOCUSED_TEST_OUTPUT.txt
PRIVATE_HANDOFF.md
```

Before execution record locally:

```text
FAILED_TEST_NAME=<exact test>
FAILED_BOUNDARY=<exact boundary>
FAILED_ROOT_CAUSE=<sanitized cause>
```

Do not return another generic reset blocker.

## Phase 1 — Physical target proof

Use the exact canonical WPF Development connection source already resolved in prompt105.

Open a short-lived read-only PostgreSQL/Npgsql connection and prove safely:

```text
connection succeeded
database name equals the prompt105 target
host is loopback or approved local Development
provider is PostgreSQL/Npgsql
```

A running WPF UI process is not required.

Do not expose credentials, complete connection strings, passfile contents, or private identifiers.

If authentication cannot be resolved through the existing protected mechanism, stop with:

```text
BLOCKED_WPF_DEV_ADMIN_CONNECTION
```

and name only the protected credential source that was checked.

## Phase 2 — Process and session readiness

The prompt105 result:

```text
no matching WPF process found
```

must be classified as:

```text
WPF_PROCESS_STOP_READY=PASS
```

Inspect sessions only for the exact authorized WPF Development database.

Use the existing approved reset utility or maintenance service to close only sessions owned by that target when needed.

Do not stop PostgreSQL globally and do not affect unrelated databases or processes.

## Phase 3 — Execute the approved R1 reset

Use the existing project-standard protected PostgreSQL maintenance/reset mechanism.

Requirements:

```text
maintenance connection is not connected to the target database
exact target name is allowlisted from prompt105
Environment remains Development
host remains local/approved Development
production/customer/reference database names remain rejected
the authorized WPF Development database is removed and recreated with the same name
encoding is UTF8
```

Do not create a different database name.

Do not manually create application tables.

Required safe markers:

```text
MAINTENANCE_CONNECTION_SUCCEEDED=true
TARGET_SESSION_COUNT_BEFORE=<count>
WPF_DEV_RESET_EXECUTED=true
WPF_DEV_DB_RECREATED=true
WPF_DEV_DB_ENCODING=UTF8
```

On failure, provide:

```text
exact reset utility/method
sanitized PostgreSQL error/SQLSTATE when available
remaining target-session count
whether maintenance connection was established
```

Use a narrow verdict:

```text
BLOCKED_WPF_DEV_MAINTENANCE_CONNECTION
BLOCKED_WPF_DEV_DROP
BLOCKED_WPF_DEV_CREATE
```

## Phase 4 — Apply migrations from zero

Use the accepted prompt097 design-time Npgsql factory and migration chain from:

```text
E:\Project2026\4POS\NailSalonNet8
```

Apply the complete attached WPF migration chain to the recreated Development database.

Required proof:

```text
provider = Npgsql
migration chain applied from zero
migration history contains the expected chain exactly once
pending migrations = 0
no EnsureCreated
no manual application-table creation
```

If migration source fails, correct the smallest production-capable migration defect, reset the Development DB again through the approved maintenance boundary, and reapply from zero.

Do not patch only the physical DB.

## Phase 5 — Physical schema proof

Prove at minimum:

```text
TblTurnPolicy exists
TblTurnAmountRule exists
TblLocalOutbox exists
ExpectedEventCount is required
EntityGuid is required
TransactionGuid and SequenceNumber exist
claim/lease fields exist
retry fields exist
SentAt exists
transaction-group checks and indexes exist
```

Use a transaction-rolled-back write probe or another non-persistent schema test to prove the three tables are writable.

## Phase 6 — Focused test closure

Rerun:

```text
the exact failed prompt105 test
all WPF Development guard tests
all WPF migration/design-time tests
migration-from-zero proof
```

Expected:

```text
all pass
0 skipped
```

Build WPF after any correction.

## End state

Leave the canonical WPF Development database:

```text
present
clean
migration-current
pending migrations = 0
ready for the later API Development reset
```

WPF may remain stopped.

Do not touch the API Development DB.

Do not start sync E2E.

## Required private artifact

Create a new versioned artifact:

```text
E:\Project2026\RecoveryReports\MainWpfDevResetExecutionV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
FAILED_TEST_ROOT_CAUSE.md
PHYSICAL_CONNECTION_PROOF.md
PROCESS_SESSION_PROOF.md
MAINTENANCE_RESET_PROOF.md
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

## Public report

Create and push only:

```text
report/report106.md
```

Include:

```text
Verdict
Exact prompt105 failed test identified yes/no
Physical target DB connection proof yes/no
No-running-WPF-process classified stop-ready yes/no
Maintenance connection proof yes/no
Target session count before reset
WPF Development reset executed yes/no
WPF Development DB recreated yes/no
UTF8 proof yes/no
WPF migration chain applied yes/no
WPF pending migrations count
Physical grouped schema proof yes/no
Focused test totals
WPF build totals
API DB mutated yes/no
Sync flow code changed yes/no
Production/customer/reference DB mutated yes/no
Private artifact yes/no
Aggregate SHA-256
```

## Verdicts

PASS:

```text
OBM_MAIN_WPF_DEV_RESET_MIGRATION_READY_FOR_API_RESET
```

Narrow blockers only:

```text
BLOCKED_WPF_DEV_ADMIN_CONNECTION
BLOCKED_WPF_DEV_MAINTENANCE_CONNECTION
BLOCKED_WPF_DEV_DROP
BLOCKED_WPF_DEV_CREATE
BLOCKED_WPF_DEV_MIGRATION_APPLY
BLOCKED_WPF_DEV_SCHEMA_PROOF
```

Do not return `BLOCKED_WPF_DEV_DROP_RECREATE` without the exact failed boundary and sanitized root cause.
