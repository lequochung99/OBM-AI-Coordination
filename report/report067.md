# Report 067 - Service UI Selected-Category Binding Proof

## 1. Verdict

`OBM_POS_SERVICE_CATEGORY_GUID_MISMATCH_REVIEW_REQUIRED`

The blank Services grid with Show inactive enabled is not caused by the inactive checkbox alone. Read-only DB proof shows the operator-selected visible category exists and is active, but it has zero Service rows. The 158 imported Services exist under other category GUIDs.

## 2. DOCS_READ_BEFORE_CODE_GATE proof

`DOCS_READ_BEFORE_CODE_GATE=PASS`

Read before source/test/doc edits:

- `E:\Project2026\4POS\NailSalonNet8\AGENTS.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `report/report065.md`
- `report/report066.md`

CanonicalDocVersion: `V002`

CanonicalDocSha256: `916AE7CE71ADA14D768DE1D11E7E5F3604416E8176AA3C59479353481AA2DFB2`

## 3. Physical screenshot evidence

The operator evidence in prompt067 reported:

- `Category / Service Manager (New) -> Services`
- A specific visible category label was selected.
- `Show inactive services` was checked.
- Service grid/list was empty.

This report avoids printing the raw category label, category GUID, service names, raw service GUIDs, connection strings, or business rows.

## 4. Read-only selected-category DB counts

Target database: `obm_pos_dev_v0_pg`

Inspection method:

- Used protected local credential path.
- Executed inside `BEGIN TRANSACTION READ ONLY`.
- Ended with `ROLLBACK`.
- Stored only counts and hashes.

Safe counts:

- Active tenant present: `true`
- Category row count: `18`
- Service row count: `158`
- Categories with services: `14`
- Categories with zero services: `4`
- Selected category row count: `1`
- Selected category active: `true`
- Selected category total services: `0`
- Selected category active services: `0`
- Selected category inactive services: `0`
- Selected category Guid equality to a service-referenced category Guid: `false`

Service distribution:

- 14 category GUID hashes have service rows.
- All 158 service rows are inactive.
- Every service-referenced category resolves to a category row.

## 5. Exact root-cause classification

Root cause class:

`A_SELECTED_CATEGORYGUID_DOES_NOT_MATCH_IMPORTED_SERVICE_REFERENCED_CATEGORYGUIDS`

Prompt066 proved aggregate DB presence. Prompt067 proves the selected visible category is one of the categories with zero Service rows. Therefore checking `Show inactive services` cannot display rows for that selected category.

## 6. Checkbox event/reload proof

Pre-change source already had:

- `Checked="InactiveFilter_Changed"`
- `Unchecked="InactiveFilter_Changed"`
- `InactiveFilter_Changed` calls the existing selected-category reload path.

Prompt067 source correction:

- `LoadServicesForSelectedCategoryAsync` now captures `IncludeInactiveServices` on the UI thread together with `SelectedFilterCategory`.
- The captured boolean is passed directly to `LoadServicesByCategoryAsync`.
- The safe status line reports `IncludeInactive=True/False`.

Expected behavior:

- Unchecked: `includeInactive=false`.
- Checked: `includeInactive=true`.
- Checked plus selected zero-service category: `SERVICE_LIST_SELECTED_CATEGORY_EMPTY`.

## 7. Selected CategoryGuid/query proof

UI path:

```text
CboCategoryFilter.SelectedItem
-> CategoryManagementRow.ServiceCategoryGuid
-> LoadServicesForSelectedCategoryAsync
-> LoadServicesByCategoryAsync(category.ServiceCategoryGuid, includeInactive)
-> TenantGuid + selected ServiceCategoryGuid query
```

Read-only proof:

- The selected category row exists.
- The selected category is active.
- The selected category has zero active and zero inactive Service rows.
- The selected category GUID does not equal any service-referenced category GUID.

No DB remap or automatic category reselection was performed.

## 8. Query rows versus displayed rows

For the operator-selected category:

- SelectedCategoryResolved: `true`
- IncludeInactive: expected `true` when the checkbox is checked
- QueryRows: `0`
- DisplayedRows: `0`
- ResultCode: `SERVICE_LIST_SELECTED_CATEGORY_EMPTY`

For a category with imported Services:

- IncludeInactive: `true`
- QueryRows: expected `>0`
- DisplayedRows: expected `>0`
- ResultCode: `SERVICE_LIST_READY`

## 9. Minimal source correction

Implemented the smallest source/UI correction:

- Added count-only `CatalogServiceListLoadStatus`.
- Added safe result codes:
  - `SERVICE_LIST_READY`
  - `SERVICE_LIST_SELECTED_CATEGORY_EMPTY`
  - `SERVICE_LIST_CATEGORY_NOT_RESOLVED`
  - `SERVICE_LIST_QUERY_FAILED`
  - `SERVICE_LIST_NOT_STARTED`
- Added helper to format safe status text with no names/GUIDs.
- Added `TxtServiceLoadStatus` to the Services tab.
- Captured selected category and checkbox state on the UI thread.
- Updated status after service query, no-category, and query failure paths.

No database mutation, activation, remap, auto-select, new catalog/cache, or API dependency was introduced.

## 10. Exact files changed

OBM source/test/doc files changed but not committed/pushed:

- `E:\Project2026\4POS\NailSalonNet8\Services\Catalog\CatalogManagementModels.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Catalog\CatalogManagerUiLoadHelper.cs`
- `E:\Project2026\4POS\NailSalonNet8\MyUserControls\CatalogServicesTab_UC.xaml`
- `E:\Project2026\4POS\NailSalonNet8\MyUserControls\CatalogServicesTab_UC.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\CatalogManagerUiLoadHelperTests.cs`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V011\CURRENT_TASK_V011.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V011\CURRENT_RESULT_V011.md`

Evidence files created under:

- `E:\Project2026\RecoveryReports\ServiceCategoryMigration\ServiceUiBindingFixV001`

Only this coordination report is committed/pushed in the coordination repository.

## 11. Build/test counts

Build:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal
```

Result: `PASS`

- Errors: `0`
- Warnings observed: `176` pre-existing/project warnings.

Focused tests:

```text
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~ServiceManager|FullyQualifiedName~ServiceLoad|FullyQualifiedName~ServiceRefresh|FullyQualifiedName~ServiceCategoryMigration|FullyQualifiedName~CatalogServices|FullyQualifiedName~InactiveService" -v minimal
```

Result: `PASS`

- Failed: `0`
- Passed: `24`
- Skipped: `0`
- Total: `24`

Note: the first parallel test run hit a transient output lock because build/test ran simultaneously. The same focused test command was rerun serially and passed.

## 12. Evidence folder/hashes

Evidence folder:

`E:\Project2026\RecoveryReports\ServiceCategoryMigration\ServiceUiBindingFixV001`

SHA-256 manifest:

```text
BA3590012A1082FA45F21E78C1C3D2287ACA472C75C05AF98A7CA19DE0ADFBC6  checkbox-reload-proof.md
17F5102B67ADA2F58D6657743E9DFF10CA9179128047AADA4A59C109A0258AEA  physical-category-service-counts.json
0D7C9C7EEE3B1490068D6DF0AF66FF00642674F52B4293019E31BA4B3B632892  README.md
06AAF3B9F2D4D47630C93DD0EE81D79F6AA92D7C269E9B1CC766AEF5A71FC174  safe-test-results.txt
984C3E7A013F829292A046FCC0582AE8EBF3F5148C7D98CB07E3C0E065365D63  selected-category-query-proof.md
79DFCA57EFB7081946E4291EB6B8B85DEA6180A557C6D318D165A4CA90265881  ui-load-call-chain.mmd
```

## 13. No DB/import/process/source-push mutation proof

- No PostgreSQL mutation was performed.
- Read-only DB inspection used rollback.
- No Service activation/deactivation was performed.
- No workbook import was run.
- No WPF process was launched or clicked automatically.
- No API/PIN/DB credential was changed.
- No raw category names, service names, GUIDs, connection strings, credentials, or business row contents were recorded.
- No OBM source commit/push was performed.

## 14. Exact operator retest steps

1. Launch WPF manually.
2. Open `Category / Service Manager (New) -> Services`.
3. Keep `Show inactive services` checked.
4. Select the same category used in the prompt067 physical observation.
5. Verify the safe status line shows:
   `SelectedCategoryResolved=True; IncludeInactive=True; QueryRows=0; DisplayedRows=0; ResultCode=SERVICE_LIST_SELECTED_CATEGORY_EMPTY`
6. Select a category with nonzero service count.
7. Verify inactive Services display and status shows:
   `ResultCode=SERVICE_LIST_READY`
8. Stop before any DB remap, activation, or import unless separately approved.

## 15. Coordination commit SHA

This report is committed and pushed in the coordination repository. The exact commit SHA is returned by Codex after `git commit`/`git push`.
