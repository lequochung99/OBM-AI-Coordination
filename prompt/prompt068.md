# Prompt 068 — Make imported Services immediately visible by selecting a category with rows and adding safe all-category visibility

## Physical operator evidence

The operator physically opened `Category / Service Manager (New) -> Services`.

Observed:

```text
Show inactive services = checked
Selected category = an existing active category
GridServices = empty
```

Prompt067 proved:

```text
Target DB service rows = 158
Services are distributed across 14 categories
The currently selected category exists and is active
The currently selected category has 0 services
```

Therefore the current empty grid is not a DB/import/binding failure. It is a category-selection UX problem: the UI can select an empty category and gives no obvious indication that other categories contain services.

## Objective

Make imported Services immediately discoverable without changing DB data.

Required behavior:

```text
- category selector shows a safe service count per category;
- categories with 0 services are visually distinguishable;
- initial selection prefers the first category that has rows under the current inactive filter;
- after Service import/refresh, selection moves to a category containing imported Services when the prior selected category has 0 rows;
- user selection is otherwise preserved;
- optional `All Categories` view may be added if it fits the existing UI cleanly;
- no DB mutation;
- no service activation;
- no category remapping;
- no hidden API dependency.
```

## Mandatory documentation gate

Before source/test/doc edits, read:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report066.md
report/report067.md
```

Record:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V002
CanonicalDocSha256=<actual hash>
```

## Required audit

Inspect:

```text
CatalogServicesTab_UC
CategoryServiceManager_UC
CategoryServiceManagementLocalService
category combo item model/template
selected-category persistence
Show inactive services checkbox handling
post-import refresh handler
GridServices ItemsSource binding
```

Read-only DB evidence may be reused from report067; do not mutate DB.

Determine:

```text
- how initial category is selected;
- whether selection is first active category, previous category, or arbitrary order;
- whether service counts are already available;
- how includeInactive changes category counts;
- how refresh/import preserves or changes selection.
```

## Preferred UX

### Category selector label

Display a label shaped like:

```text
<Category Name> (<count>)
```

Examples are illustrative only; do not print private category names in reports.

Count must reflect the current `Show inactive services` state:

```text
unchecked -> active service count
checked   -> all service count
```

### Initial selection

On first load:

```text
1. preserve a valid previous selection when it has rows;
2. otherwise choose the first category with count > 0;
3. if all categories are empty, keep first category and show explicit empty-state text.
```

Do not auto-select by hard-coded category name or GUID.

### After import or refresh

When Services are imported/refreshed:

```text
- preserve selected category if it now has rows;
- if selected category has 0 rows and other categories have rows, move to the first category with rows;
- reload GridServices immediately;
- show status: SelectedCategoryResolved, IncludeInactive, QueryRows, DisplayedRows.
```

### Optional All Categories

Add `All Categories` only if it can be implemented by the existing local catalog service without creating a second cache/query framework.

If implemented:

```text
- it appears as an explicit synthetic selector item;
- it loads services for the active tenant across categories;
- sorting remains deterministic;
- edit/activate/deactivate actions still operate on the selected service;
- category identity remains visible in the grid;
- it does not create a fake DB category.
```

If not implemented, report why category counts + auto-selection are sufficient.

## Empty state

When selected category has no rows, do not leave a silent blank grid.

Show a safe message:

```text
No services in the selected category.
Choose a category with a non-zero service count.
```

Do not show modal dialogs.

## Tests

Add focused tests for:

```text
158 services across multiple categories -> category counts are correct
includeInactive=false -> active counts only
includeInactive=true -> all counts
initial empty selected category -> first non-empty category selected
valid selected category with rows -> selection preserved
post-import selected category empty -> moves to non-empty category
all categories empty -> safe empty state
category count labels update after checkbox toggle
Refresh reloads counts and grid
optional All Categories behavior when implemented
no DB mutation
no API dependency
```

## Evidence

Create:

```text
E:\Project2026\RecoveryReports\ServiceCategoryMigration\ServiceCategorySelectionUxV001
```

Never overwrite an existing version.

Expected artifacts:

```text
README.md
SHA256SUMS.txt
pre-change-selection-flow.mmd
post-change-selection-flow.mmd
category-count-contract.md
safe-test-results.txt
```

## Prohibited actions

Do not:

```text
mutate PostgreSQL
activate/deactivate services
change service-category relationships
re-import workbooks
launch WPF automatically
change API/PIN/DB credentials
commit/push OBM source
print category/service names or raw GUIDs in report
```

## Build/test

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~ServiceManager|FullyQualifiedName~ServiceLoad|FullyQualifiedName~ServiceRefresh|FullyQualifiedName~CategorySelection|FullyQualifiedName~InactiveFilter" -v minimal
```

## Report 068

Create and push:

```text
report/report068.md
```

Required sections:

1. Verdict.
2. DOCS_READ_BEFORE_CODE_GATE proof.
3. Physical empty-category evidence.
4. Exact initial-selection behavior before change.
5. Category-count query/model behavior.
6. Final selection rules.
7. Empty-state behavior.
8. Optional All Categories decision.
9. Post-import refresh behavior.
10. Exact files changed.
11. Build/test counts.
12. Evidence folder/hashes.
13. No DB/import/process/source-push mutation proof.
14. Operator physical retest steps.
15. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_SERVICE_CATEGORY_SELECTION_UX_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_SERVICE_CATEGORY_SELECTION_BUILD_OR_TEST
```

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```
