# Prompt 107 — Close the exact WPF Development maintenance-connection blocker and complete the approved reset/migration gate

## Starting verdict

Report106 returned:

```text
BLOCKED_WPF_DEV_MAINTENANCE_CONNECTION
```

Coordination references:

```text
report/report106.md
report106 commit: d91d18a3bf775c29d845c608eaafef2afb278db1
prompt106 private artifact aggregate SHA-256:
add6d1b8c00b38dd2eabaaf55f8428186757e2bc6626ccd0567844f99599b203
```

Report106 proves:

```text
Exact prompt105 failed test identified: yes
Physical target DB connection proof: yes
No-running-WPF-process classified stop-ready: yes
Maintenance connection proof: no
WPF Development reset executed: no
WPF Development DB recreated: no
WPF migration chain applied: no
WPF pending migrations count: 1
API DB mutated: no
Sync flow code changed: no
Production/customer/reference DB mutated: no
WPF build: 0 errors, 0 warnings
```

The public report does not include the exact maintenance utility/method, sanitized exception/inner exception, SQLSTATE, checked protected credential source, or target-session count required by prompt106. This task must recover those direct facts from the private artifact before changing code or rerunning the reset.

## Architectural lock

The only canonical sync flow remains:

```text
WPF domain Save + TblLocalOutbox in one transaction
-> existing periodic WPF outbox bot
-> existing standard API sync flow
-> existing API event/delivery transaction path
-> successful API commit
-> existing SignalR notification
```

Do not create or modify:

```text
uploader
API sync endpoint
API ingest pipeline
periodic bot
ACK/delivery transport
SignalR publisher/path
```

Do not touch the API Development database.

Do not start sync E2E.

Do not modify production/customer/reference databases.

## Scope

Close only the canonical local WPF Development database maintenance/reset boundary:

```text
1. Read the complete prompt106 private artifact.
2. Extract the exact maintenance-connection failure with 100% direct evidence.
3. Correct only the smallest production-capable maintenance-boundary defect, if code/config correction is required.
4. Establish the approved maintenance connection to a non-target local PostgreSQL database.
5. Inspect/close sessions only for the exact authorized WPF Development target.
6. Drop and recreate that same target database in UTF8.
7. Apply the accepted WPF migration chain from zero.
8. Prove pending migrations = 0 and grouped schema physically exists.
9. Rerun the exact failed test and the full focused set with 0 skipped.
```

## Required private evidence intake

Read the versioned artifact created by prompt106:

```text
E:\Project2026\RecoveryReports\MainWpfDevResetExecutionV001
```

At minimum inspect:

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

Verify the aggregate SHA-256 equals:

```text
add6d1b8c00b38dd2eabaaf55f8428186757e2bc6626ccd0567844f99599b203
```

Before any implementation, record locally:

```text
FAILED_MAINTENANCE_METHOD=<exact class and method or exact command>
FAILED_MAINTENANCE_STAGE=<connection-string resolution/open/auth/database selection/other>
MAINTENANCE_DATABASE_REQUESTED=<sanitized database name>
TARGET_DATABASE_REQUESTED=<sanitized exact authorized Development target>
PROTECTED_CREDENTIAL_SOURCE_CHECKED=<name only; no secret>
SANITIZED_EXCEPTION_TYPE=<exact type>
SANITIZED_EXCEPTION_MESSAGE=<sanitized exact message>
SANITIZED_INNER_EXCEPTION_CHAIN=<types and sanitized messages>
POSTGRES_SQLSTATE=<exact value or NOT_AVAILABLE>
MAINTENANCE_CONNECTION_ESTABLISHED=false
TARGET_SESSION_COUNT_BEFORE=<count or NOT_REACHED with exact reason>
```

Do not return another blocker that merely says “maintenance connection failed.”

## Failure classification must be corrected if necessary

Classify the direct failure by the actual boundary:

```text
BLOCKED_WPF_DEV_ADMIN_CONNECTION
```

Use only when the approved protected PostgreSQL administrative credential cannot be resolved or authentication fails before a usable maintenance connection can be established.

```text
BLOCKED_WPF_DEV_MAINTENANCE_CONNECTION
```

Use only when credentials are available but opening the approved non-target maintenance database fails.

```text
BLOCKED_WPF_DEV_DROP
```

Use only when the maintenance connection succeeds but session termination or DROP DATABASE fails.

```text
BLOCKED_WPF_DEV_CREATE
```

Use only when DROP succeeds but CREATE DATABASE/UTF8 recreation fails.

Do not preserve an incorrect blocker label merely to match report106.

## Maintenance connection requirements

Use the existing project-standard protected PostgreSQL maintenance/reset mechanism.

The maintenance connection must:

```text
use PostgreSQL/Npgsql
use the same approved local Development host/port boundary
use an approved protected administrative credential source
connect to a non-target maintenance database
never connect to the database being dropped
never expose a password or complete connection string
```

If the existing maintenance connection builder incorrectly keeps the target database name, loses protected credential resolution, changes host/provider, or strips required Npgsql options, fix only that narrow defect and add focused regression coverage.

Do not create a second reset utility, a new credential framework, or a parallel database-provisioning path.

A typical PostgreSQL maintenance database may be `postgres`, but do not guess or hardcode a new value when the project already defines an approved maintenance database. Resolve and prove the canonical project value.

## Safety gates

Before destructive execution, prove all of the following again:

```text
Environment = Development
provider = Npgsql/PostgreSQL
host = loopback or explicitly approved local Development host
exact target equals the prompt105/prompt106 authorized WPF Development DB
protected production/customer/reference names remain rejected
no matching WPF process = WPF_PROCESS_STOP_READY=PASS
maintenance database != target database
API Development DB name != target database
```

Do not:

```text
stop PostgreSQL globally
terminate sessions for unrelated databases
change the authorized target name
use git reset --hard
git clean
revert unrelated local changes
manually create application tables
use EnsureCreated
```

## Session and reset execution

After maintenance connection succeeds:

```text
1. Query sessions only for the exact target database.
2. Record TARGET_SESSION_COUNT_BEFORE.
3. Close only sessions connected to that target, through the approved reset boundary.
4. Prove remaining target session count is zero before DROP, or capture the exact blocker.
5. DROP only the exact authorized WPF Development database.
6. CREATE the same database name with UTF8 encoding.
7. Reconnect and prove database identity and encoding physically.
```

Required markers:

```text
MAINTENANCE_CONNECTION_SUCCEEDED=true
MAINTENANCE_DATABASE_IS_NOT_TARGET=true
TARGET_SESSION_COUNT_BEFORE=<integer>
TARGET_SESSION_COUNT_AFTER_CLOSE=0
WPF_DEV_RESET_EXECUTED=true
WPF_DEV_DB_RECREATED=true
WPF_DEV_DB_ENCODING=UTF8
```

On failure, capture:

```text
exact utility/class/method/command
exact failed stage
sanitized exception and all relevant inner exceptions
PostgreSQL SQLSTATE when available
maintenance connection established yes/no
session count before and after closure
DROP executed yes/no
CREATE executed yes/no
```

## Migration from zero

Use the accepted prompt097 WPF design-time Npgsql factory and migration chain under:

```text
E:\Project2026\4POS\NailSalonNet8
```

Prove:

```text
provider = Npgsql
migration chain applied from zero
migration history contains the expected chain exactly once
pending migrations = 0
no EnsureCreated
no manual application-table creation
```

If migration application exposes a source migration defect, correct only the smallest production-capable defect, reset the Development DB again through the same approved maintenance boundary, and reapply from zero.

Do not patch only the physical database.

## Physical grouped-schema proof

Prove at minimum:

```text
TblTurnPolicy exists
TblTurnAmountRule exists
TblLocalOutbox exists
ExpectedEventCount is required
EntityGuid is required
TransactionGuid exists
SequenceNumber exists
claim/lease fields exist
retry fields exist
SentAt exists
transaction-group checks and indexes exist
```

Use a rolled-back write probe or another non-persistent physical test to prove the required tables are writable.

## Focused tests and build

Rerun:

```text
the exact prompt105 failed test
the exact prompt106 maintenance-boundary regression test, if one exists or is added
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

Build/test PASS does not override a failed physical maintenance/reset/migration proof.

## Required new private artifact

Preserve V001 unchanged. Create:

```text
E:\Project2026\RecoveryReports\MainWpfDevResetExecutionV002
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
V001_ARTIFACT_VERIFICATION.md
EXACT_MAINTENANCE_FAILURE.md
MAINTENANCE_CONNECTION_BEFORE.md
MAINTENANCE_CONNECTION_AFTER.md
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

Do not overwrite or delete V001.

## Public report

Create and push only:

```text
report/report107.md
```

Include:

```text
Verdict
V001 aggregate SHA verified yes/no
Exact failed maintenance method/command
Exact failed maintenance stage
Protected credential source checked (name only)
Sanitized exception type
PostgreSQL SQLSTATE or NOT_AVAILABLE
Physical target DB connection proof yes/no
No-running-WPF-process classified stop-ready yes/no
Maintenance database name sanitized
Maintenance database differs from target yes/no
Maintenance connection proof yes/no
Target session count before reset
Target session count after close
WPF Development reset executed yes/no
WPF Development DB recreated yes/no
UTF8 proof yes/no
WPF migration chain applied from zero yes/no
WPF pending migrations count
Physical grouped schema proof yes/no
Exact failed test result
Focused test totals
WPF build totals
API DB mutated yes/no
Sync flow code changed yes/no
Production/customer/reference DB mutated yes/no
Private artifact yes/no
Aggregate SHA-256
```

Do not expose secrets, complete connection strings, raw tenant/device identifiers, or private business payloads.

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

A blocked result is acceptable only with the exact failed boundary, direct sanitized error evidence, SQLSTATE when available, and the remaining session/reset state.
