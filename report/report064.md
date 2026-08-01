# REPORT-064 Service/Category Migration Preview Fix V001

## 1. Verdict

OBM_POS_SERVICE_CATEGORY_MIGRATION_PREVIEW_READY_FOR_PHYSICAL_RETEST

## 2. DOCS_READ_BEFORE_CODE_GATE proof

- DOCS_READ_BEFORE_CODE_GATE=PASS
- CanonicalDocVersion=V002
- CanonicalDocSha256=916AE7CE71ADA14D768DE1D11E7E5F3604416E8176AA3C59479353481AA2DFB2
- Read before source changes:
  - `E:\Project2026\4POS\NailSalonNet8\AGENTS.md`
  - `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md`
  - `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
  - `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
  - `report/report063.md`

## 3. Physical InvalidOperationException evidence

Operator evidence from prompt064:

```text
Migration action failed.
Safe messages:
Migration action failed: InvalidOperationException
```

The UI summary remained zero even though prompt063 workbooks existed and had verified exports:

```text
Categories = 0
Services = 0
Category Creates = 0
Category Updates = 0
Service Creates = 0
Service Updates = 0
```

## 4. Exact exception root cause and callsite

Call path:

```text
ServiceCategoryMigration_UC.Preview_Click
-> IServiceCategoryMigrationService.PreviewAsync
-> ServiceCategoryExcelMigrationWorkbook.ReadCategories
-> ServiceCategoryExcelMigrationWorkbook.ReadServices
-> ICategoryServiceManagementLocalService.LoadCategoriesAsync
-> ResolveTenantGuid(existingCategories)
-> InvalidOperationException: Current tenant identity is unavailable.
```

Root cause:
- Preview resolved the active tenant from the target category list.
- On an empty target tenant, `existingCategories` was empty.
- Tenant resolution threw before workbook counts and preview rows could be calculated.

Corrected callsite:
- `E:\Project2026\4POS\NailSalonNet8\Services\ServiceCategoryMigration\Application\ServiceCategoryMigrationService.cs`
- `PreviewAsync` now calls `ResolveActiveTenantGuid()` before target reads.
- `ResolveActiveTenantGuid()` uses station identity first and `CompanyInfo.TenantGuid` second.

## 5. Workbook validation results/counts

Expected physical workbook paths:

```text
E:\NailSallon_RS\MigrationDb\ServiceCategory\CategoryMigration.xlsx
E:\NailSallon_RS\MigrationDb\ServiceCategory\ServiceMigration.xlsx
```

Expected physical counts:

```text
Category rows = 18
Service rows = 158
```

Automated proof generated equivalent prompt063-shaped workbooks:

```text
testWorkbookCategoryRows = 18
testWorkbookServiceRows = 158
categoryCreateCountOnEmptyTarget = 18
serviceCreateCountOnEmptyTarget = 158
```

Validation failure mappings proved:
- missing workbook -> `SERVICE_CATEGORY_MIGRATION_WORKBOOK_MISSING`
- missing `Data` sheet -> `SERVICE_CATEGORY_MIGRATION_DATA_SHEET_MISSING`
- missing `Manifest` sheet -> `SERVICE_CATEGORY_MIGRATION_MANIFEST_SHEET_MISSING`
- missing required header -> `SERVICE_CATEGORY_MIGRATION_REQUIRED_HEADER_MISSING`
- unresolved service category -> `SERVICE_CATEGORY_MIGRATION_PREVIEW_BLOCKED`

## 6. Active tenant/context proof

- Preview no longer requires target categories to resolve the tenant.
- Tenant resolution order:
  1. `PosStationLocalSettings.LoadStationIdentity()?.TenantGuid`
  2. `CompanyInfo.TenantGuid`
- Focused tests set local runtime tenant context and prove preview succeeds with an empty target category set.
- API authorization is not required for preview.

## 7. DI/lifetime proof

- The existing UI continues to receive `IServiceCategoryMigrationService` through constructor injection.
- No service-provider fallback, duplicate service provider, or alternate runtime lane was introduced.
- Preview uses the injected catalog service and injected `IDbContextFactory<eNailSalonDbContext>`.
- Import remains on the same injected service path.

## 8. Target DB read-only preview proof

Preview path:
- Reads workbook files.
- Loads target categories through existing catalog read API.
- Reads target services with `TblServices.AsNoTracking()`.
- Builds preview rows/counts in memory.

Preview path does not call:
- `CreateCategoryAsync`
- `UpdateCategoryAsync`
- `CreateServiceAsync`
- `UpdateServiceAsync`
- `SaveChanges`
- `ImportAsync`

Automated test proof:
- EF InMemory target category/service counts remain zero after preview.

## 9. Structured StageId/ResultCode contract

Added/confirmed structured result fields:

```text
Succeeded
StageId
ResultCode
SafeMessage
CategoryRows
ServiceRows
CategoryCreateCount
CategoryUpdateCount
CategorySkipCount
CategoryErrorCount
ServiceCreateCount
ServiceUpdateCount
ServiceSkipCount
ServiceErrorCount
BlockingErrors
CategoryPreviewRows
ServicePreviewRows
```

Stage IDs:

```text
ResolveActiveTenant
OpenCategoryWorkbook
ValidateCategoryWorkbook
OpenServiceWorkbook
ValidateServiceWorkbook
LoadTargetCategories
LoadTargetServices
BuildCategoryPreview
BuildServicePreview
Complete
```

## 10. Preview counts and semantics

Expected successful physical preview on the empty target:

```text
StageId = Complete
ResultCode = SERVICE_CATEGORY_MIGRATION_PREVIEW_READY
Categories = 18
Services = 158
CategoryCreateCount = 18
CategoryUpdateCount = 0
CategorySkipCount = 0
CategoryErrorCount = 0
ServiceCreateCount = 158
ServiceUpdateCount = 0
ServiceSkipCount = 0
ServiceErrorCount = 0
BlockingErrors = 0
ReadyToImport = true
```

Service preview reports `Error` when its source category cannot be resolved from the category workbook. It does not silently map to a first category and does not create fake target categories during preview.

## 11. Import-button state behavior

- Import button remains disabled before preview.
- Import button is enabled only when `ReadyToImport=true`.
- Any workbook path change invalidates the preview and disables import.
- Import refuses stale preview if the current paths differ from the successful preview paths.

## 12. Exact files changed

Source:
- `E:\Project2026\4POS\NailSalonNet8\Services\ServiceCategoryMigration\Domain\ServiceCategoryMigrationModels.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\ServiceCategoryMigration\Application\ServiceCategoryMigrationService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\ServiceCategoryMigration\Infrastructure\ServiceCategoryExcelMigrationWorkbook.cs`
- `E:\Project2026\4POS\NailSalonNet8\MyUserControls\ServiceCategoryMigration_UC.xaml`
- `E:\Project2026\4POS\NailSalonNet8\MyUserControls\ServiceCategoryMigration_UC.xaml.cs`

Tests:
- `E:\Project2026\4POS\NailSalonNet8.Tests\ServiceCategoryMigration\ServiceCategoryMigrationPreviewTests.cs`

Docs/evidence:
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V008\CURRENT_RESULT_V008.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V008\CURRENT_TASK_V008.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\RecoveryReports\ServiceCategoryMigration\PreviewFixV001\README.md`
- `E:\Project2026\RecoveryReports\ServiceCategoryMigration\PreviewFixV001\SHA256SUMS.txt`
- `E:\Project2026\RecoveryReports\ServiceCategoryMigration\PreviewFixV001\preview-call-chain.mmd`
- `E:\Project2026\RecoveryReports\ServiceCategoryMigration\PreviewFixV001\exception-root-cause.md`
- `E:\Project2026\RecoveryReports\ServiceCategoryMigration\PreviewFixV001\workbook-validation-counts.json`
- `E:\Project2026\RecoveryReports\ServiceCategoryMigration\PreviewFixV001\safe-preview-result-example.json`
- `E:\Project2026\RecoveryReports\ServiceCategoryMigration\PreviewFixV001\target-readonly-proof.md`
- `E:\Project2026\RecoveryReports\ServiceCategoryMigration\PreviewFixV001\test-results.txt`

Coordination:
- `report/report064.md`

## 13. Build/test counts

Build:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal
Result: PASS
Errors: 0
```

Focused tests:

```text
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~ServiceCategoryMigration|FullyQualifiedName~ServiceMigration|FullyQualifiedName~CategoryMigration|FullyQualifiedName~ExcelMigration|FullyQualifiedName~MigrationTab|FullyQualifiedName~MigrationPreview" -v minimal
Result: PASS
Passed: 13
Failed: 0
Skipped: 0
```

Note: first non-escalated test run failed before execution because sandboxed MSBuild could not read `C:\Users\lequo\AppData\Local\Microsoft SDKs`; the same command passed with approved escalation.

## 14. Evidence folder/hashes

Evidence folder:

```text
E:\Project2026\RecoveryReports\ServiceCategoryMigration\PreviewFixV001
```

Hashes:

```text
081F1A30ADAF969C9FFAC51CF15EBC39B5E6117153CC0C4D40EC8C68D327C280  exception-root-cause.md
74BD85972B029F02AD6C79F3DEE3E673FF209941FE79B4F391CC07CDE8FE5CAA  preview-call-chain.mmd
8455E00E473603B51038E2E654B2BC10CCE75866A2ABF03611F20223BC5DE48F  README.md
9C9766FA95D958CB8E4D7F823D8FCFD8E824903B93E5EFDD11847F6718A6CD40  safe-preview-result-example.json
01BF6296B437726B557CE229EC9C4FD601C479EA430C0CAB14D51F24BB4DED6D  target-readonly-proof.md
5C77AC6D50D658AF250AED5A1AE3A7D4D393584F5DE68F10E890DA8AD0E11A03  test-results.txt
8BA82B5EFCD96E7F38A7D07D853486C2017DC2678E8308006D1796FBE411D3EC  workbook-validation-counts.json
```

## 15. No DB import/mutation/process/source-push proof

- WPF launched by automation: NO
- Workbook import executed: NO
- Source DB mutation: NO
- Target DB mutation: NO
- `obm_pos_dev_v0_pg` mutation: NO
- API tokens changed: NO
- PIN seed / Payroll blocker changed: NO
- OBM source committed/pushed: NO
- Secrets or connection strings printed: NO
- Business row contents printed: NO

## 16. Exact operator physical retest steps

1. Launch WPF manually.
2. Open Category / Service Manager (New).
3. Open the Migration tab.
4. Confirm workbook paths:
   - `E:\NailSallon_RS\MigrationDb\ServiceCategory\CategoryMigration.xlsx`
   - `E:\NailSallon_RS\MigrationDb\ServiceCategory\ServiceMigration.xlsx`
5. Click `Preview Migration` once.
6. Verify:
   - Stage = `Complete`
   - Result Code = `SERVICE_CATEGORY_MIGRATION_PREVIEW_READY`
   - Categories = `18`
   - Services = `158`
   - Category creates/updates/skips/errors populated
   - Service creates/updates/skips/errors populated
   - Preview grids populated
   - Blocking Errors = `0`
   - Import button enabled only after preview PASS
7. Stop before import unless the operator gives explicit import approval.

## 17. Coordination commit SHA

To be read from the commit that adds this report and returned in the final Codex response.
