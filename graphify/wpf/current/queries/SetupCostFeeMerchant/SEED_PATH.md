# SEED_PATH — TblSetupCostAndFeeMerchant

Task: `TASK-SETUP-COST-FEE-MERCHANT-SEED-V001`  
Source commit: `d7bd017786eb64ec686fd79abc5f7971bb05c3f2`

## Canonical initial seed (InstallationV0)

Owner chain:

```text
InstallationCoordinator
  -> BaselineSeeder.SeedMissingAsync / ExecuteGroupAsync
    -> SeedSetupGroupAsync
      -> EnsureSetupCostAndFeeMerchantAsync
```

Contract owner:

- `InstallationV0/Application/CostAndFeeMerchantSeedContract.cs`
- Stable key: `store-wide-fee-default`
- Catalog registration: `MinimalBaselineCatalog.RequiredStableKeys`
- Progress registration: `V007SeedGroups.OrderedRows` / `DisplayRows` (`CostAndFeeMerchantDefaults`)

Idempotency:

- Missing non-deleted row for `TenantGuid` → INSERT canonical defaults
- Existing non-deleted row for `TenantGuid` → no-op (preserve operator values)
- Verify uses presence/ownership (`ValidatePresence`), not overwrite of operator edits

## Legacy demo seed (not InstallationV0)

- `SeedDb/SeedDbProvider.SeedSetupCostAndFeeMerchantAsync`
- Called from `RunLegacyDemoSeedAllAsync` only
- Updated to reuse `CostAndFeeMerchantSeedContract` values
- Still stages via `_repo.CreateLocalOutboxSingleAsync` + Free-named mapper

## Graphify navigation notes

- `graphify path TblSetupCostAndFeeMerchant BaselineSeeder` returned a weak import-only path (6 hops) before this task; seed ownership was confirmed by exact symbol `SeedSetupCostAndFeeMerchantAsync` (legacy) and source inspection of `SeedSetupGroupAsync` (canonical, previously missing this table).
- After implementation, Ensure method lives in `BaselineSeeder.cs`.
