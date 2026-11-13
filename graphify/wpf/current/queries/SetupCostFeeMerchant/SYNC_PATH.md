# SYNC_PATH — TblSetupCostAndFeeMerchant

Task: `TASK-SETUP-COST-FEE-MERCHANT-SEED-V001`  
Source commit: `d7bd017786eb64ec686fd79abc5f7971bb05c3f2`

## Verdict

`EXISTING_SYNC_CONTRACT` — reuse InstallationV0 initial-seed bootstrap outbox; do not invent a new uploader.

## Canonical bootstrap outbox

Owner:

```text
BaselineSeeder (after seed groups)
  -> InitialSeedBootstrapOutbox.StageAndValidateAsync
    -> StageByQueryAsync(EntitySetupCostAndFeeMerchant)
      -> InsertOutboxRowAsync (Operation=I, Sent=0, ExpectedEventCount=1)
```

Wiring added this task:

- `EntitySetupCostAndFeeMerchant = "TblSetupCostAndFeeMerchant"`
- Included in `BootstrapEntityTypes` (count 15 → 16)
- Seed SELECT payload includes PK + TenantGuid + fee fields + audit flags
- Count SQL registered in `ReadCountsAsync`

## Mapper / DTO

- EntityType string for outbox = physical table name `TblSetupCostAndFeeMerchant`
- DTO type: `TblSetupCostAndFeeMerchantDto`
- Mapper method names still say `Free` but map Fee types
- Note: mapper currently omits `TenantGuid` copy; bootstrap staging uses SQL SELECT (includes TenantGuid), so initial-seed outbox payload is complete

## Runtime sync consumers (navigation only)

- `MyApiProviderService.FlushOutboxAsync` / group request builders
- UI save paths also call Free-named mapper → `CreateLocalOutboxSingleAsync`

## Live-lane note (obm_pos_dev_v012_pg)

`StageAndValidateAsync` hard-blocks when unexpected EntityTypes exist outside the bootstrap allowlist. This lane had 213 unexpected outbox rows pre-existing, so full StageAndValidate could not run. Focused CostFee-only outbox insert matching InsertOutboxRowAsync shape proved one `I` / Sent=0 / ExpectedEventCount=1 row; rerun staged 0.
