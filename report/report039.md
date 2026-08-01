# REPORT 039 - Phase 2 v002 Restart Hydration and Outbox Accounting

## Verdict

PHASE2_V002_RESTART_HYDRATION_AND_OUTBOX_ACCOUNTING_READY_FOR_USER_RETEST

Prompt: `prompt/prompt039.md`

## Physical Input Evidence

The operator reported that prompt038 Phase 2 v002 physically completed and the WPF UI displayed:

- `PHASE2_V002_IDENTITY_SPINE_POS1_DB_PASS_READY_FOR_WPF_TEST`
- `Rows verified: 47`
- `Outbox delta: 41`
- `Runtime profile rows changed: 1`
- `Runtime history rows inserted: 0`
- `Phase 2 v002 Complete`

On restart with the same ProductRoot, WPF incorrectly displayed the stale state:

- `Phase 2 Local DB Baseline: v001 complete; Employee v002 Runtime-State Upgrade Available`
- the mutating Phase 2 action was enabled again.

## Database Readback

Read-only PostgreSQL inspection was performed against the V0 POS database. No database writes, migrations, WPF installation actions, or Phase 2 reruns were performed.

Canonical post-v002 counts:

| Component | Count / State |
| --- | ---: |
| `dbo.TblTenant` | 2 |
| `dbo.TblPosLocal` | 2 |
| `dbo.TblPosRuntimeProfile` | 1 |
| `dbo.TblPosRuntimeStateHistory` | 1 |
| `dbo.TblEmployeePermission` | 7 |
| `dbo.TblEmployee` | 20 |
| `dbo.TblLocalOutbox` | 62 |
| `dbo.TblSystemBaselineVersion` system baseline v001 marker | 1 |
| `dbo.TblSystemBaselineVersion` Phase 2 v002 marker | 0 |
| runtime state | `Activated` |

## Marker Source Of Truth

The physical Phase 2 marker source of truth is `dbo.Phase2TrialCompletionMarker`, not `dbo.TblSystemBaselineVersion`.

`dbo.Phase2TrialCompletionMarker` contains exactly two completed rows:

- `phase2-reference-driven-trial-v001`
- `phase2-reference-driven-trial-v002-employees`

The v002 marker exists exactly once. Its schema is:

- `MarkerGuid`
- `TenantGuid`
- `PosGuid`
- `Version`
- `Status`
- `CreatedAtUtc`

`dbo.TblSystemBaselineVersion` remains the system baseline marker table and currently contains the system baseline row only. `dbo.TblSchemaVersion` remains schema/migration tracking, not the Phase 2 completion source of truth.

## Root Cause

The executor wrote and verified the durable Phase 2 state through `dbo.Phase2TrialCompletionMarker`, but WPF startup did not read that marker. The startup UI was hard-coded to show v001-complete/v002-available after Phase 1 resume, then enabled the mutating Phase 2 action from Phase 1 success alone.

That caused a false restart state even though the database had already committed Phase 2 v002.

## Source Correction

Implemented startup hydration from the durable database state:

- added `InstallationV0\Phase2\Phase2StartupHydrationService.cs`;
- wired `InstallationV0Window` to call hydration after Phase 1 resume/redeem;
- reads `dbo.Phase2TrialCompletionMarker` in a read-only serializable transaction;
- verifies Tenant/POS identity, runtime profile, runtime state, v002 permission rows, v002 employee rows, and outbox evidence;
- when v002 is complete, displays `Phase 2 Local DB Baseline: Phase 2 v002 Complete`;
- changes the button to `Verify Local Database Baseline` and disables the mutating action;
- shows invariant mismatch instead of enabling a rerun if committed state is inconsistent.

The executor UI text was also clarified:

- `Outbox inserted this run`
- `Outbox total after run`

This prevents the previous ambiguous `Outbox delta` wording from being mistaken for total outbox state.

## Identity Spine

Read-only reconciliation showed the Phase 1 checkpoint/bootstrap identity matches durable database state:

| Invariant | Result |
| --- | --- |
| exact Tenant row count | 1 |
| exact POS row count | 1 |
| runtime TenantGuid match | true |
| runtime TenantCode match | true |
| runtime PosGuid match | true |
| runtime slot match | true |
| runtime InstallationGuid match | true |
| runtime SourceClientId constraint match | true |
| runtime state | `Activated` |

## Outbox Accounting

Total outbox rows after v002: 62.

By entity and operation:

| Entity | Operation | Count |
| --- | --- | ---: |
| `TblEmployee` | `I` | 20 |
| `TblEmployeePermission` | `I` | 7 |
| `TblParameterSetting` | `I` | 2 |
| `TblSetupLoginMethod` | `I` | 6 |
| `TblSetupPaymentMethod` | `I` | 12 |
| `TblSetupPrinter` | `I` | 10 |
| `TblSetupServicesMethod` | `I` | 2 |
| `TblSetupWeird` | `I` | 2 |
| `TblTenant` | `I` | 1 |

By source-client shape:

| Source Shape | Count |
| --- | ---: |
| canonical POS_D lower-case shape | 41 |
| historical POS_N shape | 21 |

Important v002 closure evidence:

- `TblEmployee` canonical POS_D insert outbox rows: 20
- `TblEmployeePermission` canonical POS_D insert outbox rows: 4

## Outbox Delta Explanation

The prompt038 UI value `Outbox delta: 41` was the real transaction delta from the v002 operation, not the total outbox count and not only the missing permission/employee rows.

It consists of:

- 20 employee outbox rows;
- 4 canonical permission outbox rows;
- 17 additional canonical POS_D setup/baseline outbox rows for regular seed rows adopted during the v002 transaction.

No permission outbox closure is required for v002: the required employee and permission outbox evidence exists.

Historical v001 outbox rows remain in the older POS_N source-client shape. They were not rewritten in this task. A future Production V1 reconciliation should decide whether historical v001 rows need a versioned migration to canonical POS_D source-client shape.

## Marker Schema Recommendation

For Production V1, keep a durable marker table but extend it beyond the current trial marker shape. Recommended additional columns:

- `LocalInstallationGuid`
- `InstallationAttemptGuid`
- `ManifestHash`
- `BaselineRowCount`
- `OutboxRowCount`
- optional runtime profile identifier

## Files Changed

Source files changed for prompt039:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2StartupHydrationService.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\PostgreSqlPhase2ReferenceSeedExecutor.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

Coordination report created:

- `report/report039.md`

## Builds And Tests

Executed:

- `dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj`
  - result: PASS
  - warnings: 0
  - errors: 0
- `dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj`
  - result: PASS
  - warnings: 176
  - errors: 0
- `dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"`
  - result: PASS
  - passed: 46
  - failed: 0
  - skipped: 0

## Operator Restart Retest

Expected retest behavior after running the latest WPF build with the same ProductRoot:

- WPF should read durable DB state during startup/resume.
- UI should show `Phase 2 Local DB Baseline: Phase 2 v002 Complete`.
- The mutating Phase 2 install action should not be enabled.
- The proof text should show marker found, identity spine verified, runtime state `Activated`, permission count 7, employee count 20, and outbox accounting evidence.
- No Phase 2 database operation should run automatically.

## Safety

- No WPF run was performed during prompt039.
- No Phase 2 rerun was performed.
- No PostgreSQL mutation was performed.
- No secrets, raw credentials, pairing codes, JWTs, or protected identifiers were printed.
- No source repository commit or push was performed.

