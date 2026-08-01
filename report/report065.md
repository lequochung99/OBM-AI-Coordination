# Report 065 — Service/Category Independent Migration Lanes

## 1. Verdict

OBM_POS_SERVICE_CATEGORY_INDEPENDENT_MIGRATION_LANES_READY_FOR_PHYSICAL_RETEST

## 2. DOCS_READ_BEFORE_CODE_GATE Proof

- DOCS_READ_BEFORE_CODE_GATE=PASS
- CanonicalDocVersion=V002
- CanonicalDocSha256=916AE7CE71ADA14D768DE1D11E7E5F3604416E8176AA3C59479353481AA2DFB2
- Canonical architecture document was not versioned or changed for this UI/import-scope correction.

Documents/reports read before source edits:

- `E:\Project2026\4POS\NailSalonNet8\AGENTS.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `report/report063.md`
- `report/report064.md`

## 3. Physical Ambiguous Combined-Lane Evidence

Prompt065 recorded the physical ambiguity:

- Category workbook path and Service workbook path were both active in one UI session.
- UI exposed one `Preview Migration` action and one combined import action.
- Operator could not confidently run Service-only import after Categories already existed.

The source audit confirmed the ambiguity was real behavior, not just wording.

## 4. Exact Pre-Change Combined Preview/Import Behavior

Pre-change call path:

- `ServiceCategoryMigration_UC.Preview_Click`
- `IServiceCategoryMigrationService.PreviewAsync(ServiceCategoryWorkbookPaths)`
- `ServiceCategoryMigrationService.PreviewAsync`
- `ServiceCategoryMigration_UC.Import_Click`
- `IServiceCategoryMigrationService.ImportAsync(ServiceCategoryWorkbookPaths)`
- `ServiceCategoryMigrationService.ImportAsync`

Proven pre-change behavior:

- Preview always read both Category and Service workbooks.
- Import always read both Category and Service workbooks.
- Import processed Categories before Services.
- `Import Selected Workbooks` had no durable independent selection state.
- Scope was inferred from populated paths, creating an unsafe Service-only ambiguity.
- Service preview depended on combined in-memory category/reference state.
- A stale combined preview could be conceptually reused after only one workbook path changed.

## 5. Customer/GiftCard Pattern Audit and Reuse

Audited:

- `E:\Project2026\4POS\NailSalonNet8\MyUserControls\Settings\CustomerMigrationViewModel.cs`
- `E:\Project2026\4POS\NailSalonNet8\MyUserControls\Settings\GiftCardMigrationViewModel.cs`

Reused/equivalent patterns:

- explicit file/path per workflow;
- preview-before-import state gate;
- Create/Update/Skip/Error preview rows;
- stale-preview invalidation after path/target changes;
- safe status/result display;
- focused tests for command state and import scope.

No new generic migration framework was introduced.

## 6. Final Category Lane UI/Commands/State

Category lane now has:

- Category workbook path;
- Browse;
- Clear;
- Preview Categories;
- Import Categories;
- Category summary/messages;
- Category preview grid.

Category state is independent through `LanePreviewState` and tracks path fingerprint, stage, result code, readiness, and preview rows.

## 7. Final Service Lane UI/Commands/State

Service lane now has:

- Service workbook path;
- Browse;
- Clear;
- Preview Services;
- Import Services;
- Service summary/messages;
- Service preview grid.

Service state is independent through its own `LanePreviewState`. Optional Category workbook metadata is read-only for source mapping and is not an import target.

## 8. Explicit Migration Scope API

Public combined import/preview API was replaced with explicit methods:

- `PreviewCategoriesAsync`
- `ImportCategoriesAsync`
- `PreviewServicesAsync`
- `ImportServicesAsync`

Domain model now includes explicit `ServiceCategoryMigrationScope.Categories` and `ServiceCategoryMigrationScope.Services`.

## 9. Category-Existing/Idempotency Behavior

Category preview semantics:

- equal existing target category -> `Skip`;
- existing target category with supported field differences -> `Update`;
- missing target category -> `Create`;
- invalid/colliding row -> `Error`.

Tests prove matching target Categories produce Skip behavior and do not write Services.

## 10. Service Category-Resolution Behavior

Service preview/import resolves target Categories from current target state first, with optional Category workbook metadata as read-only source reference.

If the target Category cannot be resolved, Service preview blocks with:

- `SERVICE_MIGRATION_TARGET_CATEGORY_MISSING`

Service import does not create or update Categories.

## 11. Transaction/Entity-Write Isolation Proof

Implemented isolation:

- Category import reads Category workbook and writes Categories only.
- Service import reads Service workbook, optionally reads Category workbook as metadata, and writes Services only.

Focused tests prove:

- Category import writes zero Services.
- Service import with populated Category path writes zero Categories.

## 12. State Invalidation/Stale-Preview Proof

Implemented lane invalidation:

- Category path change/clear invalidates Category state only.
- Service path change/clear invalidates Service state only.
- Category import invalidates Service preview because target Category resolution can change.
- Service import does not re-import Categories.
- `LanePreviewState.CanImport(...)` rejects stale path/fingerprint imports.

## 13. Exact Files Changed

OBM source/test/doc files changed but not committed/pushed:

- `E:\Project2026\4POS\NailSalonNet8\MyUserControls\ServiceCategoryMigration_UC.xaml`
- `E:\Project2026\4POS\NailSalonNet8\MyUserControls\ServiceCategoryMigration_UC.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\ServiceCategoryMigration\Application\IServiceCategoryMigrationService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\ServiceCategoryMigration\Application\ServiceCategoryMigrationService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\ServiceCategoryMigration\Domain\ServiceCategoryMigrationModels.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\ServiceCategoryMigration\ServiceCategoryMigrationPreviewTests.cs`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V009\CURRENT_TASK_V009.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V009\CURRENT_RESULT_V009.md`

Coordination report created:

- `report/report065.md`

## 14. Build/Test Counts

Build:

- Command: `dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal`
- Result: PASS
- Summary: Build succeeded, 0 warnings, 0 errors.

Tests:

- Command: `dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~ServiceCategoryMigration|FullyQualifiedName~ServiceMigration|FullyQualifiedName~CategoryMigration|FullyQualifiedName~ExcelMigration|FullyQualifiedName~MigrationTab|FullyQualifiedName~MigrationPreview|FullyQualifiedName~IndependentMigration" -v minimal`
- Result: PASS
- Summary: Failed 0, Passed 17, Skipped 0, Total 17.

Note: test build emitted pre-existing nullable/analyzer warnings outside this slice; the filtered tests passed.

## 15. Evidence Folder/Hashes

Evidence folder:

- `E:\Project2026\RecoveryReports\ServiceCategoryMigration\IndependentLanesV001`

SHA-256 manifest:

```text
7623878629ECC60AD1A783229D5C0D6862F060182C586DA2B19DFF6151049593  command-scope-matrix.csv
81AE2A9F553B0D41912D6184B8E6F36A9701C3BA2D303A1023C2FEEF7EE30599  customer-giftcard-pattern-reuse.md
C9A406B9C9CDCBA43573BB54283F613CBF9C08C7A4A585C732350801E106E707  post-change-independent-flow.mmd
401AE36FEC8B0DAB2A99FF8799DA910B5AAFCB72DC965263E518B0E30C02F2F2  pre-change-combined-flow.mmd
1CFF852837016B8E49BD1456DA38B97F2DEFD67053076E3D3CC3909E7B119078  README.md
FCE91245353134F90ED21B89AF1783CDF35D2970DB6B49AD8879BE6B569B7860  safe-test-results.txt
38A8003BD9493A8707D96E7BB03A9ADE939207297314A30A3A51266CDD2983FD  state-invalidation-matrix.csv
```

## 16. No DB/Import/Process/Source-Push Mutation Proof

Not performed:

- No WPF launch.
- No workbook import.
- No `obm_pos_dev_v0_pg` mutation.
- No local POS PostgreSQL connection or script execution for this task.
- No source DB mutation.
- No process stop/start.
- No OBM source commit or push.
- No secrets, connection strings, workbook row contents, or private identifiers printed.

Only this coordination report is intended to be committed/pushed.

## 17. Exact Physical Retest Steps

1. Launch WPF manually.
2. Open Category / Service Manager (New) -> Migration.
3. Confirm Category and Service sections are visually separate.
4. With Categories already present, leave Category path populated or clear it.
5. Click Preview Services only.
6. Verify result code `SERVICE_MIGRATION_PREVIEW_READY`.
7. Verify Service counts and zero Category write intent.
8. If approved, click Import Services only.
9. Verify Category row count/data/outbox did not change.
10. Verify Service rows were imported/updated as expected.

## 18. Coordination Commit SHA

Coordination commit SHA is reported in the final Codex response after commit creation and push. A commit hash cannot be embedded into the same committed file without changing that commit hash.
