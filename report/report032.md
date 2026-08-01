# Report 032 — Phase 2 v002 Varchar Length Correction

Generated: 2026-07-31

## 1. Verdict

`PHASE2_V002_VARCHAR_LENGTH_FIX_READY_FOR_USER_RETEST`

The source correction is implemented and verified by build/tests. The physical WPF Phase 2 v002 retry was not executed in this Codex run.

## 2. Physical 22001 Evidence Classification

Observed physical failure:

- SQLSTATE: `22001`
- safe error: value too long for `character varying(20)`
- stage: Phase 2 v002 employee seed
- visible build label before correction: `prompt031`

Classification:

`PHASE2_V002_EMPLOYEE_TEXT_LENGTH_MISMATCH`

No raw employee names, PINs, contacts, payroll values, secrets, connection strings, tokens, or private configuration values were printed or persisted in this report.

## 3. Post-Failure Rollback Proof

Read-only rollback proof was performed through a transaction that was explicitly rolled back.

| Target proof | Count/state |
| --- | ---: |
| `TblEmployee` | 0 |
| `TblLocalOutbox` | 21 |
| v001 marker | 1 |
| v002 marker | 0 |
| `TblPosRuntimeProfile` | 1 |
| runtime profile state | `Activated` |
| `TblPosRuntimeStateHistory` | 1 |

This proves the failed v002 operation did not leave a partial employee seed, v002 marker, runtime duplicate, or committed target mutation.

## 4. Exact Offending Table/Column

Exact offending target:

- table: `TblEmployee`
- column: `LoginNumber`
- target type: `character varying`
- target max length: `20`

Root mismatch:

- prompt031 reset value shape: `UNCONFIGURED-` plus an 8-character stable suffix
- generated length: 21
- target maximum: 20

The failure was caused by a deterministic non-secret reset value exceeding the target schema length. It was not caused by printing or copying raw reference employee values.

## 5. Varchar-Length Matrix

Target varchar/text matrix inspected read-only:

| Table | Column | Type | Max length |
| --- | --- | --- | ---: |
| `Phase2TrialCompletionMarker` | `Version` | text | |
| `Phase2TrialCompletionMarker` | `Status` | text | |
| `TblEmployee` | `LoginNumber` | character varying | 20 |
| `TblEmployee` | `NickName` | character varying | 50 |
| `TblEmployee` | `Email` | character varying | 100 |
| `TblEmployee` | `CellPhone` | character varying | 30 |
| `TblEmployee` | `PhotoPath` | text | |
| `TblEmployee` | `SecurityNumber` | character varying | 100 |
| `TblEmployee` | `PermissionName` | character varying | 50 |
| `TblEmployee` | `LoginStatus2` | character varying | 50 |
| `TblEmployee` | `Background` | character varying | 50 |
| `TblEmployee` | `Foreground` | character varying | 50 |
| `TblEmployee` | `SettingDate` | character varying | 10 |
| `TblEmployee` | `TurnNote` | character varying | 1000 |
| `TblLocalOutbox` | `SourceClientId` | character varying | 100 |
| `TblLocalOutbox` | `EntityType` | character varying | 100 |
| `TblLocalOutbox` | `EntityId` | character varying | 100 |
| `TblLocalOutbox` | `Operation` | character | 1 |
| `TblLocalOutbox` | `Payload` | text | |
| `TblLocalOutbox` | `Processor` | character varying | 50 |
| `TblLocalOutbox` | `ErrorMessage` | text | |
| `TblPosRuntimeProfile` | `TenantCode` | character varying | 32 |
| `TblPosRuntimeProfile` | `DeviceRegistrationId` | character varying | 160 |
| `TblPosRuntimeProfile` | `SourceClientId` | character varying | 80 |
| `TblPosRuntimeProfile` | `DatabaseName` | character varying | 128 |
| `TblPosRuntimeProfile` | `EnvironmentName` | character varying | 40 |
| `TblPosRuntimeProfile` | `ProfileVersion` | character varying | 40 |
| `TblPosRuntimeProfile` | `SchemaVersion` | character varying | 40 |
| `TblPosRuntimeProfile` | `BaselineVersion` | character varying | 80 |
| `TblPosRuntimeProfile` | `AppVersion` | character varying | 40 |
| `TblPosRuntimeProfile` | `RuntimeState` | character varying | 40 |
| `TblPosRuntimeProfile` | `RecoveryReasonCode` | character varying | 80 |
| `TblPosRuntimeStateHistory` | `PreviousState` | character varying | 40 |
| `TblPosRuntimeStateHistory` | `NewState` | character varying | 40 |
| `TblPosRuntimeStateHistory` | `ReasonCode` | character varying | 80 |
| `TblPosRuntimeStateHistory` | `CorrelationId` | character varying | 80 |

## 6. Length-Aware Transform Correction

Implemented correction:

- `TblEmployeeTextLimits` now declares the target text limits used by the v002 transform.
- `LoginNumber` reset now uses a schema-aware stable-shortening helper with a maximum of 20 characters.
- display-safe employee text fields are normalized against target limits before insert.
- private/security/contact fields remain reset or cleared according to the existing prompt030/prompt031 policy.
- schema mismatch now fails closed with `PHASE2_V002_TEXT_LENGTH_SCHEMA_MISMATCH`.
- unusably small limits fail closed with `PHASE2_V002_TEXT_LENGTH_LIMIT_TOO_SMALL`.
- PostgreSQL exceptions are wrapped with safe metadata only: SQLSTATE, schema, table, column, constraint, and datatype.

The v002 marker version remains unchanged:

`phase2-reference-driven-trial-v002-employees`

## 7. Collision-Safe Display Shortening Policy

For non-secret display fields that exceed the target column length, the transform keeps deterministic readable text plus a stable suffix derived from the existing row identity material.

This preserves replay stability and avoids reporting raw reference employee values. The correction does not use random truncation and does not create a new marker family.

## 8. Stable GUID/Idempotency Proof

Static idempotency proof remains intact:

- employee GUID remapping remains deterministic;
- outbox identities remain deterministic;
- runtime profile already `Activated` remains verify-only;
- runtime history remains append guarded;
- v002 marker remains version-stable and marker-last.

Physical same-version replay proof is pending the operator retry.

## 9. Outbox/Runtime-History/Marker Corrections

No outbox schema, runtime-history schema, or marker-version defect was found.

Prompt032 adds safe PostgreSQL exception classification around the shared Phase 2 target transaction so future failures can identify safe table/column metadata without exposing private data.

## 10. Source Files Changed

Prompt032-related source files:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\PostgreSqlPhase2ReferenceSeedExecutor.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\reference-driven-v002-employees-r1\README.md`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

## 11. Build/Test Commands And Counts

Commands:

- `dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj`: PASS, 0 warnings, 0 errors.
- `dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj`: PASS, 0 warnings, 0 errors.
- `dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"`: PASS, 43 passed, 0 failed, 0 skipped.

## 12. Physical Retry Result

Not executed.

Reason:

- prompt032 source correction and verification are complete;
- operator physical WPF retry is the next acceptance step;
- no automatic WPF retry was performed by Codex.

Expected next operator action:

- start the corrected WPF build showing `prompt032`;
- execute the Phase 2 v002 employee action once;
- verify no `22001` length failure;
- verify employee seed, outbox, marker-last, runtime state, and replay behavior.

## 13. Runtime Profile/History Physical Deltas

No retry was run, so runtime profile/history remain at the rollback-proof state:

- runtime profile count: 1
- runtime profile state: `Activated`
- history count: 1
- v002 marker count: 0

## 14. v002 Marker-Last/Replay Proof

Static proof:

- the marker version remains `phase2-reference-driven-trial-v002-employees`;
- source still inserts the marker only after employee, outbox, and runtime checks;
- replay stability is preserved through deterministic identifiers and marker versioning.

Physical marker-last and replay proof are pending operator action.

## 15. Prompt032 Label Proof

Active build label:

`prompt032`

Focused tests assert older active build labels are not current for this correction.

## 16. No Secret / Reference Mutation / Source Push Proof

Confirmed:

- reference database mutation: none;
- manual target cleanup: none;
- local POS PostgreSQL schema/manual migration action: none;
- WPF physical retry: not run;
- no secrets, connection strings, employee names, PINs, contacts, payroll values, tokens, or private config printed;
- OBM source repo was not committed or pushed;
- only this coordination report is committed/pushed.

## 17. Coordination Commit SHA

Final pushed commit SHA is reported by Codex after commit/push. Embedding the final SHA inside this file would change the commit hash.
