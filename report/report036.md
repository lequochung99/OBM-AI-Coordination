# Report 036 - PostgreSQL 42601 Syntax Correction for Phase2 v002 Employees

## Verdict

PHASE2_V002_POSTGRES_SYNTAX_FIX_READY_FOR_USER_RETEST

This report covers prompt/prompt036.md. The source has been corrected and verified by build/tests. The final physical WPF seed/provisioning action was not run.

## Physical Failure Classification

The operator-reported PostgreSQL failure was SQLSTATE 42601, PostgreSQL syntax error, during the Phase2 v002 employee seed trial.

The failing area is the permission parent read/update SQL used before employee insert. With the current target state containing existing permission rows and zero employee rows, the first practical failing stage is:

- StageId: PERMISSION_PARENT_SELECT
- Safe statement fingerprint for the old bad SELECT template: d94bdd2edad5fc6d
- Parser position/routine from the original physical failure: not available in prior evidence because prompt035 diagnostics did not yet include those fields.

Prompt036 now records safe diagnostic fields for future PostgreSQL exceptions:

- SqlState
- StageId
- Position
- InternalPosition
- Routine
- StatementFingerprint
- SchemaName
- TableName
- ColumnName
- ConstraintName
- DataTypeName

No SQL text, credentials, connection strings, row payloads, tokens, or secrets are printed.

## Root Cause

The raw PostgreSQL SQL template used doubled quoted identifiers inside a raw C# string, for example:

```sql
FROM dbo.""TblEmployeePermission""
```

That literal is invalid PostgreSQL syntax. The corrected form is:

```sql
FROM dbo."TblEmployeePermission"
```

The same correction was applied to the permission parent UPDATE and SELECT templates.

## Source Corrections

Changed source files:

- E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs
- E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\PostgreSqlPhase2ReferenceSeedExecutor.cs
- E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\reference-driven-v002-employees-r4\README.md
- E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs

Correction summary:

- Build label updated to prompt036.
- Permission parent SELECT/UPDATE SQL now uses valid PostgreSQL identifier quoting.
- Safe PostgreSQL exception diagnostics now include StageId and statement fingerprint.
- Focused tests now prove the bad doubled identifier pattern is absent.
- Versioned artifact folder added under reference-driven-v002-employees-r4.

## Preserved Prompt035 Behavior

The correction preserves:

- 7-row employee permission reconciliation.
- Permission parent seeding before employee seeding.
- Actual parent GUID map readback from TblEmployeePermission.
- 20 employee target rows.
- varchar(20) target length safeguards.
- Single target transaction.
- Permission and employee outbox insertion.
- Runtime profile/history integration.
- v002 marker remains last.
- Marker remains phase2-reference-driven-trial-v002-employees.

## SQL Stage Inventory

All audited stages are fixed-template or fixed-shape parameterized operations. No audited stage builds an empty VALUES list.

| Stage | Behavior | Safety result |
| --- | --- | --- |
| TARGET_PREFLIGHT | Reads current database and safety state | Static SQL, no dynamic identifiers |
| ADVISORY_LOCK | Takes transaction-scoped advisory lock | Static SQL, parameterized key |
| V001_MARKER_VERIFY | Checks existing canonical baseline marker | Static SQL, parameterized marker |
| PERMISSION_PARENT_SELECT | Reads existing permission by PermissionName | Corrected quoted identifiers, one parameter |
| PERMISSION_PARENT_UPDATE | Updates existing permission row | Corrected quoted identifiers, parameterized update |
| PERMISSION_PARENT_INSERT | Inserts missing permission rows | Fixed row dictionary, non-empty columns |
| PERMISSION_PARENT_READBACK | Reads actual permission GUID map | Static SELECT, parameterized permission names |
| PERMISSION_OUTBOX_INSERT | Emits permission outbox rows | Fixed outbox column set |
| EMPLOYEE_INSERT | Inserts 20 employee rows | Fixed reference-driven row set |
| EMPLOYEE_OUTBOX_INSERT | Emits employee outbox rows | Fixed outbox column set |
| RUNTIME_PROFILE_SELECT | Reads runtime profile | Static SQL |
| RUNTIME_PROFILE_UPDATE | Updates runtime profile metadata only | Static SQL, parameterized |
| RUNTIME_HISTORY_INSERT | Inserts runtime transition only if absent | Static INSERT SELECT, parameterized |
| EXCLUDED_TABLE_VERIFY | Verifies no prohibited business table rows | Fixed audited table list |
| V002_MARKER_INSERT | Inserts final v002 marker last | Fixed marker row shape |

## Empty Set Behavior

The executor uses per-row operations for permissions, employees, and outbox rows. If a source collection is empty, the loop does not execute and no invalid empty VALUES command is generated.

## Rollback and No Partial Commit Proof

Read-only target verification was performed using a PostgreSQL READ ONLY transaction and ROLLBACK.

Observed target counts after the fix/build/test cycle:

- dbo.TblEmployeePermission: 3
- dbo.TblEmployee: 0
- dbo.TblLocalOutbox: 21
- phase2-reference-driven-trial-v002-employees marker: 0
- dbo.TblPosRuntimeProfile: 1
- RuntimeState: Activated
- dbo.TblPosRuntimeStateHistory: 1
- Existing baseline marker: 2026.07.15-system-baseline-v1

This proves prompt036 did not partially apply employee v002 seed data to the target database.

## Expected Physical Retest Delta

On the next operator-run physical retest, a successful v002 transaction is expected to add:

- TblEmployeePermission: +4, reaching 7 total permissions.
- TblEmployee: +20.
- TblLocalOutbox: +24, from 21 to 45.
- TblPosRuntimeProfile row count: unchanged at 1.
- TblPosRuntimeStateHistory row count: unchanged if Activated transition already exists.
- TblSystemBaselineVersion: +1 v002 marker, inserted last.

Same-version replay policy remains unchanged: after the v002 marker exists, the same version must not insert duplicate employees, permissions, runtime history, or outbox rows.

## Verification Commands

Commands run:

```powershell
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Results:

- InstallationV0 build: PASS, 0 warnings, 0 errors.
- NailSalonNet8 build: PASS, 0 warnings, 0 errors.
- InstallationV0 focused tests: PASS, 43 passed, 0 failed, 0 skipped.

## Physical Retest Instructions

For the operator retest:

1. Run the current WPF build.
2. Use the existing Phase2 v002 reference-driven employee seed path.
3. Do not manually alter the target database before retest.
4. Confirm no PostgreSQL 42601 occurs.
5. Confirm the expected deltas above.
6. Confirm v002 marker is present only after the transaction completes.

## Safety Confirmation

- No final physical WPF seed/provisioning run was performed.
- No local POS database data was modified by Codex.
- No credentials, passfiles, connection strings, tokens, row payloads, or private GUID maps were printed.
- No source commit or source push was performed.
- Only this coordination report is intended for commit/push.

