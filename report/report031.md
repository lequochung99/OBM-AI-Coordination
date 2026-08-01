# Report 031 — Phase 2 v002 Employee Upgrade Runtime-State Integration

Generated: 2026-07-31

## 1. Verdict

`PHASE2_V002_EMPLOYEES_AND_RUNTIME_STATE_READY_FOR_USER_TEST`

The source integration is implemented and verified by build/tests. The physical v002 employee/runtime-state action was not executed in this Codex run.

## 2. Current TblEmployee Implementation Versus Physical State

Prompt030 implementation is present and extended by prompt031:

- `TblEmployee` is selected from the reference database.
- employee private/contact/security fields are reset or excluded.
- employee GUIDs and permission FKs are remapped deterministically.
- employee rows/outbox/marker share the Phase 2 target transaction.

Current target physical state before prompt031 mutation:

| Proof | Value |
| --- | ---: |
| `TblEmployee` count | 0 |
| v001 marker count | 1 |
| v002 marker count | 0 |
| `TblLocalOutbox` count | 21 |

This proves v002 employee seed has not yet physically run.

## 3. Runtime Table/Model/Caller Audit

| Table/model | Purpose | Key/scope | Writer | Reader/UI |
| --- | --- | --- | --- | --- |
| `TblPosRuntimeProfile` / `PosRuntimeProfileRecord` | current runtime profile and current `RuntimeState` | singleton key 1, Tenant/POS/device/installation scoped | `PostgresPosRuntimeProfileRepository.UpsertAsync`, prompt031 executor transaction SQL | `RuntimeProfileStartupAssessmentService`, `DatabaseStartupAssessmentService`, runtime control/rollout |
| `TblPosRuntimeStateHistory` | append-only transition audit | `RuntimeProfileGuid` plus previous/new state/reason | `AppendHistoryAsync`, prompt031 executor transaction SQL | diagnostics/audit |
| `Phase2TrialCompletionMarker` | immutable seed/upgrade completion proof | version + Tenant/POS | Phase 2 executor marker-last insert | InstallationV0 status/hydration |
| `TblPosTerminalConfig` | payment terminal/device configuration | Tenant/POS/provider | `PosTerminalConfigService` | terminal setup/runtime payment config |
| Phase 1 checkpoint | machine-side API authorization proof | ProductRoot checkpoint + protected credential ref | `Phase1InstallationService` | InstallationV0 resume |

## 4. Current Row Counts And Sanitized State Proof

Read-only target proof:

- database: `obm_pos_dev_v0_pg`
- user: `hung`
- `TblPosRuntimeProfile`: 1
- current `RuntimeState`: `Activated`
- `TblPosRuntimeStateHistory`: 1
- `TblPosTerminalConfig`: 0
- `Phase2TrialCompletionMarker`: 1

No credentials, connection strings, terminal secrets, employee names, or private configuration values are reported.

## 5. Authoritative Source-Of-Truth Decision

Decision:

- `TblPosRuntimeProfile` is the current runtime state source of truth.
- `TblPosRuntimeStateHistory` is append-only transition history.
- `Phase2TrialCompletionMarker` is immutable seed-version completion proof.
- `TblPosTerminalConfig` is terminal/device configuration only.
- Phase 1 checkpoint is machine-side API authorization proof.

`InstalledHealthy` is a startup assessment result, not a `PosRuntimeState` enum value.

## 6. Canonical Post-v002 Runtime State

Selected canonical post-v002 runtime state:

`Activated`

Reason: source enum contains `Installing`, `Activated`, `RecoveryRequired`, and `Disabled`; startup code treats `RuntimeState=Activated` as the condition that can yield `InstalledHealthy`.

## 7. TblPosTerminalConfig Classification

`TblPosTerminalConfig` is not used as the app installation state.

It participates in payment terminal/provider configuration and is deliberately absent from the prompt031 Phase 2 executor status decision.

## 8. Transaction Integration Details

Prompt031 extends the real PostgreSQL executor so one target transaction now performs:

1. target guard and V008 rollback-anchor validation;
2. advisory transaction lock;
3. v001 marker verification;
4. reference employee read through read-only reference transaction;
5. insert/adopt selected rows and deterministic outbox;
6. runtime profile verification/update to `Activated`;
7. runtime history append only on actual transition;
8. v002 marker insert last;
9. readback/invariant checks before commit.

Any exception rolls back employees, employee outbox, runtime profile update, runtime history append, and v002 marker.

## 9. Runtime Profile Update And History Append Behavior

Implemented behavior:

- profile missing: `PHASE2_RUNTIME_PROFILE_MISSING`
- identity mismatch: `PHASE2_RUNTIME_PROFILE_IDENTITY_MISMATCH`
- profile state `Disabled`/`RecoveryRequired`: `PHASE2_RUNTIME_PROFILE_STATE_BLOCKED`
- profile state `Installing`: transition to `Activated`, append one history row
- profile state already `Activated`: verify only, no duplicate history append
- unsupported transition: `PHASE2_RUNTIME_PROFILE_TRANSITION_DENIED`

Current physical profile is already `Activated`, so a physical prompt031 run should verify without appending duplicate history solely for replay.

## 10. Marker-Last/Readback Proof

Static proof:

- `EnsureActivatedRuntimeProfileAsync` is called before selecting/inserting the v002 marker.
- v002 marker version is `phase2-reference-driven-trial-v002-employees`.
- tests assert profile/history integration appears before marker-last logic.

Physical marker-last proof is pending operator action.

## 11. Mismatch/Fail-Closed Result Codes

Implemented safe result codes:

- `PHASE2_V002_PRIOR_V001_MARKER_MISSING`
- `PHASE2_RUNTIME_PROFILE_MISSING`
- `PHASE2_RUNTIME_PROFILE_IDENTITY_MISMATCH`
- `PHASE2_RUNTIME_PROFILE_STATE_BLOCKED`
- `PHASE2_RUNTIME_PROFILE_TRANSITION_DENIED`
- `PHASE2_RUNTIME_PROFILE_NOT_ACTIVATED`
- `PHASE2_EXCLUDED_RUNTIME_DELTA`

## 12. Physical v002 Employee Seed Counts/Outbox Deltas

Not executed.

Expected reference employee count remains 20, with physical target employee count still 0 before the operator action.

## 13. Runtime Profile/History/Marker Physical Deltas

Not executed.

Current pre-action state:

- runtime profile count: 1
- runtime profile state: `Activated`
- history count: 1
- v002 marker count: 0

## 14. Restart/UI Hydration Proof

Static UI proof:

- prompt031 label is active.
- WPF status copy now shows `Employee v002 Runtime-State Upgrade Available`.
- after successful action result, WPF displays runtime profile/history counts and `Phase 2 v002 Complete`.

Physical restart proof is pending operator action.

## 15. Same-Version Zero-Delta Proof

Static idempotency proof:

- deterministic employee/outbox identities are retained;
- runtime history uses `WHERE NOT EXISTS`;
- already `Activated` runtime profile is verify-only;
- marker remains version-stable.

Physical replay proof is pending operator action.

## 16. Management/Checkout Filtering Proof

Static proof remains from prompt030:

- management screens use existing `EmployeeType`/permission logic;
- checkout staff selection remains Staff-filtered;
- prompt031 does not alter staff/non-staff UI filtering.

Physical WPF proof is pending v002 seed and operator test.

## 17. Source Files Changed

Prompt031-related source files:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\PostgreSqlPhase2ReferenceSeedExecutor.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

## 18. Build/Test Commands And Counts

Commands:

- `dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj`: PASS, 0 warnings, 0 errors.
- `dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj`: PASS, 0 warnings, 0 errors.
- `dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"`: PASS, 42 passed, 0 failed, 0 skipped.

## 19. Prompt031 Label Proof

Active label:

`prompt031`

Active window title pattern:

`OBM InstallationV0 Phase 1/2 - prompt031`

Focused tests assert prompt030/prompt028/prompt029 are absent from the active build label.

## 20. No Secret / Reference Mutation / Source Push Proof

Confirmed:

- reference database mutation: none;
- target v002 seed mutation: none;
- no secrets, connection strings, terminal credentials, employee names, PINs, contacts, payroll values, or private config printed;
- OBM source repo was not committed or pushed;
- only this coordination report is committed/pushed.

## 21. Coordination Commit SHA

Final pushed commit SHA is reported by Codex after commit/push. Embedding the final SHA inside this file would change the commit hash.
