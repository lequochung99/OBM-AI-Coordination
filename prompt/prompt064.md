# Prompt 064 — Fix Service/Category Migration Preview InvalidOperationException

## Physical operator evidence

The operator physically opened the new `Migration` tab in the Category / Service Manager and clicked `Preview Migration`.

Visible result:

```text
Migration action failed.
Safe messages:
Migration action failed: InvalidOperationException
```

Summary remained zero:

```text
Categories = 0
Services = 0
Category Creates = 0
Category Updates = 0
Service Creates = 0
Service Updates = 0
```

Workbook paths displayed in the UI:

```text
E:\NailSallon_RS\MigrationDb\ServiceCategory\CategoryMigration.xlsx
E:\NailSallon_RS\MigrationDb\ServiceCategory\ServiceMigration.xlsx
```

Both files were created by prompt063 and export verification passed:

```text
Category rows = 18
Service rows = 158
```

The operator has not clicked Import. Target DB must remain unchanged.

## Objective

Make `Preview Migration` work physically and display the real preview for both workbooks.

Required outcome:

```text
Categories = 18
Services = 158
Category/Create/Update/Skip/Error counts populated
Service/Create/Update/Skip/Error counts populated
preview grids populated
Import button enabled only when preview has no blocking errors
```

Do not import automatically.

## Mandatory documentation gate

Before editing source/tests/docs/project files, read completely:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report063.md
```

Record:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V002
CanonicalDocSha256=<actual current hash>
```

## First task — identify exact exception source

Do not guess from `InvalidOperationException` alone.

Trace the click path end to end:

```text
ServiceCategoryMigration_UC Preview button
-> event handler/command
-> IServiceCategoryMigrationService
-> workbook reader
-> active tenant/runtime context
-> target DB preview query
-> row mapping/status calculation
-> UI result binding
```

Capture safely:

```text
StageId
ResultCode
ExceptionType
InnerExceptionType
SafeMessage
source file/method/line
call stack without secrets/business values
```

Do not print workbook row contents, service/category names, GUIDs, passwords, or connection strings.

## Audit likely failure boundaries

Inspect and classify each boundary:

### A. Workbook access

Verify:

```text
both files exist
files are readable and not locked
Data sheet exists
Manifest sheet exists
required headers exist
row counts are 18 and 158
workbook library/license initialization is valid
```

### B. Active tenant/runtime context

Verify:

```text
CompanyInfo.TenantGuid is populated
CompanyInfo.PosGuid is populated
current local runtime tenant is resolvable
preview service receives the active TenantGuid
```

Do not require API authorization.

### C. Dependency injection/lifetime

Verify the UI receives exactly one working `IServiceCategoryMigrationService` instance and that required DB/context/workbook dependencies are registered with correct lifetimes.

Find:

```text
missing registration
duplicate registration
scope mismatch
service provider created twice
null factory/service locator fallback
UI constructed outside DI
```

### D. Preview state assumptions

Search all `InvalidOperationException` throws and unsafe LINQ calls in the migration path:

```text
Single()
First()
FirstAsync()
SingleAsync()
GetRequiredService()
throw new InvalidOperationException
```

Replace false assumptions with explicit validation/result codes where appropriate.

### E. Target DB schema/query

Preview may read target DB but must not mutate it.

Verify target tables/columns and query filters match current schema:

```text
TblServiceCategory
TblService
TenantGuid
ServiceCategoryGuid relationship
active/deleted/status fields
```

Use read-only queries only during investigation.

## Required structured preview result

Add or correct a structured result contract such as:

```text
ServiceCategoryMigrationPreviewResult
- Succeeded
- StageId
- ResultCode
- SafeMessage
- CategoryRows
- ServiceRows
- CategoryCreateCount
- CategoryUpdateCount
- CategorySkipCount
- CategoryErrorCount
- ServiceCreateCount
- ServiceUpdateCount
- ServiceSkipCount
- ServiceErrorCount
- BlockingErrors
```

The UI must render this result instead of collapsing every exception to only `InvalidOperationException`.

Expected safe stages include:

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

Do not show raw exception messages to the operator when they could contain local paths or data. Show a safe actionable message plus stage/result code.

## Preview semantics

Preview must remain read-only.

Required ordering:

```text
1. Read/validate Category workbook
2. Read/validate Service workbook
3. Resolve current active target tenant
4. Read current target categories/services
5. Build category preview
6. Build service preview using category mapping
7. Display summary and row grids
```

Service preview must report `Error` when its category cannot be resolved. It must not auto-create a fake category or silently map to the first category.

## Import button behavior

Before successful preview:

```text
Import Selected Workbook = disabled
```

After successful preview:

```text
No blocking errors -> enabled
Blocking errors -> disabled
```

Do not allow import to run against stale preview after file path changes. Changing either workbook path must invalidate preview and disable Import until Preview runs again.

## Tests

Add focused tests for at least:

```text
valid prompt063 workbooks -> preview succeeds
category count = 18
service count = 158
active tenant resolved from local runtime context
API unavailable/401 does not block preview
missing Data sheet -> explicit stage/result
missing Manifest sheet -> explicit stage/result
missing required header -> explicit stage/result
missing workbook file -> explicit stage/result
unresolved service category -> Service Error row
preview performs no INSERT/UPDATE/DELETE
file path change invalidates preview
Import disabled before preview
Import enabled only after successful no-error preview
InvalidOperationException is not exposed as the only UI message
```

Use disposable/in-memory/test target data where possible. Do not mutate `obm_pos_dev_v0_pg` in automated tests.

## Physical diagnostic option

Do not launch WPF automatically if an operator instance may be active.

If the exact root cause cannot be proven statically, add safe diagnostics and return:

```text
OBM_POS_SERVICE_CATEGORY_MIGRATION_PREVIEW_DIAGNOSTICS_READY
```

The operator will rerun Preview and provide the displayed StageId/ResultCode.

Prefer fully fixing the root cause in this prompt when source/test evidence is sufficient.

## Build and test

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~ServiceCategoryMigration|FullyQualifiedName~ServiceMigration|FullyQualifiedName~CategoryMigration|FullyQualifiedName~ExcelMigration|FullyQualifiedName~MigrationTab|FullyQualifiedName~MigrationPreview" -v minimal
```

## Evidence

Create the next available versioned folder:

```text
E:\Project2026\RecoveryReports\ServiceCategoryMigration\PreviewFixV001
```

Never overwrite.

Include:

```text
README.md
SHA256SUMS.txt
preview-call-chain.mmd
exception-root-cause.md
workbook-validation-counts.json
safe-preview-result-example.json
target-readonly-proof.md
test-results.txt
```

No workbook row contents or secrets.

## Safety

Do not:

```text
import into target DB automatically
mutate source DB
mutate obm_pos_dev_v0_pg during investigation/tests
launch WPF automatically
change API tokens
change PIN seed work
resolve prompt062 Payroll blocker
print business rows/secrets
commit/push OBM source
```

## Documentation updates

Canonical V002 architecture remains unchanged.

Preserve current task/result files under the next history version before updating them.

Update `CURRENT_RESULT.md` with the proven preview root cause and fix.

Update `CURRENT_TASK.md` to physical Preview retest, then Category import and Service import only after preview PASS.

## Report 064

Create and push:

```text
report/report064.md
```

Required sections:

1. Verdict.
2. DOCS_READ_BEFORE_CODE_GATE proof.
3. Physical InvalidOperationException evidence.
4. Exact exception root cause and callsite.
5. Workbook validation results/counts.
6. Active tenant/context proof.
7. DI/lifetime proof.
8. Target DB read-only preview proof.
9. Structured StageId/ResultCode contract.
10. Preview counts and semantics.
11. Import-button state behavior.
12. Exact files changed.
13. Build/test counts.
14. Evidence folder/hashes.
15. No DB import/mutation/process/source-push proof.
16. Exact operator physical retest steps.
17. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_SERVICE_CATEGORY_MIGRATION_PREVIEW_READY_FOR_PHYSICAL_RETEST
```

```text
OBM_POS_SERVICE_CATEGORY_MIGRATION_PREVIEW_DIAGNOSTICS_READY
```

```text
BLOCKED_SERVICE_CATEGORY_MIGRATION_PREVIEW_ROOT_CAUSE
```

```text
BLOCKED_SERVICE_CATEGORY_MIGRATION_PREVIEW_BUILD_OR_TEST
```
