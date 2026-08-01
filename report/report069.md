# Report 069 - Persist Service Display Ordering Per Category

## 1. Verdict

`BLOCKED_SERVICE_DISPLAY_ORDERING_BUILD_OR_TEST`

The service-ordering implementation slice is source-complete and the service/MainWindow focused subset passed, but the exact prompt069 test command failed because the broad `FullyQualifiedName~Migration` filter pulled three unrelated migration-file tests whose expected ApiServer SQL files are absent in the current worktree.

## 2. DOCS_READ_BEFORE_CODE_GATE proof

`DOCS_READ_BEFORE_CODE_GATE=PASS`

Read before implementation edits:

- `E:\Project2026\4POS\NailSalonNet8\AGENTS.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `report/report065.md`
- `report/report066.md`
- `report/report067.md`

`report/report068.md` did not exist at execution time.

CanonicalDocVersion: `V002`

CanonicalDocSha256: `916AE7CE71ADA14D768DE1D11E7E5F3604416E8176AA3C59479353481AA2DFB2`

## 3. Existing schema/order-field audit

- Service manual order field: `TblService.NoItem`.
- Service price field: `TblService.ServicePrice`.
- Category display order field: `TblServiceCategory.NoItem`.
- Category sort mode contract exists in source as `ServiceOrderMode`.
- Optional owner-applied DB column patch exists at `E:\Project2026\4POS\NailSalonNet8\docs\sql\patch-service-order-mode-v1.sql`.
- Runtime code probes for the optional `TblServiceCategory.ServiceOrderMode` column and does not execute DDL.
- Outbox payload helper includes `ServiceOrderMode` for Category payloads.

## 4. Final persistent model and migration decision

No new duplicate Service order field was added. Manual order continues to use `TblService.NoItem` scoped by `TenantGuid + ServiceCategoryGuid`.

No automatic physical DB migration was run. The existing owner patch remains the intended schema path for `TblServiceCategory.ServiceOrderMode`; runtime keeps the existing fallback only when that owner column is absent.

## 5. Sort-mode enum/value contract

Source enum values:

- `PriceAscending`
- `PriceDescending`
- `Manual`

UI display labels:

- `Price Low -> High`
- `Price High -> Low`
- `Custom Order`

No active `Customer Order` typo match was found in source/tests after the correction.

## 6. Shared deterministic ordering policy

Shared policy owner: `E:\Project2026\4POS\NailSalonNet8\Services\Catalog\ServiceOrderHelper.cs`

Main POS bridge: `E:\Project2026\4POS\NailSalonNet8\Services\Catalog\PosServiceCatalogOrder.cs`

Policy:

- `PriceAscending`: price ascending, then `NoItem`, then service name, then service GUID.
- `PriceDescending`: price descending, then `NoItem`, then service name, then service GUID.
- `Manual`: `NoItem`, then service name, then service GUID.

The current source model uses non-null `decimal` price fields.

## 7. Management radio-button behavior

`CatalogServicesTab_UC` already guards initialization with `_suppressOrderModeEvents`, persists explicit changes through `SetServiceOrderModeAsync`, refreshes the selected category, and skips no-op mode saves.

Price-mode changes do not rewrite every Service row. Switching into Manual may snapshot the current display order only when the existing manual sequence is not already valid.

## 8. Custom Move Up/Down transaction behavior

`CategoryServiceManagementLocalService.MoveServiceAsync` requires `ServiceOrderMode.Manual`, builds a full-category order, renumbers changed Service rows, and queues intended Service outbox events only for changed rows.

Move buttons are disabled outside Custom/Manual mode by `ServiceOrderHelper.ComputeMoveButtonState`.

## 9. Inactive-row ordering policy

The existing policy reorders against the full Category sequence while using the current visible scope for the operator action. Hidden inactive rows keep stable positions and the final full sequence remains duplicate-free.

## 10. MainWindow load/order path

Corrected real selected-category path:

```text
MainWindow.Lv_Category_SelectionChanged
-> LoadCategoryServicesAsync
-> PosServiceCatalogOrder.LoadOrderedActiveByCategoryAsync
-> ServiceOrderModeSchemaEnsure.ReadModeAsync
-> active Services for selected Category from local PostgreSQL
-> ServiceOrderHelper.OrderServices
-> PosServiceCardItem.FromSortedServices
-> Lv_ServiceComplication.ItemsSource
```

Legacy settings/service screens were also adjusted so their load path uses `PosServiceCatalogOrder` and no longer rewrites service order simply by loading.

## 11. Sync/outbox proof

- Mode save: one intended Category update/outbox event unless no-op.
- Price mode change: does not rewrite every Service row.
- Manual snapshot: Service outbox rows only for changed Service rows.
- Manual move: Service outbox rows only for changed Service rows.
- Refresh/reload: no outbox event.

DTO/source audit shows `TblServiceCategoryDto.ServiceOrderMode` and `TblServiceDto.NoItem` are present in the existing mapping surface.

## 12. Migration-workbook ordering behavior

Workbook/import audit found existing relevant ordering surface:

- Category migration preview preserves `NoItem`.
- Service migration preview/import preserves `NoItem`.
- Replace import models and local service include Category/Service `NoItem`.

No physical workbook import was run.

## 13. Exact files/migrations changed

OBM source/test/doc files changed locally, not committed/pushed:

- `E:\Project2026\4POS\NailSalonNet8\MainWindow.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\MyUserControls\Services_Product_Comlicated_UC.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\MyUserControls\Services_Product_Simple_UC.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\MainWindowServiceOrderIntegrationTests.cs`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V012\CURRENT_TASK_V012.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V012\CURRENT_RESULT_V012.md`

No new migration file was added and no migration was applied.

## 14. Build/test counts

Build:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal
```

Result: `PASS`

- Errors: `0`
- Warnings observed: `176`

Exact prompt069 test command:

```text
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~ServiceOrder|FullyQualifiedName~ServiceSort|FullyQualifiedName~CatalogServices|FullyQualifiedName~MainWindow|FullyQualifiedName~ServiceCategory|FullyQualifiedName~Migration" -v minimal
```

Result: `FAIL`

- Failed: `3`
- Passed: `154`
- Skipped: `0`
- Total: `157`

Failing unrelated tests:

- `PayrollThreeTableFoundationTests.MigrationSql_IsAdditiveAndDoesNotSeedDefaultFiveDollars`
- `PosRuntimeProfileFoundationTests.Runtime_profile_migration_is_idempotent_and_does_not_overwrite_business_tables`
- `PosRuntimeProfileFoundationTests.Migration_declares_singleton_key_not_only_tenant_pos_unique`

Service ordering/MainWindow subset:

```text
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~ServiceOrder|FullyQualifiedName~ServiceSort|FullyQualifiedName~CatalogServices|FullyQualifiedName~MainWindow|FullyQualifiedName~ServiceCategory" -v minimal
```

Result: `PASS`

- Failed: `0`
- Passed: `90`
- Skipped: `0`
- Total: `90`

## 15. Evidence folder/hashes

Evidence folder:

`E:\Project2026\RecoveryReports\ServiceCategoryMigration\ServiceDisplayOrderingV001`

SHA-256 manifest:

```text
F60A15C4E9261F5A13DF56EEB12E82380ABAB89F252DC21D181DE8C1FAF5B12D  mainwindow-order-flow.mmd
C11E35C808D0DFD66D41A745E2D0DA8CA134C66BDD526B5C2F0783F7E77F71AE  management-order-flow.mmd
27ED1B73223C022E8749F6D76542EE5F09D7C703A698271B1763207FBF41E9DD  ordering-field-inventory.csv
491AA88279E9BE28DBE94E5AE08312B8FC44DD089A65D98C8D221969369EE791  outbox-write-matrix.csv
90DAC9E0E1D115FBD61260A97950DF1499300718906FCC9EFB1EB02FE0D13727  README.md
5A17D1A3F90C7A57E5BC51842C8C81B01142AAFBF900B21CD8B3EC833F51E7A9  sort-mode-matrix.csv
6DF0BDA071D7FE10A262761C4C4BA55C2A2F3E01D2F1E2944617E9238F8115D1  test-results.txt
```

## 16. No DB/process/source-push mutation proof

- No WPF launch.
- No PostgreSQL create/update/delete/import/provisioning command.
- No workbook import.
- No API or PlatformApp process stop/start.
- No OBM source commit or push.
- No service names, raw GUIDs, prices tied to named services, credentials, connection strings, or business rows recorded.

## 17. Exact operator physical retest steps

Physical retest should wait until the unrelated migration-file test gate is resolved or explicitly waived.

When approved:

1. Launch WPF manually.
2. Open one Category with multiple Services.
3. Choose `Price Low -> High`; verify Service Manager order.
4. Open MainWindow and select the same Category; verify the same ascending order.
5. Choose `Price High -> Low`; verify both screens.
6. Choose `Custom Order`; move Services up/down; verify both screens.
7. Close/restart WPF; verify persisted mode/order.
8. Switch to another Category and verify independent order.
9. Keep API unavailable/401 and verify local ordering still works.

## 18. Coordination commit SHA

This report is committed and pushed in the coordination repository. The exact commit SHA is returned by Codex after commit/push.
