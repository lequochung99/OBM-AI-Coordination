# Report 066 - Service DB/UI Visibility Verification

## 1. Verdict

`OBM_POS_SERVICES_PARTIAL_IMPORT_REVIEW_REQUIRED`

Primary physical classification: `SERVICES_PRESENT_BUT_FILTERED_BY_STATUS`.

Reason: the target database contains the expected 158 Service rows for the active tenant and all rows have resolvable categories, but all 158 rows are inactive. The default Service UI path excludes inactive rows. A safe workbook-vs-target hash comparison also found 2 business-key mismatches, so final data acceptance needs operator review before declaring complete.

## 2. DOCS_READ_BEFORE_CODE_GATE proof

`DOCS_READ_BEFORE_CODE_GATE=PASS`

Read before source/docs edits:

- `E:\Project2026\4POS\NailSalonNet8\AGENTS.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `report/report063.md`
- `report/report064.md`
- `report/report065.md`

CanonicalDocVersion: `V002`

CanonicalDocSha256: `916AE7CE71ADA14D768DE1D11E7E5F3604416E8176AA3C59479353481AA2DFB2`

## 3. Read-only physical DB classification

Target database: `obm_pos_dev_v0_pg`

Inspection method:

- Used the configured protected local credential path.
- Executed only inside `BEGIN TRANSACTION READ ONLY`.
- Ended with `ROLLBACK`.
- Did not print password, connection string, service names, raw GUIDs, or business rows.

Safe physical counts:

- Active tenant present: `true`
- Runtime profile count: `1`
- Service total for active tenant: `158`
- Service active true: `0`
- Service active false: `158`
- Status columns present: `false`
- Deleted columns present: `false`
- Resolvable category relation count: `158`
- Orphan service-category count: `0`
- Duplicate ServiceGuid group count: `0`
- Duplicate normalized service-name/category group count: `0`

Sanitized latest relevant local outbox counts:

- `TblService` / `I`: `158`
- `TblService` / `U`: `160`
- `TblServiceCategory` / `I`: `18`
- `TblServiceCategory` / `U`: `3`
- `TblSetupServicesMethod` / `I`: `2`

## 4. Workbook-versus-target safe counts

Workbook: `E:\NailSallon_RS\MigrationDb\ServiceCategory\ServiceMigration.xlsx`

- Workbook Service rows: `158`
- Workbook `IsActive=true`: `0`
- Workbook `IsActive=false`: `0`
- Workbook `IsActive` blank/other string values: `158`
- Workbook distinct business keys: `158`
- Target distinct business keys: `158`
- Business key matches: `156`
- Workbook business keys missing from target: `2`
- Target rows not in workbook by business key: `2`
- Workbook distinct source GUIDs: `158`
- Source GUIDs matching target ServiceGuid: `0`
- Source GUIDs missing from target ServiceGuid: `158`
- Target active true: `0`
- Target active false: `158`

The source GUID mismatch is expected because the current local service creation path generates new target `ServiceGuid` values.

## 5. Exact Service UI load path

```text
Services_aMain_UC.Btn_New_Service_Click
-> CategoryServiceManager_UC constructed
-> CategoriesTab + ServicesTab + optional MigrationTab initialized
-> CategoryServiceManager_UC.EnsureDataLoadedAsync
-> CategoriesTab.ForceLoadAsync
-> ServicesTab.ForceLoadAsync
-> CatalogServicesTab_UC.LoadOrRefreshAsync
-> CategoryServiceManagementLocalService.LoadCategoriesAsync(includeInactive: true)
-> CatalogManagerUiLoadHelper selection
-> CatalogServicesTab_UC.LoadServicesForSelectedCategoryAsync
-> CategoryServiceManagementLocalService.LoadServicesByCategoryAsync(categoryGuid, includeInactive: IncludeInactiveServices)
-> _services ObservableCollection update
-> GridServices.ItemsSource
```

Post-import refresh path after the correction:

```text
ServiceCategoryMigration_UC.ImportServices
-> ImportServicesAsync result
-> if Committed=true and RequiresUiRefresh=true
-> ServicesImported event
-> CategoryServiceManager_UC handler
-> ServicesTab.ForceLoadAsync
-> existing Service list reload
```

## 6. Exact query/filter predicates

Observed local Service display filters:

- Tenant: local catalog service resolves the active tenant and queries rows for that tenant.
- Category: `LoadServicesByCategoryAsync` loads rows for the selected `ServiceCategoryGuid`.
- Active: when `includeInactive=false`, the query applies `x.IsActive`.
- Include inactive toggle: `CatalogServicesTab_UC.IncludeInactiveServices => ChkShowInactiveServices.IsChecked == true`.
- Binding: `GridServices.ItemsSource = _services`.

The active filter is the proven visibility blocker because the database currently has `0` active rows and `158` inactive rows.

No active `IsDeleted`, status, ServiceType, remote API, or Platform authorization predicate was found in the local Service display path.

## 7. Import path classification when applicable

Import was not run automatically in this task.

Physical import classification from read-only evidence:

- Services were imported into the target database.
- They were not imported to a wrong tenant/database.
- They did not roll back to zero rows.
- They were not orphaned from categories.
- The imported rows are currently inactive and are therefore hidden by the default UI filter.
- Hash-only comparison found 2 stable business-key mismatches requiring review.

## 8. Root cause

Two issues were proven:

1. Physical visibility issue: all 158 target Service rows are inactive, while the default Service UI query filters to active rows only.
2. Import parser defect: the workbook `IsActive` column contains non-boolean string values for all rows. The previous boolean parser converted unknown strings to `false`, causing imported Services to become inactive instead of falling back to the existing default of active.

The UI also needed a committed import refresh signal so successful Service imports refresh the existing Service tab without restarting WPF.

## 9. Source correction and UI refresh behavior

Corrected source behavior:

- Unknown boolean workbook text now returns `null` instead of `false`, preserving existing default values such as `IsActive = true`.
- Service import result now includes sanitized commit/refresh proof fields.
- Migration UI raises refresh events only when the import is committed and refresh is required.
- `CategoryServiceManager_UC` handles those events by reusing existing `ForceLoadAsync` methods.
- No second Service catalog/cache, remote event bus, API dependency, or migration framework was introduced.

## 10. Structured import/result contract

`ServiceCategoryMigrationCommitResult` now exposes/uses:

- `Succeeded`
- `StageId`
- `ResultCode`
- `CreateCount`
- `UpdateCount`
- `SkipCount`
- `ErrorCount`
- `Committed`
- `TargetTenantResolved`
- `ServiceRowsAfterCommitCount`
- `RequiresUiRefresh`

The UI refreshes only when `Committed=true` and `RequiresUiRefresh=true`.

## 11. Exact files changed

OBM source/test files changed for prompt066:

- `E:\Project2026\4POS\NailSalonNet8\Services\ServiceCategoryMigration\Domain\ServiceCategoryMigrationModels.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\ServiceCategoryMigration\Application\ServiceCategoryMigrationService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\ServiceCategoryMigration\Infrastructure\ServiceCategoryExcelMigrationWorkbook.cs`
- `E:\Project2026\4POS\NailSalonNet8\MyUserControls\ServiceCategoryMigration_UC.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\MyUserControls\CategoryServiceManager_UC.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\ServiceCategoryMigration\ServiceCategoryExcelMigrationTests.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\ServiceCategoryMigration\ServiceCategoryMigrationPreviewTests.cs`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V010\CURRENT_TASK_V010.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V010\CURRENT_RESULT_V010.md`

Evidence files created under:

- `E:\Project2026\RecoveryReports\ServiceCategoryMigration\ServiceDbUiVerificationV001`

Only this coordination report is committed/pushed in the coordination repository. OBM source is not committed or pushed.

## 12. Build/test counts

Build:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal
```

Result: `PASS`

- Warnings: `0`
- Errors: `0`

Focused tests:

```text
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~ServiceCategoryMigration|FullyQualifiedName~ServiceMigration|FullyQualifiedName~ServiceManager|FullyQualifiedName~ServiceLoad|FullyQualifiedName~ServiceRefresh|FullyQualifiedName~IndependentMigration" -v minimal
```

Result: `PASS`

- Failed: `0`
- Passed: `19`
- Skipped: `0`
- Total: `19`

## 13. Evidence folder/hashes

Evidence folder:

`E:\Project2026\RecoveryReports\ServiceCategoryMigration\ServiceDbUiVerificationV001`

SHA-256 manifest:

```text
4D66B983B23C605F0FE48D0F18C62905AC64F8AA4679A2E7F08375493AD9A075  import-to-ui-refresh-flow.mmd
AEC329FEF69FD42641C3921C282DA77F476344D51E154975A1841FDD90584BEB  physical-db-service-counts.json
ACEE6988AE47F0B4C3080BADDE482A139EBC4FF4CF6BAE894AC3E0EBCF6BC724  README.md
48FCEB9F164737084FA1204B1900F59197FD18581DF5CBF346574113134B6310  safe-test-results.txt
B10F7EE2E8854305FB4F56C6126B66CBBE51DE0634FD28BEF77340A7B789B10D  service-ui-filter-inventory.csv
C9692E45B4BA33F7E7E5BF6A1BE5C0F5138CB85982FA56EEC4652661D22FCBDC  service-ui-load-path.mmd
2250499A1FA9A7357D293AE84A6D5143131C06AB429B341486178239DE4286F9  workbook-vs-db-counts.json
```

## 14. No DB/import/process/source-push mutation proof

- No workbook import was run automatically.
- No WPF process was launched or clicked automatically.
- No PostgreSQL mutation was performed by this task; inspection was read-only and rolled back.
- No source/reference database mutation was performed.
- No API/PIN/DB credential was changed.
- No service names, raw GUIDs, credentials, connection strings, or business row contents were printed into this report.
- No OBM source commit/push was performed.

## 15. Exact operator retest steps

1. Launch WPF manually.
2. Open `Category / Service Manager (New)`.
3. Open the `Services` tab.
4. Enable `Show inactive services`.
5. Verify imported Services become visible by category without requiring an API connection.
6. Review the 2 hash-only business-key mismatches before data acceptance.
7. If operator approves data correction, run `Preview Services` from the Migration tab.
8. Stop before `Import Services` unless explicitly approving the data-correction import.
9. If import is approved, click `Import Services` once and verify the Service list refreshes without restarting WPF.

## 16. Coordination commit SHA

This report is committed and pushed in the coordination repository. The exact commit SHA is returned by Codex after `git commit`/`git push`.
