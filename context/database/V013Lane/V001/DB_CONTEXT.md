# V013 Lane — DB_CONTEXT V001

Status: `VERIFIED`  
Task: `PROMPT305B — Prepare Clean V013 WPF/API Lane`  
Investigator: Cursor  
Evidence timestamp: `2026-08-06T17:30:00-04:00` (approx)

## Lane identity

| Field | Value | Confidence |
|---|---|---|
| Strategy | **A — clean create + migrate + CostFee seed/outbox flush** (not template clone) | FACT |
| WPF DB | `obm_pos_dev_v013_pg` | FACT |
| API DB | `obm_api_dev_v013_pg` | FACT |
| TenantGuid (seed proof) | `921df16c-a48d-4d0b-b4e7-fcd8ddf71a2c` | FACT |
| PosGuid | `ec2b15e9-b7db-4837-8cb9-9d3d7042f90f` | FACT |
| V012 mutated? | **NO** (read-only recheck: CostFee=1, outbox Sent=2 still) | FACT |

## Preferred disposable names (source)

| Surface | Constant / script | Value |
|---|---|---|
| WPF InstallationV0 | `InstallationV0BuildInfo.PreferredDisposableDatabaseName` | `obm_pos_dev_v013_pg` |
| API | `ApiLocalDatabaseNames.PreferredDisposableDatabaseName` | `obm_api_dev_v013_pg` |
| API start | `start-api-local-v013.ps1` / default `start-api-local.ps1 -Database` | `obm_api_dev_v013_pg` |

## Migration state

### WPF `obm_pos_dev_v013_pg`

- Created via `CleanLocalDatabaseService` (postgres provision, owner `hung`)
- AppliedMigrationCount=3, Pending=0
- Migrations:
  - `20260802123558_InitialWpfPostgreSqlV001`
  - `20260805212555_AddServiceOrderModeToWpfCategory`
  - `20260806020138_AddTblWebWorkAbsenceToWpf`
- Critical tables present: `TblLocalOutbox`, `TblSetupCostAndFeeMerchant`, `TblWebWorkAbsence`, `TblWebTenantWorkingHour`, `TblWebEmployeeWorkingHour`, `TblServiceCategory.ServiceOrderMode`

### API `obm_api_dev_v013_pg`

- Empty create (postgres) + OWNER/GRANT to `hung`
- `dotnet ef database update --context ExternalDbContext`
- Migrations applied:
  - `20260802125539_InitialApiPostgreSqlV001`
  - `20260805183849_AddServiceOrderModeToApiCategory`
  - `20260806001809_AddWebAppointmentItemAssignmentState`
- EventLog CHECKs intact (ExpectedEventCount≥1, SequenceNumber≥1, Sequence≤Expected)
- Assignment CHECK: `BookingAssignmentType = ANY (1..6)`
- Assignment columns present on `TblWebAppointmentItem`

## Cost/Fee seed/outbox/API (V013 proof)

| Check | Result |
|---|---|
| WPF CostFee rows | 1 |
| WPF outbox EntityType CostFee | OutboxId=1, Sent=**2**, ErrorMessage NULL |
| API CostFee rows | 1 |
| API EventLog CostFee | EventSequence=1, Operation=I |
| Path | Contract seed + bootstrap-shaped outbox insert + `FlushOutboxAsync` |

## Runtime

| Check | Result |
|---|---|
| API `/health` | 200 |
| API `/health/ready` | 200 |
| API active DB | `obm_api_dev_v013_pg` (start-api-local log) |
| WPF SpacePOS `database-settings.json` | pointed to `obm_pos_dev_v013_pg`; V012 backup under report V001 |
| today-booked (App JWT) | **200** `[]` |

## Counts (sanitized)

| DB | Metric | Count |
|---|---|---|
| WPF V013 | CostFee | 1 |
| WPF V013 | CostFee outbox Sent=2 | 1 |
| API V013 | CostFee | 1 |
| API V013 | EventLog CostFee | 1 |
| WPF V012 | CostFee (untouched) | 1 |

## Blockers / risks

- Source tree still has large unrelated dirty hunks (`MyApiProviderService`, `EntitiesService`, BookingConsole, Manage Schedule). CostFee allowlist fixes live in WT but were **not** cleanly committed in 305A-FIX.
- Full InstallationV0 pairing/seed of all baseline tables not re-run on V013 (CostFee-focused lane proof only).
- `hung` lacks CREATEDB; DB create requires postgres provisioning role.
- Graphify full `update .` previously hung on this machine.

## Invalidation

- Drop/recreate V013 DBs
- Change PreferredDisposable names away from v013
- CostFee outbox/API allowlist regressions
- EventLog CHECK loosened

## Report pointer

`E:\Project2026\RecoveryReports\V013\Prompt305B_PrepareCleanV013Lane\V001\PROMPT305B_PREPARE_CLEAN_V013_LANE_REPORT.md`  
Verdict: `V013_CLEAN_LANE_PREP_PASS`
