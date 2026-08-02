# Prompt139 Runtime Route Unknown Failure Report

## Verdict

`BLOCKED_PHYSICAL_MAINWINDOW_PROOF_NOT_RUN_AFTER_R7_FIX`

The prompt139 source correction is implemented and focused build/tests pass, but the required visible WPF physical proof was not completed in this Codex runner. Do not mark this as the final PASS verdict until an operator confirms MainWindow opens, remains responsive offline for 60 seconds, and restarts twice directly to MainWindow.

## Root-Cause Classification

`R7_DATABASE_CONFIGURATION_OR_RUNTIME_CREDENTIAL_LOAD_FAILURE`

The failing Open OBM-POS path reused the existing startup owner and collapsed a known PostgreSQL runtime credential failure into `UnknownFailure`, which InstallationV0 surfaced as `POS_RUNTIME_ROUTE_UNKNOWNFAILURE`.

Sanitized cause:

- ProductRoot local database settings were present and referenced a protected local database credential file.
- The Visual Studio launch profile also supplied a stale `SPACEPOS_ConnectionStrings__PostgreSqlConnection` override.
- Before prompt139, `SpacePosRuntimeConfiguration` added ProductRoot values before `SPACEPOS_` environment variables, so the stale launch-profile connection string won.
- That stale connection string did not provide a PostgreSQL password, producing a known Npgsql missing-password authentication/setup failure.
- `LocalPosStartupService` then mapped the known credential failure to generic `UnknownFailure`.

## Call Chain And Owner Reused

The existing route owner was reused:

`InstallationV0Window Open OBM-POS command`
-> `InstallationV0Module`
-> `App.OpenInstalledPosFromInstallationV0Async`
-> startup reassessment
-> `LocalPosStartupService`
-> runtime profile / database route assessment
-> InstallationV0 route/result mapping
-> MainWindow launch owner when the local state is ready.

No second startup coordinator, runtime-profile repository, pairing flow, API flow, sync flow, SignalR flow, or manual ApplicationReady bypass was added.

## DB Structural State

Read-only physical probing of the existing target DB found:

- EF migration history object: present.
- Migration-history rows observed: `1`.
- `TblPosRuntimeProfile`: present.
- Current runtime-profile row count: `1`.
- Current runtime-profile semantic state: `Activated`, treated as the existing canonical installed/ApplicationReady equivalent by the startup owner.
- `TblPosRuntimeStateHistory`: present.
- State-history rows observed: `2`.
- Tenant/POS structural rows: present.

Pending migration count was not independently proven by a post-fix physical WPF retest in this runner, so the final physical approval remains blocked.

## Before / After Route Behavior

Before:

- Open OBM-POS reached `InstallationV0OpenObmPos`.
- Credential configuration failed with a known Npgsql missing-password condition.
- The known condition became `LocalPosStartupDecision.UnknownFailure`.
- UI result: `POS_RUNTIME_ROUTE_UNKNOWNFAILURE`.

After:

- ProductRoot split database credential values override stale `SPACEPOS_` launch-profile connection-string values for materialized keys.
- Known Npgsql missing-password failures map to a specific database authentication/configuration decision instead of generic `UnknownFailure`.
- If the local installed state is still valid, the existing startup route can proceed to MainWindow.
- If credential configuration is still invalid, the operator should see a specific safe configuration/authentication blocker, not `POS_RUNTIME_ROUTE_UNKNOWNFAILURE`.

## Destructive-Operation Counts

Prompt139 execution counts:

- `DROP DATABASE`: `0`
- `DROP SCHEMA`: `0`
- `TRUNCATE`: `0`
- `EnsureDeleted`: `0`
- database recreate/copy-over: `0`
- manual runtime-profile insert/update to force readiness: `0`

## Source Files Changed Locally

- `4POS/NailSalonNet8/Modules/Configuration/SpacePosRuntimeConfiguration.cs`
- `4POS/NailSalonNet8/Services/Startup/LocalPosStartupService.cs`
- `4POS/NailSalonNet8/InstallationV0/Application/InstallationV0BuildInfo.cs`
- `4POS/NailSalonNet8.Tests/Startup/LocalPosStartupResultTests.cs`
- `4POS/NailSalonNet8.Tests/InstallationV0/InstallationV0Phase1Tests.cs`

The source tree already contained unrelated uncommitted/untracked work from previous prompts; no source commit was created or pushed by this report step.

## Build And Test Totals

- InstallationV0 build: succeeded, `0` warnings, `0` errors.
- Test-project build: succeeded, `1` existing warning group, `0` errors.
- Focused Startup/InstallationV0 tests: `128` total, `128` passed, `0` failed, `0` skipped.

Focused coverage includes:

- `ApplicationReady` / existing `Activated` routes to MainWindow.
- `DatabaseReady` does not open MainWindow.
- Missing profile returns a recoverable route/result, not unknown.
- Duplicate/inconsistent profile does not open MainWindow.
- Known credential/configuration failure is specific, not generic unknown.
- Open-command enablement remains aligned with the canonical route/readiness owner.

## Physical MainWindow / Restart Result

Not physically proven in this runner.

Required operator retest:

1. Run the latest WPF build with visible label `prompt139`.
2. Load the existing protected installation checkpoint.
3. Verify/resume local DB safely against the existing DB only.
4. Confirm pending migrations are `0`.
5. Confirm exactly one current runtime-profile row.
6. Confirm current state is `ApplicationReady` or canonical equivalent `Activated`.
7. Click Open OBM-POS once.
8. Confirm MainWindow opens and remains responsive for at least 60 seconds.
9. Leave API offline and confirm MainWindow remains usable.
10. Restart WPF twice and confirm both launches open MainWindow directly without InstallationV0 flashing.

## Private Artifact

Local artifact version: `WpfV005RuntimeRoutePrompt139V001`

Manifest SHA-256: `CAD5F635BAF2CB68A9A7E1BFB5CABF3EAA38914F75458349E1F972B240289A28`

The private artifact contains sanitized investigation, verification, and operator retest notes only. It does not include passwords, full connection strings, JWTs, pairing codes, raw identity GUIDs, or private business data.

## Coordination Commit

Prompt coordination commit SHA: `3afc3cf41c08f87cd1eedacf63bd5c788f0a4ce4`
