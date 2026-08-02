# Prompt 089 — Operator-assisted Price Rule Save exception capture and local PostgreSQL proof without Docker

## Current state

Read `report/report088.md` and all local-only Price Rule evidence before editing.

Prompt088 correctly stopped with:

```text
BLOCKED_PRICE_RULE_SAVE_ROOT_CAUSE_UNPROVEN
```

The operator has physically reproduced:

```text
Price / Amount Rule Settings
-> Add Row
-> enter a rule
-> Save
-> Save fails
```

Docker is not running and no disposable PostgreSQL endpoint was reachable non-interactively. This must not cause another speculative patch.

The operator already has a working local PostgreSQL server used by OBM-POS. Use an operator-assisted workflow instead of Docker.

## Authoritative constraints

- OBM-POS remains PostgreSQL/Npgsql-only.
- Do not reintroduce SQL Server code/packages/configuration.
- Do not test checkout/payment.
- Do not create or activate a Turn Policy.
- Do not automatically mutate the operator's current database.
- A manual UI Save performed by the operator is allowed for reproduction.
- Disposable PostgreSQL database creation/drop is allowed only through an operator-run script using interactive credentials.
- Never print, persist, inspect, or commit passwords, connection strings, raw GUIDs, rule names, or payloads.
- No OBM source commit/push.

## Objective

1. Capture the exact real Price Rule Save exception from the operator's current WPF runtime.
2. Prove the exact failing stage and code path.
3. Fix only the proven defect.
4. Prove the final Save/outbox/receiver boundary with a disposable database on the operator's local PostgreSQL server, without Docker.

Do not claim ready until both physical exception proof and real PostgreSQL integration proof are complete.

## Documentation/evidence gate

Read completely:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report086.md
report/report087.md
report/report088.md
```

Read all local Price Rule evidence.

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
EVIDENCE_MODE=OPERATOR_ASSISTED_PHYSICAL_EXCEPTION_CAPTURE
ARCHITECTURE_DECISION=POSTGRESQL_ONLY
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase A — freeze and document the current Save implementation

Before editing, return privately the complete current method bodies, repository-relative paths, and line ranges for:

1. Price Rule Save button handler.
2. DataGrid focused-cell/row commit helper.
3. New/dirty/deleted row detection.
4. Rule-set validation.
5. Tenant resolution.
6. Policy/draft resolution used by Save, if any.
7. Production Price Rule Save service.
8. Insert/update/delete entity mapping.
9. Transaction begin/commit/rollback.
10. Outbox creation.
11. SaveChangesAsync calls.
12. Reload-after-save.
13. Exception/result mapping.

Also return the exact schema/mapping matrix for every editable field plus all non-visible save-side fields such as:

```text
TenantGuid
PolicyGuid or policy/version key
CreatedAt
UpdatedAt
row version/concurrency token
IsDeleted/status
```

Explicitly compare every C# DateTime/DateTimeOffset property with its PostgreSQL column type.

## Phase B — add safe structured physical diagnostics

If the current Save UI exposes only a generic failure, add temporary or permanent sanitized structured diagnostics that preserve:

```text
StageId
ResultCode
TenantResolved
PolicyResolved
NewRows
DirtyRows
DeletedRows
ValidatedRows
InsertedRows
UpdatedRows
DeletedDbRows
OutboxRows
Committed
ReloadSucceeded
ExceptionType
InnerExceptionType
PostgreSqlState
SafeDatabaseObject
SafeConstraintOrColumn
SaveChangesCallNumber
```

Do not display:

```text
rule names
amount values
raw IDs/GUIDs
payloads
connection strings
credentials
employee/customer/business data
```

The final UI may show a concise status line; detailed sanitized diagnostics may be written to a versioned local evidence file.

Do not swallow the inner exception.

## Phase C — operator physical reproduction package

Create a new versioned local evidence folder, for example:

```text
PriceRulePhysicalSaveCaptureV001
```

Include:

```text
README.md
Capture-PriceRuleSave-Diagnostics.ps1
expected-safe-output.md
SHA256SUMS.txt
```

The package must not be pushed to GitHub.

README must instruct the operator:

```text
1. Stop all old WPF processes.
2. Rebuild WPF.
3. Launch the current build manually.
4. Open Price / Amount Rule Settings.
5. Add exactly one rule using a simple valid test range approved by the operator.
6. Keep focus in the last edited cell and click Save once.
7. Copy the safe status line and local sanitized diagnostic file path.
8. Do not retry Save or perform other setup/checkout actions before collecting diagnostics.
```

`Capture-PriceRuleSave-Diagnostics.ps1` may collect only the sanitized local diagnostic file and assembly provenance:

```text
running assembly path
assembly hash/timestamp
build label
safe result fields
```

It must not connect to the DB or read credentials.

## Phase D — classify the proven root cause

After physical reproduction, classify using the narrowest proven code:

```text
GRID_EDIT_NOT_COMMITTED
VALIDATION_OR_PRECISION_FAILURE
MANDATORY_POLICY_FK_MISSING
TENANT_OR_POLICY_SCOPE_MISMATCH
DATE_TIME_MAPPING_MISMATCH
ENTITY_TRACKING_CONFLICT
COLUMN_MAPPING_MISMATCH
SCHEMA_COLUMN_MISSING
NOT_NULL_OR_FK_CONSTRAINT
UNIQUE_OR_EXCLUSION_CONSTRAINT
OUTBOX_PAYLOAD_FAILURE
OUTBOX_INFRASTRUCTURE_UNAVAILABLE
TRANSACTION_BOUNDARY_WRONG
RELOAD_MISREPORTED_AS_SAVE_FAILURE
OTHER_PROVEN_CAUSE
```

Do not patch before the inner exception and failing statement are proven.

## Phase E — implement the smallest proven correction

Preserve initialization model A:

```text
zero rules -> empty editor
operator Add Row/Reset Defaults -> in-memory only
explicit Save -> first DB materialization
```

No Draft/no Active Policy must remain valid unless the actual schema proves a mandatory policy FK. If a mandatory policy FK is proven, do not silently create a policy. Return a precise blocker and propose the minimal domain redesign required.

The final save boundary must be:

```text
commit current DataGrid edit
-> detect final new/dirty/deleted set
-> validate complete rule set
-> resolve tenant and any proven mandatory scope
-> begin one Npgsql transaction
-> apply insert/update/delete
-> create canonical outbox event(s) in same DbContext
-> one SaveChangesAsync when possible
-> commit once
-> reload
```

Outbox unavailable/failure must roll back rule changes.

## Phase F — operator-run disposable PostgreSQL integration harness without Docker

Create a local-only operator-run package, for example:

```text
PriceRuleLocalPostgresE2EV001
```

Include:

```text
README.md
Run-PriceRule-LocalPostgres-E2E.ps1
Drop-PriceRule-LocalPostgres-E2E.ps1
expected-safe-output.md
SHA256SUMS.txt
```

The script must:

1. Use the operator's existing local PostgreSQL server.
2. Require explicit operator input for host/port/user if not already approved.
3. Use `psql -W` or an operator-set `PGPASSFILE`; never print or inspect credentials.
4. Refuse to use the production/current target DB name.
5. Create a uniquely named disposable database.
6. Apply the current schema through the approved existing harness.
7. Run the production Price Rule Save service and receiver/apply integration tests.
8. Drop the disposable database after completion.
9. Produce sanitized PASS/FAIL output only.

Required real PostgreSQL proof:

```text
zero-rule load succeeds
Add Row + Save inserts valid rule set
exact expected outbox event count
receiver apply reproduces same final rule set
receiver replay idempotent
receiver creates no outbox echo
outbox failure rolls back rules
validation failure writes nothing
no-op save writes nothing
wrong tenant rejected
focused cell edit committed
reload failure distinguished from commit failure
Test Lookup equals production selector
```

If the operator-run disposable harness cannot be established safely, return:

```text
BLOCKED_PRICE_RULE_LOCAL_POSTGRES_E2E_NOT_RUN
```

Do not claim ready based only on unit tests.

## Phase G — sync contract

Trace and prove the complete existing path:

```text
Price Rule local change
-> TblLocalOutbox
-> DTO/payload
-> API/relay
-> WPF inbound apply
-> Price Rule table
-> runtime refresh
```

Receiver must support idempotent insert/update/delete and no local outbox echo.

If the DTO/apply contract is absent, implement the smallest additive contract using the existing generic sync framework. Do not create a second channel.

## Tests/build

Run:

```text
WPF build
focused Price Rule save/outbox/apply tests
PostgreSQL-only guard tests
```

Preserve:

```text
UseSqlServer active matches = 0
SQL Server direct package refs = 0
production ProviderName = Npgsql.EntityFrameworkCore.PostgreSQL
```

## Reporting sequence

### First response, before operator physical reproduction

Do not claim the defect fixed. Return:

```text
PRICE_RULE_PHYSICAL_DIAGNOSTIC_CAPTURE_READY
```

with exact local package paths and operator steps.

Do not create a final public report yet unless the coordination workflow requires an interim ultra-minimal blocker report.

### Final response, after operator returns diagnostics and runs the local PostgreSQL E2E package

Create and push only ultra-minimal:

```text
report/report089.md
```

Allowed fields:

```text
verdict
physical root cause proven yes/no
save boundary proven yes/no
outbox contract proven yes/no
receiver apply proven yes/no
local PostgreSQL E2E proven yes/no
build/test counts
current DB automatically mutated no
checkout tested no
one evidence SHA-256
```

## Valid final verdicts

```text
OBM_POS_PRICE_RULE_SAVE_SYNC_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_PRICE_RULE_PHYSICAL_DIAGNOSTICS_NOT_RETURNED
```

```text
BLOCKED_PRICE_RULE_LOCAL_POSTGRES_E2E_NOT_RUN
```

```text
BLOCKED_PRICE_RULE_MANDATORY_POLICY_SCOPE
```

```text
BLOCKED_PRICE_RULE_SYNC_CONTRACT_MISSING
```
