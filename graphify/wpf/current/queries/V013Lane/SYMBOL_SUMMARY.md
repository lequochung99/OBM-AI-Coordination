# V013Lane — targeted Graphify / source navigation

Task: PROMPT305B  
Date: 2026-08-06  
Do not load full `graph.json`.

## Queries used

1. `graphify query "V013 database lane PreferredDisposableDatabaseName InstallationV0 migrate"`  
   - Surfaced: `InstallationV0LaneGuard`, `LocalDatabaseInstaller.CreateOrResumeAndMigrateAsync`, `InstallationV0BuildInfo`, LaneGuard tests.

## Source files used (targeted)

| Area | Path |
|---|---|
| WPF preferred DB | `InstallationV0/Application/InstallationV0BuildInfo.cs` |
| WPF lane guard | `InstallationV0/Application/InstallationV0LaneGuard.cs` |
| WPF migrate | `Services/Bootstrap/CleanLocalDatabaseService.cs` |
| WPF CostFee seed | `InstallationV0/Application/CostAndFeeMerchantSeedContract.cs`, `BaselineSeeder.EnsureSetupCostAndFeeMerchantAsync` |
| WPF outbox stage | `InstallationV0/Local/InitialSeedBootstrapOutbox.cs` |
| WPF flush allowlist | `Services/MyApiProviderService .cs` (`ValidateOutboxGroup`) |
| API preferred DB | `Infrastructure/ApiLocalDatabaseNames.cs` |
| API migrate | `DesignTimeExternalDbContextFactory` + `dotnet ef database update` |
| API group apply | `Sync/Service/EntitiesService.cs` (shape/envelope/domain for CostFee) |
| Assignment stamp | `Data/Models/WebBookingAssignmentType.cs` |
| ServiceOrderMode | `BookingConsoleCatalogOrder.cs` (`PriceAscending`/`PriceDescending`/`Manual`) |
| Today-booked | `BookingConsoleAppController.GetTodayBookedCustomersForPos` |

## Prior reusable artifacts

- `graphify/wpf/current/queries/SetupCostFeeMerchant/*`
- `context/database/TblSetupCostAndFeeMerchant/V001/DB_CONTEXT.md`
- PROMPT305A-FIX report (V012 CostFee flush PASS)
