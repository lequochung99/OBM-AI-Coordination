# TblSetupCostAndFeeMerchant — DB_CONTEXT V001

Status: `VERIFIED`  
Task: `TASK-SETUP-COST-FEE-MERCHANT-SEED-V001`  
Investigator: Cursor  
Evidence timestamp (source DB read-only): `2026-08-06T16:15:00-04:00` (approx; SELECT-only)

## Canonical names

| Kind | Value | Confidence |
|---|---|---|
| Entity | `TblSetupCostAndFeeMerchant` | FACT |
| Physical table | `dbo."TblSetupCostAndFeeMerchant"` | FACT |
| DbSet | `TblSetupCostAndFeeMerchants` | FACT |
| DTO | `TblSetupCostAndFeeMerchantDto` | FACT |
| Alternate spelling | `TblSetupCostAndFreeMerchant` | FACT — mapper method names only; no table/entity |

## Source commits / Graphify

| Item | Value |
|---|---|
| WPF source commit investigated | `d7bd017786eb64ec686fd79abc5f7971bb05c3f2` |
| Graphify snapshot | `NailSalonNet8/graphify-out` (AST update 2026-08-06 after seed wiring) |
| Graph freshness label | Built-from commit `d7bd0177` |
| Sanitized query artifacts | `graphify/wpf/current/queries/SetupCostFeeMerchant/` |

## Source DB evidence (READ-ONLY)

| Field | Value |
|---|---|
| Database | `enailsalon_phasee1_pos1_pg` |
| Mutation count from this task | `0` (SELECT-only; row GUID unchanged: `f192c5b8-8a95-419b-a2e2-a8775b4374be`) |
| Free-named table present? | NO |
| Fee-named table present? | YES |
| Row count | `1` |

### Columns / types

| Column | Type | Classification |
|---|---|---|
| SetupCostAndFeeMerchantGuid | uuid PK DEFAULT gen_random_uuid() | AUDIT_GENERATED (new Guid on seed) |
| TenantGuid | uuid NOT NULL | RUNTIME_IDENTITY_REPLACE |
| NameCostAndFee | varchar(100) NOT NULL | SAFE_SEED_CONFIGURATION (`????`) |
| ClientFeeInAmount | numeric(14,4) NOT NULL | SAFE_SEED_CONFIGURATION (`0`) |
| ClientFeeInPercent | numeric(14,4) NOT NULL | SAFE_SEED_CONFIGURATION (`2`) |
| BankFeeInAmount | numeric(14,4) NOT NULL | SAFE_SEED_CONFIGURATION (`0`) |
| BankFeeInPercent | numeric(14,4) NOT NULL | SAFE_SEED_CONFIGURATION (`0`) |
| IsActived | boolean NOT NULL | SAFE_SEED_CONFIGURATION (`false`) |
| CreatedAt | timestamp NOT NULL | AUDIT_GENERATED |
| UpdatedAt | timestamp NULL | AUDIT_GENERATED / NULL on insert |
| IsDeleted | boolean NOT NULL | SAFE_SEED_CONFIGURATION (`false`) |

PK: `SetupCostAndFeeMerchantGuid`  
Unique business key beyond PK: none (idempotency uses TenantGuid + non-deleted presence)  
FK: none observed in PostgreSQL constraints

### Sanitized canonical seed values

```text
NameCostAndFee     = ????
ClientFeeInAmount  = 0
ClientFeeInPercent = 2
BankFeeInAmount    = 0
BankFeeInPercent   = 0
IsActived          = false
IsDeleted          = false
TenantGuid         = <installation identity.TenantGuid>
SetupCostAndFeeMerchantGuid = <new Guid>
```

No secrets/credentials/processor keys in this table. Secret-copy audit: PASS.

## Active WPF target lane (pre/post)

| Field | Value |
|---|---|
| Resolved active DB | `obm_pos_dev_v012_pg` via `C:\ProgramData\SpacePOS\Config\database-settings.json` |
| Pre-state | table present, **0 rows** |
| Post-state | 1 non-deleted row for tenant `921df16c-a48d-4d0b-b4e7-fcd8ddf71a2c` with canonical fee values |
| Idempotent rerun insert | `inserted=0`, still 1 row |
| CostFee outbox | 0 → 1 (`Operation=I`, `Sent=0`, `ExpectedEventCount=1`); rerun staged 0 |

## Seed ownership

| Item | Value |
|---|---|
| Canonical owner | `BaselineSeeder.EnsureSetupCostAndFeeMerchantAsync` |
| Contract | `CostAndFeeMerchantSeedContract` (`StableKey=store-wide-fee-default`) |
| Group | `V007SeedGroups.SetupDefaults` / display `CostAndFeeMerchantDefaults` |
| Idempotency | missing non-deleted TenantGuid row → insert; existing → preserve |
| Legacy | `SeedDbProvider.SeedSetupCostAndFeeMerchantAsync` now uses same contract values |

## Mapper / outbox / sync

| Item | Value |
|---|---|
| Sync contract | YES — `InitialSeedBootstrapOutbox.EntitySetupCostAndFeeMerchant` |
| Staging owner | `InitialSeedBootstrapOutbox.StageAndValidateAsync` |
| EntityType | `TblSetupCostAndFeeMerchant` |
| Expected event | one `I` row per seeded business row, Sent=0 |
| Live StageAndValidate | BLOCKED on this lane by pre-existing 213 unexpected EntityTypes outside bootstrap allowlist (pre-existing hygiene issue) |

## Tests

- `NailSalonNet8.Tests/InstallationV0/CostAndFeeMerchantSeedContractTests.cs`
- Updated: `InitialSeedBootstrapOutboxContractTests` (bootstrap count 16)
- Updated: `V007EmployeeRosterSeedTests` (display rows 16)

Focused filter result: **33 passed / 0 failed**.

## Known risks / unknowns

- `NameCostAndFee = "????"` is the literal source value (also used historically in SeedDbProvider); may be encoding placeholder — do not invent a nicer name without operator approval.
- `IsActived=false` matches source; runtime still loads values when inactive (`MainServices`).
- Mapper Free method names omit `TenantGuid` mapping; bootstrap SQL staging includes TenantGuid.
- Full `StageAndValidateAsync` on `obm_pos_dev_v012_pg` currently blocked by unexpected outbox EntityTypes unrelated to this table.

## Invalidation conditions (must reinvestigate)

- Physical table renamed or Fee/Free spelling changes in migrations
- Fee columns/types change or new secret columns appear
- InstallationV0 seed ownership moves off BaselineSeeder / InitialSeedBootstrapOutbox
- Source DB `enailsalon_phasee1_pos1_pg` fee defaults change and operator wants seed updated
- Sync EntityType naming diverges from physical table name
