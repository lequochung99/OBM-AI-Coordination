# SYMBOL_SUMMARY — TblSetupCostAndFeeMerchant

Task: `TASK-SETUP-COST-FEE-MERCHANT-SEED-V001`  
Product root: `E:\Project2026\4POS\NailSalonNet8`  
Source commit investigated: `d7bd017786eb64ec686fd79abc5f7971bb05c3f2`  
Graphify snapshot: `graphify-out` rebuilt 2026-08-06 (AST update after this task); freshness label still cites `d7bd0177`  
Queries used: `graphify explain`, `graphify query`, `graphify path` (no full `graph.json` load)

## Spelling resolution (FACT)

| Name | Graphify node? | Role |
|---|---|---|
| `TblSetupCostAndFeeMerchant` | YES | Canonical EF entity + physical table + DbSet |
| `TblSetupCostAndFreeMerchant` | NO entity/table node | Historical mapper method naming only (`MapperHelpper` region / method names) |
| `TblSetupCostAndFeeMerchantDto` | YES | Sync/DTO type (`Dtos/SharedModels/TblSetupCostAndFeeMerchantDto.cs`) |

Canonical spelling: **Fee** (`TblSetupCostAndFeeMerchant`).  
`Free` is a naming mismatch in mapper method identifiers only; both mapper methods operate on Fee entity/DTO.

## Core symbols

- Entity: `MyData/TblSetupCostAndFeeMerchant.cs`
- DbSet: `eNailSalonDbContext.TblSetupCostAndFeeMerchants` → table `TblSetupCostAndFeeMerchant`
- DTO: `TblSetupCostAndFeeMerchantDto`
- Mapper methods (legacy Free names):
  - `TblSetupCostAndFreeMerchantDto_to_TblSetupCostAndFreeMerchant`
  - `TblSetupCostAndFreeMerchant_to_TblSetupCostAndFreeMerchantDto`
- Runtime load: `MainServices.ReloadCustomerTransactionFeePolicyCoreAsync`
- UI (out of scope for this task): `Setting_MerchantTerminal`, `CustomerTransactionFeePolicy_UC`

## Ambiguity

None after Graphify + source + PostgreSQL agreement on **Fee**.
