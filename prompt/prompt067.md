# Prompt 067 — Prove why Services remain invisible with Show inactive enabled, then fix the actual UI load/binding path

## Physical operator evidence

The operator opened `Category / Service Manager (New) -> Services` after prompt066.

Visible state:

```text
Category = Weird_Service
Show inactive services = checked
Service grid/list = empty
```

Prompt066 already proved by read-only DB audit:

```text
Target DB = obm_pos_dev_v0_pg
Service rows for active tenant = 158
IsActive=true = 0
IsActive=false = 158
Resolvable category relations = 158
Orphan category relations = 0
```

Therefore the current physical evidence disproves the assumption that simply checking `Show inactive services` is sufficient.

Do not mutate DB to activate rows yet. First prove the exact selected-category/query/binding defect.

## Mandatory documentation gate

Before source/test/doc edits, read completely:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report065.md
report/report066.md
```

Record:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V002
CanonicalDocSha256=<actual current hash>
```

## Exact objective

Determine and correct one of these classes:

```text
A. Selected CategoryGuid does not match the Guid referenced by imported Services.
B. IncludeInactiveServices checkbox state is not propagated to the query.
C. Category selection changed but the Service reload did not run.
D. Query returns rows but ObservableCollection/Grid binding is not updated.
E. UI loads a different tenant/database/context than the DB audit.
F. Another proven filter/order/cache issue.
```

Do not guess. Capture counts at every stage.

## Phase 1 — read-only DB distribution by category

Using the protected local DB credential, inspect only inside:

```text
BEGIN TRANSACTION READ ONLY;
...
ROLLBACK;
```

For the active tenant, record sanitized counts only:

```text
Category row count
Service row count
Service count grouped by target ServiceCategoryGuid
Count of categories with >0 Services
Count of categories with 0 Services
Selected visible category business key/hash
Selected category Guid equality to service-referenced category Guid
Selected category total services
Selected category inactive services
Selected category active services
```

Do not print category/service names, raw GUIDs, connection strings, or business rows.

The physical dropdown label `Weird_Service` may be used only to locate the row locally; do not print private row content in the public report.

## Phase 2 — trace the exact UI path

Audit and instrument safely:

```text
CategoryServiceManager_UC
CatalogServicesTab_UC
Category selector ItemsSource/SelectedItem/SelectedValue
ChkShowInactiveServices Checked/Unchecked handlers
ForceLoadAsync
LoadOrRefreshAsync
LoadServicesForSelectedCategoryAsync
CategoryServiceManagementLocalService.LoadServicesByCategoryAsync
ObservableCollection update
GridServices.ItemsSource
```

Record exact pre-change behavior:

```text
selected category object type
selected category Guid field used
includeInactive argument passed
query tenant Guid
query category Guid
DB rows returned
rows after in-memory filtering/sorting
ObservableCollection count after assignment
GridServices.Items.Count after assignment when testable without WPF launch
```

Find every early return, stale `_isLoaded`/`_isRefreshing` guard, cancellation path, exception swallow, or dispatcher issue.

## Phase 3 — test the checkbox behavior

Prove whether checking `Show inactive services` triggers a reload.

Required behavior:

```text
unchecked -> query includeInactive=false
checked   -> query includeInactive=true and reload immediately
```

If the checkbox is bound only to a property but has no reload event, wire `Checked` and `Unchecked` to the existing reload method.

Do not create a second service list/cache.

## Phase 4 — category identity correctness

Audit import and UI category identity together:

```text
TblService.ServiceCategoryGuid
TblServiceCategory.ServiceCategoryGuid
migration category mapping
service import target-category resolution
category dropdown selected object Guid
```

If Services point to valid categories but the dropdown contains duplicate/logically equivalent categories with different GUIDs, classify and report counts.

Do not silently remap or mutate DB in this prompt.

If the issue is a source/target business-key mismatch from the 2 prompt066 mismatches, keep the fix read-only/source-only and return a precise review verdict rather than changing DB.

## Phase 5 — minimal correction

Apply only the proven source/UI correction, such as:

```text
- reload on checkbox Checked/Unchecked;
- remove stale loaded-state guard;
- use SelectedItem.ServiceCategoryGuid rather than SelectedValue/path mismatch;
- pass includeInactive=true correctly;
- refresh ObservableCollection on dispatcher;
- clear and refill the existing collection;
- reselect the same logical category after refresh;
- show safe empty-state diagnostics when selected category truly has zero Services.
```

Do not activate Services or mutate DB.

## Phase 6 — diagnostics

Add a safe non-modal diagnostic/status line in the Services tab for Development/diagnostic builds only, or structured test-visible result, containing counts only:

```text
SelectedCategoryResolved=True/False
IncludeInactive=True/False
QueryRows=<count>
DisplayedRows=<count>
ResultCode=<safe code>
```

No raw GUIDs or names.

Valid result codes may include:

```text
SERVICE_LIST_READY
SERVICE_LIST_SELECTED_CATEGORY_EMPTY
SERVICE_LIST_CATEGORY_NOT_RESOLVED
SERVICE_LIST_QUERY_FAILED
```

Do not add a startup modal.

## Tests

Add focused tests for:

```text
selected category with inactive Services + checkbox checked -> rows displayed
selected category with inactive Services + checkbox unchecked -> zero rows
checkbox Checked triggers reload
checkbox Unchecked triggers reload
selected category Guid is passed exactly to query
query rows populate ObservableCollection/Grid model
category with zero Services -> safe empty result, not stale previous rows
changing category reloads Services
post-import refresh preserves selected logical category when possible
no API dependency
no DB mutation
```

Use EF InMemory/disposable test harness only.

## Evidence folder

Create the next available folder:

```text
E:\Project2026\RecoveryReports\ServiceCategoryMigration\ServiceUiBindingFixV001
```

Preserve:

```text
README.md
SHA256SUMS.txt
physical-category-service-counts.json
ui-load-call-chain.mmd
checkbox-reload-proof.md
selected-category-query-proof.md
safe-test-results.txt
```

No raw names/GUIDs/business rows/secrets.

## Required build/tests

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~ServiceManager|FullyQualifiedName~ServiceLoad|FullyQualifiedName~ServiceRefresh|FullyQualifiedName~ServiceCategoryMigration|FullyQualifiedName~CatalogServices|FullyQualifiedName~InactiveService" -v minimal
```

## Prohibited actions

Do not:

```text
mutate PostgreSQL
activate/deactivate Services
run workbook import
launch/click WPF automatically
change API/PIN/DB credentials
print raw names/GUIDs/connection strings
commit/push OBM source
```

## Report 067

Create and push:

```text
report/report067.md
```

Required sections:

1. Verdict.
2. DOCS_READ_BEFORE_CODE_GATE proof.
3. Physical screenshot evidence.
4. Read-only selected-category DB counts.
5. Exact root-cause classification.
6. Checkbox event/reload proof.
7. Selected CategoryGuid/query proof.
8. Query rows versus displayed rows.
9. Minimal source correction.
10. Exact files changed.
11. Build/test counts.
12. Evidence folder/hashes.
13. No DB/import/process/source-push mutation proof.
14. Exact operator retest steps.
15. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_SERVICE_LIST_INACTIVE_FILTER_AND_BINDING_READY_FOR_PHYSICAL_RETEST
```

```text
OBM_POS_SERVICE_CATEGORY_GUID_MISMATCH_REVIEW_REQUIRED
```

```text
BLOCKED_SERVICE_LIST_UI_LOAD_ROOT_CAUSE
```
