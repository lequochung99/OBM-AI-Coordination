# Prompt 065 — Split Category and Service migration into independent lanes and prevent unintended category re-import

## Physical operator evidence

The operator physically opened the prompt064 Migration tab after Category data was already present in the target DB.

Current UI still shows both workbook paths active at the same time:

```text
Category workbook = E:\NailSallon_RS\MigrationDb\ServiceCategory\CategoryMigration.xlsx
Service workbook  = E:\NailSallon_RS\MigrationDb\ServiceCategory\ServiceMigration.xlsx
```

The operator cannot clear/remove the Category workbook path and cannot select Service-only migration with confidence.

The current UI uses one combined preview/import session and one button:

```text
Preview Migration
Import Selected Workbook
```

This creates a real safety problem:

```text
Category already exists/imported
+ Category workbook path remains active
+ operator wants to import only Service
-> UI does not make it explicit whether Category will be re-imported
```

Do not ask the operator to continue importing in this ambiguous state.

## Operator decision

Category and Service migration must become two independent migration lanes.

Required behavior:

```text
Category lane
-> select/clear Category workbook
-> Preview Categories
-> Import Categories only

Service lane
-> select/clear Service workbook
-> Preview Services
-> Import Services only
```

A Service import must never write Categories merely because the Category workbook path is populated.

A Category import must never write Services.

The Category workbook may remain visible as an optional reference source for resolving source category GUID/name relationships during Service preview, but it must not be an active import target unless the operator explicitly selects Category import.

## Mandatory documentation gate

Before editing source, tests, UI, project files, or current documentation, read completely:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report063.md
report/report064.md
```

Record before first source edit:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V002
CanonicalDocSha256=<actual current hash>
```

Canonical architecture does not need a new version for this UI/import-scope correction.

## First task — audit current combined-session behavior

Trace the exact current path:

```text
ServiceCategoryMigration_UC
Preview_Click
Import_Click
IServiceCategoryMigrationService.PreviewAsync
IServiceCategoryMigrationService.ImportAsync
ServiceCategoryMigrationResult / preview model
```

Report exact pre-change behavior:

```text
- whether Preview always reads both workbooks;
- whether Import always processes both categories and services;
- what `Import Selected Workbook` actually selects;
- whether selection state exists but is invisible/implicit;
- whether service import depends on an in-memory category preview;
- whether category rows marked Skip can still create outbox/audit rows;
- whether stale combined preview can be reused after only one path changes.
```

Do not infer from button text; prove the actual call path.

## Required UI redesign

Keep the existing Migration tab, but split it visually and behaviorally.

### Category Migration section

Required controls:

```text
Category workbook path
Browse...
Clear
Preview Categories
Import Categories
Category summary
Category preview grid
```

Default path:

```text
E:\NailSallon_RS\MigrationDb\ServiceCategory\CategoryMigration.xlsx
```

Rules:

```text
- Clear empties only the Category path;
- changing/clearing Category path invalidates only Category preview;
- Import Categories is disabled until Category preview passes;
- Import Categories calls category-only import path;
- category-only import never reads/imports Service workbook;
- after successful category import, Category preview refreshes and rows should become Skip/Update according to target state;
- successful Category import does not automatically trigger Service import.
```

### Service Migration section

Required controls:

```text
Service workbook path
Browse...
Clear
Preview Services
Import Services
Service summary
Service preview grid
```

Default path:

```text
E:\NailSallon_RS\MigrationDb\ServiceCategory\ServiceMigration.xlsx
```

Rules:

```text
- Clear empties only the Service path;
- changing/clearing Service path invalidates only Service preview;
- Import Services is disabled until Service preview passes;
- Import Services calls service-only import path;
- service-only import never creates/updates Categories;
- service-only import resolves each service category against the current target DB first;
- optional Category workbook may be used only as read-only source metadata when needed for source mapping diagnostics;
- if target category is missing, the service row is Error and import is blocked;
- no fake/default category assignment;
- no automatic category create during Service import.
```

## Explicit migration scope model

Introduce or enforce an explicit enum/value such as:

```text
ServiceCategoryMigrationScope.Categories
ServiceCategoryMigrationScope.Services
```

or equivalent clear API methods:

```text
PreviewCategoriesAsync
ImportCategoriesAsync
PreviewServicesAsync
ImportServicesAsync
```

Prefer separate methods if they make the code simpler and harder to misuse.

Do not retain a public combined `ImportAsync` path that can accidentally import both unless there is a proven separate bulk-migration use case. If kept internally, it must require an explicit scope argument and must never infer scope from non-empty paths.

## Category already exists behavior

The current target DB already contains Categories.

Required Category preview semantics:

```text
matching source category already exists and equal
-> Skip

matching source category exists but allowed fields differ
-> Update

source category not found in target
-> Create

collision/tenant mismatch/invalid row
-> Error
```

Re-running Category import must be idempotent:

```text
Skip rows
-> no category mutation
-> no duplicate TblLocalOutbox
-> no duplicate migration audit row if the existing framework treats rerun as already applied
```

The operator must be able to leave Category path populated, preview it as all Skip, and still run Service-only import without Category writes.

## Service category resolution

For each service row, resolve target category with this priority based on proven identifiers:

```text
1. exact source/category GUID mapping when safe for the active tenant;
2. existing migration source map when present;
3. normalized category business key/name only when the existing migration contract explicitly allows it;
4. otherwise Error.
```

Do not use the first category. Do not create a category from Service import.

If Category workbook is cleared, Service preview must still work when all target categories already exist and can be resolved from Service workbook fields/current target DB.

If Service workbook lacks enough category metadata without Category workbook, show a precise safe error explaining that the Category workbook is required as a read-only mapping reference, but still do not import Categories.

## Preview and import state isolation

Maintain separate state objects:

```text
CategoryPreviewState
ServicePreviewState
```

Each must track:

```text
WorkbookPath
WorkbookHash or last-write fingerprint
TenantGuid snapshot
StageId
ResultCode
ReadyToImport
PreviewRows
Create/Update/Skip/Error counts
BlockingErrors
```

Rules:

```text
- changing Category path invalidates only Category state;
- changing Service path invalidates only Service state;
- importing Categories invalidates/refreshes Service preview because target category resolution may change;
- importing Services does not invalidate Category preview unless shared target metadata truly changes;
- no stale preview may be imported after workbook/path/tenant changes;
- each import button must name the exact entity being imported.
```

## Structured result codes

Use precise result codes, for example:

```text
CATEGORY_MIGRATION_PREVIEW_READY
CATEGORY_MIGRATION_IMPORT_SUCCEEDED
CATEGORY_MIGRATION_IMPORT_BLOCKED
SERVICE_MIGRATION_PREVIEW_READY
SERVICE_MIGRATION_IMPORT_SUCCEEDED
SERVICE_MIGRATION_IMPORT_BLOCKED
SERVICE_MIGRATION_TARGET_CATEGORY_MISSING
```

Do not show a generic `Migration action failed` when a specific safe reason is available.

## Reuse proven Customer/GiftCard migration patterns

Audit the previously working Customer and GiftCard migration implementation and reuse relevant proven patterns for:

```text
- independent preview/import command state;
- explicit selected file;
- Create/Update/Skip/Error row status;
- stale-preview invalidation;
- transaction boundary;
- idempotency;
- outbox/audit behavior;
- safe error display.
```

Do not create a second generic migration framework if the existing Customer/GiftCard components can be reused or extracted safely.

Report the exact files/patterns reused.

## Transaction boundaries

Required:

```text
Import Categories
-> one category-only transaction

Import Services
-> one service-only transaction
```

A Service transaction must contain zero category insert/update/delete operations.

A Category transaction must contain zero service insert/update/delete operations.

## Tests

Add focused tests for at least:

```text
Category path can be cleared independently
Service path can be cleared independently
Category path change invalidates only Category preview
Service path change invalidates only Service preview
Preview Categories reads only Category workbook
Preview Services reads Service workbook and target categories
Import Categories writes categories only
Import Services writes services only
Import Services with populated Category path performs zero category writes
Existing categories + Category preview -> Skip/Update counts, no unintended creates
Category Skip rerun -> no duplicate outbox/audit
Service preview works with Category path cleared when target categories resolve
Service preview reports precise error when category mapping is unavailable
Category import refreshes/invalidates Service preview
Service import does not re-import Categories
stale preview import is rejected
button labels and handlers are entity-specific
Customer/GiftCard proven pattern reuse or equivalence proof
```

Use EF InMemory/disposable harness only. Do not mutate `obm_pos_dev_v0_pg`.

## Physical retest safety

Do not launch WPF or import workbooks automatically.

The operator will physically test:

```text
1. Category already exists.
2. Clear Category path or leave it populated.
3. Preview Services only.
4. Confirm Service counts and category resolution.
5. Import Services only.
6. Verify Category row count/data/outbox did not change.
```

## Evidence folder

Create the next available versioned folder:

```text
E:\Project2026\RecoveryReports\ServiceCategoryMigration\IndependentLanesV001
```

Preserve:

```text
README.md
SHA256SUMS.txt
pre-change-combined-flow.mmd
post-change-independent-flow.mmd
command-scope-matrix.csv
state-invalidation-matrix.csv
customer-giftcard-pattern-reuse.md
safe-test-results.txt
```

No business row contents, secrets, or raw private identifiers.

## Documentation updates

Preserve current task/result under the next versioned history folder before updating.

Update current docs to state:

```text
- Category and Service migration are independent lanes;
- Category workbook path is not an implicit import selection;
- Service import never writes Categories;
- Category import never writes Services;
- next task is physical Service-only preview/import retest.
```

Canonical V002 remains unchanged.

## Required builds/tests

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~ServiceCategoryMigration|FullyQualifiedName~ServiceMigration|FullyQualifiedName~CategoryMigration|FullyQualifiedName~ExcelMigration|FullyQualifiedName~MigrationTab|FullyQualifiedName~MigrationPreview|FullyQualifiedName~IndependentMigration" -v minimal
```

## Prohibited actions

Do not:

```text
import into obm_pos_dev_v0_pg automatically
mutate source DB
launch WPF
hard-code tenant/category GUIDs
create categories during Service import
create services during Category import
silently infer import scope from non-empty paths
change PIN seed/Payroll blocker
commit/push OBM source
print secrets or business rows
```

## Report 065

Create and push:

```text
report/report065.md
```

Required sections:

1. Verdict.
2. DOCS_READ_BEFORE_CODE_GATE proof.
3. Physical ambiguous combined-lane evidence.
4. Exact pre-change combined preview/import behavior.
5. Customer/GiftCard pattern audit and reuse.
6. Final Category lane UI/commands/state.
7. Final Service lane UI/commands/state.
8. Explicit migration scope API.
9. Category-existing/idempotency behavior.
10. Service category-resolution behavior.
11. Transaction/entity-write isolation proof.
12. State invalidation/stale-preview proof.
13. Exact files changed.
14. Build/test counts.
15. Evidence folder/hashes.
16. No DB/import/process/source-push mutation proof.
17. Exact physical retest steps.
18. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_SERVICE_CATEGORY_INDEPENDENT_MIGRATION_LANES_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```

```text
BLOCKED_SERVICE_CATEGORY_SCOPE_ISOLATION
```

```text
BLOCKED_SERVICE_CATEGORY_INDEPENDENT_LANES_BUILD_OR_TEST
```
