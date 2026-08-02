# Prompt140 Bootstrap Before ApplicationReady Report

## Verdict

`BLOCKED_PROMPT140_PHYSICAL_BOOTSTRAP_REPAIR_AND_MAINWINDOW_PROOF_NOT_RUN`

Source correction is implemented and focused build/tests pass, but this run did not perform the required visible WPF repair, MainWindow data verification, API-offline 60-second check, or two-restart physical proof. Do not claim the PASS verdict until the operator retest completes.

## Root-Cause Classification

`PREMATURE_APPLICATIONREADY_ACTIVATED_BEFORE_BASELINE_BOOTSTRAP`

The active Phase 2 UI path used a readiness-only executor that upserted Tenant/POS identity and then wrote `Activated` without applying the approved Phase 2 baseline/bootstrap data. Normal startup then treated `Activated` plus Tenant/POS identity as enough to open MainWindow, so OBM-POS could open against an empty baseline.

## Premature-State Call Chain

| Concern | Active owner | Preconditions | Transaction boundary | Observed defect |
|---|---|---|---|---|
| DatabaseReady / installing state | InstallationV0 Phase 2 readiness executor | local DB config and checkpoint identity | separate readiness transaction | wrote intermediate runtime state without baseline proof |
| ApplicationReady equivalent | InstallationV0 Phase 2 readiness executor | Tenant/POS upsert only | second readiness transaction | wrote `Activated` before baseline/bootstrap rows and marker existed |
| Baseline/bootstrap | approved Phase 2 reference bootstrap owner | checkpoint identity plus target DB | not invoked by the active UI path | required settings/roles/printers/parameters stayed at zero |
| Normal runtime route | startup assessment service | runtime profile `Activated`, Tenant/POS matched | read-only startup assessment | allowed MainWindow without completion marker/baseline counts |

## DB Classification

`B2_BOOTSTRAP_PARTIALLY_APPLIED_STATE_PREMATURELY_ACTIVATED`

Sanitized count-only pre-repair evidence from the existing DB:

| Item | Count / state |
|---|---:|
| EF migration history rows | 1 |
| runtime profile | 1:Activated |
| runtime state history rows | 2 |
| Tenant rows | 1 |
| POS rows | 1 |
| Phase2 completion marker table | MISSING |
| setup login methods | 0 |
| payment methods | 0 |
| default role/permission rows | 0 |
| required parameters | 0 |
| printer defaults | 0 |
| setup weird rows | 0 |
| local outbox total | 0 |
| installation outbox rows | 0 |
| employees | 0 |
| services | 0 |
| customers | 0 |
| invoices | 0 |
| output info | 0 |

## Source Correction

Changed locally:

- `4POS/NailSalonNet8/InstallationV0/Application/InstallationV0BuildInfo.cs`
- `4POS/NailSalonNet8/InstallationV0/Database/WpfPostgreSqlMigrations.sql`
- `4POS/NailSalonNet8/InstallationV0/Phase2/PostgreSqlPhase2ReferenceSeedExecutor.cs`
- `4POS/NailSalonNet8/InstallationV0/Presentation/InstallationV0Window.cs`
- `4POS/NailSalonNet8/Services/Startup/RuntimeProfileStartupAssessmentService.cs`
- `4POS/NailSalonNet8.Tests/InstallationV0/InstallationV0Phase1Tests.cs`
- `4POS/NailSalonNet8.Tests/Startup/RuntimeProfileStartupAssessmentServiceTests.cs`

Correction summary:

- InstallationV0 now calls the existing approved Phase 2 reference bootstrap executor instead of the readiness-only executor.
- Phase 2 bootstrap no longer stages `TblLocalOutbox` rows during installation/bootstrap.
- Phase 2 writes `Activated` only after baseline rows and completion marker are inserted/verified in the same transaction.
- Startup assessment requires completion marker plus baseline counts before `Activated` can route to MainWindow.
- The migration script now creates the Phase 2 completion marker table idempotently for already-migrated DBs.
- The obsolete prior-marker hard gate no longer blocks repair of the current incomplete DB.

## Transaction Boundary

Before:

`Tenant/POS upsert -> write Installing/DatabaseReady -> commit -> write Activated -> commit`

The approved bootstrap data was not part of the final readiness transaction.

After:

`Tenant/POS identity -> approved baseline/bootstrap rows -> completion marker -> write Activated/runtime history -> commit`

Installation/bootstrap outbox rows must remain zero.

## Build And Test Totals

- InstallationV0 build: succeeded, `0` warnings, `0` errors.
- Test-project build: succeeded, existing warnings, `0` errors.
- Focused InstallationV0/Startup tests: `129` total, `129` passed, `0` failed, `0` skipped.

## Destructive-Operation Counts

- `DROP DATABASE`: `0`
- `DROP SCHEMA`: `0`
- `TRUNCATE`: `0`
- `EnsureDeleted`: `0`
- DB recreate/copy-over/reset: `0`
- manual runtime-profile force patch: `0`
- manual business-row insert/update/delete: `0`
- pairing/API/sync/SignalR change: `0`

## Physical Evidence

Not completed in this runner.

Required operator retest:

1. Run latest WPF build with visible label `prompt140`.
2. Do not reset/recreate/drop/truncate/copy the DB.
3. Let startup block normal MainWindow while marker/baseline counts are missing.
4. Run Install/Resume Local Database Baseline once.
5. Verify pending migrations are `0`.
6. Verify exactly one current runtime profile row.
7. Verify current state is `Activated/ApplicationReady` only after bootstrap completion.
8. Verify completion marker exists and baseline counts match the approved source owner.
9. Verify installation/bootstrap `TblLocalOutbox` rows remain `0`.
10. Open MainWindow and verify baseline-backed screens are no longer empty where baseline data is required.
11. Keep MainWindow responsive for at least 60 seconds with API offline.
12. Restart WPF twice and confirm both launches open MainWindow directly without InstallationV0 flashing.

## Private Artifact

Local artifact version: `WpfV005BootstrapBeforeApplicationReadyPrompt140V001`

Manifest SHA-256: `156906FE3CA7882B1F9F1B77588163040498449B8F9F81C417E4D120A0043A8B`

## Coordination Commit

Prompt coordination commit SHA: `64de67cff80aa5c9a8e63a3ab3dd608deced1bdb`
