# report146 — PROMPT305C Full InstallationV0 Baseline Seed / Outbox Drain (V013)

## Verdict

`V013_FULL_BASELINE_SEED_OUTBOX_DRAIN_PASS`

## Summary

Full V007 `BaselineSeeder.SeedMissingAsync` on V013 staged **107** bootstrap outbox rows (16 EntityTypes). After API ensure-missing Tenant/PosLocal fix and canonical `FlushOutboxAsync` redrain, **all 107 Sent=2**, API EventLog=107, CostFee + employee WH proven. Service/category intentionally not in this seed. V012 untouched. Source local-only; API fix remains in mixed dirty WT (not committed).

## DBs

- WPF: `obm_pos_dev_v013_pg`
- API: `obm_api_dev_v013_pg`
- Employee source: `enailsalon_phasee1_pos1_pg` (read-only)

## Seed / drain

| Metric | Value |
|---|---|
| Seed result | `BASELINE_SEED_OK` RowsChanged=107 |
| Outbox after seed | 107 (106 Sent=0 + 1 CostFee Sent=2) |
| Outbox after drain | 107 Sent=2, Sent=3=0 |
| Token | PosStation (minted) |
| Mid-blocker | WH HTTP 500 → `FK_TblWebEmployeeWorkingHour_Tenant` (Tenant EventLog without domain row) |
| Fix | `EnsureMissingPlatformOwnedIdentityAsync` + replay heal in `EntitiesService` (WT) |

## API counts (selected)

Tenant=1, PosLocal=1, EmployeePermission=7, Employee=20, LoginMethod=3, PaymentMethod=6, Printer=5, ParameterSetting=2, WebEmployeeWorkingHour=43, CostFee=1, ServiceCategory=0, Service=0, EventLog=107

## Feature smoke

| Area | Result |
|---|---|
| Cost/Fee | PASS (WPF/API ≥1, outbox Sent=2, EventLog) |
| ServiceOrderMode column | present both sides; no category rows (seed exclusion) |
| Booking create probe | SKIPPED — no services/categories |
| Assignment CHECK | present (1..6) |
| Manage Schedule allow/apply | code paths present; employee WH synced; tenant WH/absence not seeded |
| today-booked | 200 `[]` with PosStation |

## Builds/tests

- WPF/API build: 0 errors
- API filtered: 13/13 pass
- WPF bootstrap/CostFee filter: 21 pass; 2 fail (stale V012 preferred-DB name asserts)

## Git

- Source branch: `recovery/wpf-installation-reset-cursor-v001` @ `7838685`
- Source: **local-only / no origin**; no new source commit (EntitiesService mixed dirt)
- Coordination: this report + prompt146 + DB_CONTEXT V002

## Local artifact

Path: `E:\Project2026\RecoveryReports\V013\Prompt305C_FullInstallationBaselineOutboxDrain\V001\PROMPT305C_FULL_INSTALLATION_BASELINE_OUTBOX_DRAIN_REPORT.md`  
SHA256: `A4ACEC00403702254DA9999AA1DD8E090D6BAC8744B87F57515B958BD5C7DDAF`

## Next

PROMPT305D — V013 service/category catalog seed/sync + BookingConsole create probe; clean-commit API Tenant ensure-missing; update stale V012 preferred-DB unit tests.
