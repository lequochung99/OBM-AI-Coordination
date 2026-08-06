# report145 — PROMPT305B Prepare Clean V013 WPF/API Lane

## Verdict

`V013_CLEAN_LANE_PREP_PASS`

## Task and source state

| Field | Value |
|---|---|
| Prompt label | `PROMPT305B — Prepare Clean V013 WPF/API Lane` |
| Coordination prompt file | `prompt/prompt145.md` |
| Executor | Cursor |
| Source branch | `recovery/wpf-installation-reset-cursor-v001` |
| Source remote `origin` | **missing** (local-only; not pushed) |
| Local source commits | `2502a17`, `7838685` |
| Coordination prior commit | `6c7821d` (DB_CONTEXT/task pointer; lacked numbered report) |
| WPF DB | `obm_pos_dev_v013_pg` |
| API DB | `obm_api_dev_v013_pg` |
| Strategy | A — clean migration + CostFee seed/outbox flush (not V012 template clone) |
| V012 mutated? | **NO** |

## Local artifact (authoritative filesystem copy)

| Field | Value |
|---|---|
| Local path | `E:\Project2026\RecoveryReports\V013\Prompt305B_PrepareCleanV013Lane\V001\PROMPT305B_PREPARE_CLEAN_V013_LANE_REPORT.md` |
| SHA256 | `88E644BD6BDDF228ADAFB39BC15C563F8B92A68A9DB3CFE902B36C5591B16B5C` |
| Size bytes | `5808` |
| Evidence filenames (not committed) | `wpf-migrate.log`, `api-migrate.log`, `seed-flush.log`, `api-start.log`, `database-settings.json.v012-backup`, MigrateHarness/, SeedFlushHarness/ |

## Reused context

- `context/database/V013Lane/CURRENT.md` + `V001/DB_CONTEXT.md` (already on `6c7821d`)
- `context/database/TblSetupCostAndFeeMerchant/V001/DB_CONTEXT.md`
- `graphify/wpf/current/queries/V013Lane/SYMBOL_SUMMARY.md`
- `graphify/wpf/current/queries/SetupCostFeeMerchant/SYNC_PATH.md`
- PROMPT305A-FIX CostFee allowlist physical pass on V012

## Summary

Paired clean V013 DBs were created and migrated. CostFee was seeded on WPF V013, staged to outbox, and flushed via `FlushOutboxAsync` to API V013 (`Sent=2`, API row + EventLog). API `/health` and `/health/ready` returned 200 against `obm_api_dev_v013_pg`. SpacePOS config pointed to V013 with V012 backup. Focused tests 24/24. Unrelated dirty WPF/API hunks were not staged. Full InstallationV0 baseline seed of all tables was not re-run (CostFee-focused lane proof).

## V013 DBs

| DB | Action |
|---|---|
| `obm_pos_dev_v013_pg` | Created empty + WPF EF/Npgsql migrations via `CleanLocalDatabaseService` |
| `obm_api_dev_v013_pg` | Created empty + `dotnet ef database update` (3 migrations) |

## WPF schema proof

| Check | OK |
|---|---|
| `TblLocalOutbox` | yes |
| `TblSetupCostAndFeeMerchant` | yes |
| `TblServiceCategory.ServiceOrderMode` | yes |
| `TblWebWorkAbsence` / TenantWH / EmployeeWH | yes |
| AppliedMigrationCount | 3 |
| PendingMigrationCount | 0 |

WPF migrations:

1. `20260802123558_InitialWpfPostgreSqlV001`
2. `20260805212555_AddServiceOrderModeToWpfCategory`
3. `20260806020138_AddTblWebWorkAbsenceToWpf`

Preferred disposable: `InstallationV0BuildInfo.PreferredDisposableDatabaseName = obm_pos_dev_v013_pg`.

## API schema proof

API migrations:

1. `20260802125539_InitialApiPostgreSqlV001`
2. `20260805183849_AddServiceOrderModeToApiCategory`
3. `20260806001809_AddWebAppointmentItemAssignmentState`

| Check | OK |
|---|---|
| `TblSetupCostAndFeeMerchant` | yes |
| `TblEventLog` group CHECKs | yes |
| `ServiceOrderMode` | yes |
| Assignment columns + CHECK 1..6 | yes |
| Schedule tables | yes |

## Cost/Fee proof

| Check | Result |
|---|---|
| WPF seed count | 1 (`500b12bb-fe51-4568-9c55-854ce47b351d`) |
| WPF outbox | OutboxId=1, Sent=**2**, ErrorMessage NULL |
| Flush path | `MyApiProviderService.FlushOutboxAsync` |
| API CostFee | 1 |
| API EventLog | EventSequence=1, Operation=I |
| Manual Sent / API insert | not used |

## Critical feature coverage

| Area | Result |
|---|---|
| ServiceOrderMode | `PriceAscending` / `PriceDescending` / `Manual` |
| Booking EventLog | CHECKs intact; CostFee EventLog via sync-transaction-group |
| Assignment | AnyTech→1 null; specific emp→2 |
| Manage Schedule | tables + allow/apply present; no disruptive absences created |
| Today-booked | App JWT → HTTP 200 `[]` |

## Runtime

| Check | Result |
|---|---|
| API start | `start-api-local-v013.ps1` |
| `/health` / `/health/ready` | 200 / 200 |
| Active API DB | `obm_api_dev_v013_pg` |
| WPF config | SpacePOS → `obm_pos_dev_v013_pg` (V012 backup retained) |

## Validation

| Check | Result |
|---|---|
| WPF build | PASS |
| API build | PASS |
| Focused tests `CostAndFeeMerchant\|InstallationV0LaneGuard` | 24/24 PASS |

## Risks / next

- Unrelated dirty source hunks still unsafe to bulk-commit; CostFee allowlist/API apply may still sit in mixed WT files.
- Full InstallationV0 baseline seed/drain on V013 not done.
- Next: `PROMPT305C — V013 Full InstallationV0 Pairing/Baseline Seed + Outbox Drain`

## Manual operator verification

```text
MANUAL_UI_CHECK_REQUIRED
- Operator owns visual UI acceptance on V013.
- Confirm SpacePOS database-settings.json points to obm_pos_dev_v013_pg when launching WPF.
- Confirm API started via start-api-local-v013.ps1.
```
