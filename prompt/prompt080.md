# Prompt 080 — Operator-run physical outbox proof and no-Draft Turn Settings closure

## Current state

Prompt079 completed the bounded source correction, but its final verdict remained:

```text
BLOCKED_PHYSICAL_OUTBOX_CORRELATION_AMBIGUOUS
```

Confirmed by report079:

```text
Automatic silent-outbox skip removed: yes
No-Draft Turn Settings load corrected: yes
Zero policy mutation on open/reload: yes
Build: PASS
Focused tests: PASS
Physical outbox exact-match: ambiguous
```

The ambiguity is operational, not a new source defect: the execution agent could not authenticate to the operator's local PostgreSQL database non-interactively without prompting for a credential.

This prompt must not weaken credential controls or use a fallback database role.

## Authoritative credential and safety rules

For the physical local DB verification:

```text
Runtime DB role: hung
Target DB: obm_pos_dev_v0_pg
Read-only only
No fallback to postgres
No password in chat, command line, source, report, logs, environment output, or evidence
```

The operator may execute PowerShell/psql manually and enter the password interactively with `psql -W`.

An already-approved local `PGPASSFILE` may be used only when the operator sets it. The script must not read, print, copy, inspect, hash, or persist the credential file contents.

Force read-only at both session and transaction levels:

```text
PGOPTIONS=-c default_transaction_read_only=on
BEGIN TRANSACTION READ ONLY;
...
ROLLBACK;
```

## Scope

Create a safe operator-run verification package that proves whether one physical Employee Weight Save creates exactly one matching canonical employee update in `TblLocalOutbox`.

Also provide the exact physical retest steps for the corrected no-Draft `Employee Turn Settings` load path.

Do not make another speculative source change unless the existing prompt079 source demonstrably fails a focused static/test audit.

Do not run checkout/payment.

## Documentation/evidence gate

Read before work:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report078.md
report/report079.md
```

Read the complete local-only evidence artifacts from prompt078 and prompt079.

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
EVIDENCE_MODE=OPERATOR_RUN_PHYSICAL_READ_ONLY_PROOF
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Lane A — create the operator-run outbox proof package

Create the next versioned local evidence folder, for example:

```text
EmployeeWeightOutboxPhysicalProofV001
```

Never overwrite an existing version.

Required files:

```text
README.md
Capture-EmployeeWeightOutboxBaseline.ps1
Verify-EmployeeWeightOutboxAfterSave.ps1
employee-weight-outbox-proof.sql
expected-safe-output.md
SHA256SUMS.txt
```

Do not commit or push these local scripts/evidence to GitHub.

### A1. Inspect exact current schema and source contract

Using source/schema metadata only, prove the exact current columns used for:

```text
TblEmployee primary key
TblEmployee TenantGuid
TblEmployee EmployeePermission/Staff eligibility
TblEmployee UpdatedAt
TblEmployee Employee Weight field
TblLocalOutbox primary key
TblLocalOutbox TenantGuid
TblLocalOutbox entity/table name
TblLocalOutbox entity/business key
TblLocalOutbox operation code
TblLocalOutbox payload
TblLocalOutbox occurrence/created timestamp
TblLocalOutbox transaction/idempotency fields
```

Do not guess column names in the script.

The private handoff must include the exact sanitized query and explain every predicate.

### A2. Baseline script

`Capture-EmployeeWeightOutboxBaseline.ps1` must:

1. Verify `psql` exists.
2. Verify the requested DB user is exactly `hung`.
3. Verify the target DB name is exactly `obm_pos_dev_v0_pg`.
4. Refuse `postgres` or any fallback role.
5. Use `psql -W` when no operator-set `PGPASSFILE` is present.
6. Never echo the password or `PGPASSFILE` contents.
7. Set session read-only.
8. Run `BEGIN TRANSACTION READ ONLY` and `ROLLBACK`.
9. Capture only safe baseline metadata needed for the later delta proof, such as:
   - UTC/local proof timestamp according to the actual DB timestamp contract;
   - total employee outbox count;
   - maximum safe occurrence timestamp;
   - no raw employee key, employee name, GUID, payload, or private row value.
10. Write a small local baseline JSON file inside the versioned evidence folder.

The baseline file may contain only counts, timestamps, booleans, schema-contract version, and hashes. It must not contain credentials, employee names, raw IDs, or payloads.

### A3. Operator physical save between the two scripts

README must instruct the operator to:

```text
1. Run Capture-EmployeeWeightOutboxBaseline.ps1.
2. Launch WPF manually.
3. Open Employee Weight Settings.
4. Change exactly one non-sensitive Employee Weight numeric value on one current-tenant Staff employee.
5. Click Save once.
6. Confirm local Save succeeds and the value persists after reload.
7. Do not perform any other employee edit, migration, queue, setup, or checkout action before the verification script.
8. Close or leave WPF idle.
9. Run Verify-EmployeeWeightOutboxAfterSave.ps1 immediately.
```

The operator-triggered UI Save is the only authorized mutation in this flow.

### A4. After-save verification script

`Verify-EmployeeWeightOutboxAfterSave.ps1` must use the captured baseline and run read-only queries only.

It must safely prove:

```text
EmployeeRowsUpdatedAfterBaseline = 1
EmployeeUpdateOutboxRowsAfterBaseline = 1
MatchingEmployeeKeyCount = 1
OperationIsUpdate = True
TenantMatches = True
PayloadContainsEmployeeWeightField = True
DuplicateMatchingOutboxRows = 0
DistinctTransactionCount = 1
```

Use hashes/equality booleans internally where needed. Do not print:

```text
employee name
raw EmployeeGuid
raw TenantGuid
raw outbox GUID
raw payload
Employee Weight value
PIN/login number
connection string
password
```

If other unrelated outbox traffic exists after the baseline, the script must distinguish it from the employee update using the exact entity name, operation, tenant, employee key correlation, timestamps, and payload field contract.

If unique correlation remains impossible, return a precise safe result:

```text
PHYSICAL_OUTBOX_CORRELATION_AMBIGUOUS
```

and identify which aggregate count prevented uniqueness without exposing private identifiers.

### A5. Required final result codes

Success:

```text
EMPLOYEE_WEIGHT_PHYSICAL_OUTBOX_PROOF_PASS
```

Failure variants must be explicit:

```text
EMPLOYEE_WEIGHT_PHYSICAL_OUTBOX_MISSING
EMPLOYEE_WEIGHT_PHYSICAL_OUTBOX_DUPLICATE
EMPLOYEE_WEIGHT_PHYSICAL_OUTBOX_PAYLOAD_FIELD_MISSING
EMPLOYEE_WEIGHT_PHYSICAL_EMPLOYEE_UPDATE_NOT_UNIQUE
PHYSICAL_OUTBOX_CORRELATION_AMBIGUOUS
READ_ONLY_CREDENTIAL_NOT_AVAILABLE
WRONG_DATABASE_OR_ROLE_REJECTED
```

## Lane B — no-Draft Employee Turn Settings physical closure

Do not create or activate a policy.

The operator physical retest must be:

```text
1. Rebuild and launch WPF manually.
2. Open Employee Turn Settings while the DB still has no Draft policy.
3. Confirm no TURN_POLICY_DRAFT_LOAD_FAILED modal appears.
4. Confirm the page displays No Draft / Not Configured.
5. Click Reload.
6. Confirm Reload remains read-only and does not create a policy.
7. Confirm Employee Weight Edit still opens independently and loads Staff employees.
```

Provide a read-only before/after DB proof query for policy counts that returns aggregate counts only. It must prove opening/reloading created zero policy rows.

If the old modal still appears, capture and report:

```text
running assembly path
assembly timestamp/hash
launch profile
exact remaining call site/result code
```

Do not patch again until binary provenance and the active call site are proven.

## Source audit for silent outbox skip

Prompt079 states the automatic silent skip was removed. Verify the current source still satisfies:

```text
outbox infrastructure unavailable
-> explicit save failure
-> transaction rollback
```

No active production path may still do:

```csharp
if (factory == null)
    return 0;
```

for an Employee Weight save.

Add/retain focused tests proving unavailable outbox infrastructure cannot commit the employee update.

## Build/tests

Run the WPF build and focused Employee Weight/Turn Settings/outbox tests.

Do not broaden into unrelated migration suites.

## Mutation boundaries

Allowed:

```text
local script/evidence creation
source/test correction only when directly proven necessary
operator manual Employee Weight Save during physical proof
read-only PostgreSQL verification
```

Forbidden:

```text
automatic DB mutation by Codex
fallback to postgres role
credential capture/storage/output
policy creation/activation
checkout/payment test
OBM source commit/push
```

## Private handoff requirements

Return directly to the operator:

1. Exact script paths.
2. Exact manual commands to run.
3. Whether the command will use interactive `-W` or an operator-set `PGPASSFILE`.
4. Safe expected output.
5. Exact schema predicates used, with private values omitted.
6. No-Draft physical retest steps.
7. Build/test counts.
8. Any source changes made locally.
9. Clear statement that no credential was read or printed.

## Public report

Create and push only an ultra-minimal:

```text
report/report080.md
```

It may contain only:

```text
verdict
operator-run proof package ready yes/no
silent outbox skip absent yes/no
no-Draft source/test closure yes/no
build/test counts
current DB automatically mutated no
checkout tested no
one aggregate evidence SHA-256
```

Do not push scripts, queries, schema details, source paths, database names, credentials, counts from the operator DB, or architecture metadata.

## Valid verdicts

Before the operator runs the package:

```text
OBM_POS_OPERATOR_OUTBOX_PROOF_PACKAGE_READY
```

After the operator returns a successful safe output and it is incorporated into local evidence:

```text
OBM_POS_EMPLOYEE_WEIGHT_OUTBOX_AND_NO_DRAFT_PHYSICAL_PASS
```

Blocked states:

```text
BLOCKED_READ_ONLY_CREDENTIAL_NOT_AVAILABLE
BLOCKED_OPERATOR_OUTBOX_CORRELATION_AMBIGUOUS
BLOCKED_NO_DRAFT_PHYSICAL_RETEST
BLOCKED_BUILD_OR_TEST
```
