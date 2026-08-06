# V013 Lane — DB_CONTEXT V002

Status: `VERIFIED`  
Task: `PROMPT305C — Full InstallationV0 Baseline Seed / Outbox Drain`  
Investigator: Cursor  
Evidence timestamp: `2026-08-06T18:30:00-04:00` (approx)

## Lane identity

| Field | Value | Confidence |
|---|---|---|
| WPF DB | `obm_pos_dev_v013_pg` | FACT |
| API DB | `obm_api_dev_v013_pg` | FACT |
| TenantGuid | `921df16c-a48d-4d0b-b4e7-fcd8ddf71a2c` | FACT |
| PosGuid | `ec2b15e9-b7db-4837-8cb9-9d3d7042f90f` | FACT |
| Seed mode | Full `BaselineSeeder.SeedMissingAsync` + bootstrap outbox | FACT |
| Drain | `FlushOutboxAsync` + PosStation JWT | FACT |
| V012 mutated? | **NO** | FACT |

## Outbox / API after full drain

| Metric | Value |
|---|---|
| WPF outbox total | 107 |
| WPF Sent=2 | 107 |
| WPF Sent=3 | 0 |
| API EventLog total | 107 |
| API EventLog invalid seq | 0 |
| API TblTenant / TblPosLocal | 1 / 1 |
| API TblEmployee | 20 |
| API TblWebEmployeeWorkingHour | 43 |
| API TblSetupCostAndFeeMerchant | 1 |
| API TblServiceCategory / TblService | 0 / 0 (not in baseline seed) |

## Blocker resolved during drain

- Symptom: 43× WH `OUTBOX_GROUP_HTTP_500`
- Cause: `FK_TblWebEmployeeWorkingHour_Tenant` — EventLog Tenant without domain row (platform skip)
- Fix: API `EnsureMissingPlatformOwnedIdentityAsync` (+ replay heal); WT only, not source-committed

## Runtime

- API health/ready 200 on V013
- SpacePOS DB → `obm_pos_dev_v013_pg`
- today-booked PosStation → 200 `[]`

## Pointers

- Local report: `E:\Project2026\RecoveryReports\V013\Prompt305C_FullInstallationBaselineOutboxDrain\V001\PROMPT305C_FULL_INSTALLATION_BASELINE_OUTBOX_DRAIN_REPORT.md`
- Coordination: `report/report146.md`
- Prior: V001 CostFee-only lane prep (PROMPT305B)
