# Prompt 069 — Persist Service display order per Category and apply it on MainWindow

## Operator decision

The Service management screen already exposes three display-order modes:

```text
Price Low -> High
Price High -> Low
Custom Order
```

The operator requires these controls to become the canonical persisted Service ordering for the selected Category and to control the Service order shown on the real MainWindow.

This must not remain a temporary sort applied only inside the management grid.

The current UI label `Customer Order`, if present anywhere, is a typo and must be corrected to:

```text
Custom Order
```

## Required behavior

For each Service Category independently:

```text
Price Low -> High
-> MainWindow Services in that Category are sorted by price ascending

Price High -> Low
-> MainWindow Services in that Category are sorted by price descending

Custom Order
-> MainWindow Services in that Category follow the persisted manual order
-> Move Up / Move Down changes that persisted order
```

The mode and manual order must survive:

```text
closing/reopening the management screen
closing/restarting WPF
API unavailable or HTTP 401
another local POS loading the same synchronized tenant data, when the existing sync contract supports it
```

## Mandatory documentation gate

Before source, schema, tests, or documentation edits, read completely:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report065.md
report/report066.md
report/report067.md
```

Also read `report/report068.md` if it exists by execution time. If prompt068 has not yet run, inspect the current source directly and do not assume its changes.

Record before the first implementation edit:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V002
CanonicalDocSha256=<actual current hash>
```

Canonical architecture does not need a new version unless this task proves a contract change beyond local catalog ordering.

## Scope boundary

This task is about Service display order only.

Do not:

```text
change Service prices
activate/deactivate Services in bulk
rerun Service import
change Category/Service GUIDs
create a new migration framework
create a remote ordering service
require API availability for local ordering
change employee PIN/authentication behavior
mutate production/reference databases automatically
commit/push OBM source
```

Source/schema/tests/docs changes needed for ordering are allowed. Physical DB migration/application remains operator-controlled unless an existing normal app migration path safely applies it during startup.

## Phase 1 — audit current ordering model before changing anything

Inspect exact source/schema and report all existing ordering fields and callers.

Search at minimum:

```text
TblService
TblServiceCategory
DisplayOrder
SortOrder
OrderIndex
ItemOrder
Sequence
Position
Price
MoveUp
MoveDown
Price Low
Price High
Custom Order
Customer Order
CatalogServicesTab_UC
CategoryServiceManagementLocalService
MainWindow
service button/menu creation
service category selection
```

Inventory:

```text
existing Service order field(s)
existing Category sort-mode field(s)
current radio-button event handlers
current Move Up / Move Down implementation
current management-grid ordering
current MainWindow Service query/order path
current API/outbox/sync mapping for any relevant fields
```

Do not add duplicate fields when an existing canonical field already serves the purpose.

## Phase 2 — canonical persistence model

### A. Sort mode belongs to the Category

The selected ordering mode is category-specific.

Required conceptual values:

```text
PriceAscending
PriceDescending
Custom
```

Prefer a strongly typed enum in source.

Reuse an existing `TblServiceCategory` field when source proves one exists. If no persistent field exists, add one minimal field to the canonical Category model/schema, with a backward-compatible default of `Custom` unless existing production behavior clearly proves another default.

If a new schema field is required:

```text
- add a forward WPF migration;
- update the API model/migration only when the existing Category sync contract requires schema parity;
- update existing DTO/mapping/outbox serialization through the current CRUD path;
- do not introduce a second settings table for one enum;
- do not use environment variables or local JSON as the category sort authority.
```

### B. Manual order belongs to the Service row

Reuse the existing Service ordering field if present.

If no field exists, add one minimal integer field to the canonical Service model/schema. The order is scoped by:

```text
TenantGuid + ServiceCategoryGuid
```

Requirements:

```text
- deterministic integer sequence;
- no duplicate effective positions after a reorder transaction;
- new Service receives a stable position at the end of its Category;
- moving a Service to another Category assigns it to the end of the target Category;
- deleting/deactivating a Service does not corrupt ordering;
- import preserves a valid source order field when the workbook contract includes it, otherwise appends deterministically;
- no GUID hard-coding.
```

## Phase 3 — deterministic ordering rules

Implement one shared ordering policy used by both the management screen and MainWindow.

### Price Low -> High

Conceptual order:

```text
Price ascending
then persisted custom order ascending
then normalized ServiceName ascending
then ServiceGuid ascending
```

### Price High -> Low

Conceptual order:

```text
Price descending
then persisted custom order ascending
then normalized ServiceName ascending
then ServiceGuid ascending
```

### Custom Order

Conceptual order:

```text
persisted custom order ascending
then normalized ServiceName ascending
then ServiceGuid ascending
```

Use the actual canonical price field proven by source. Handle nullable prices deterministically; null price values should sort last in both price modes unless existing business rules prove another behavior.

Do not duplicate sorting logic in multiple UI code-behind files. Use one focused policy/helper/query composition path.

## Phase 4 — Service management UI behavior

The existing controls must work as follows.

### Radio modes

When the operator changes the mode for the selected Category:

```text
- persist the Category sort mode through the existing Category CRUD/update/outbox path;
- refresh the management list immediately;
- do not modify Service prices;
- do not rewrite manual order values merely because a price mode is selected;
- show a safe success/failure status;
- preserve the selected Category.
```

Do not silently persist during initial control binding/loading. Guard against radio-button initialization events triggering unintended updates.

### Move Up / Move Down

```text
- enabled only in Custom Order mode;
- disabled in price modes;
- requires one selected Service;
- swaps/resequences only within the selected Category;
- persists atomically through the existing Service update/outbox path;
- refreshes the management grid immediately;
- does not require WPF restart or API connectivity.
```

Audit the `Show inactive services` interaction. Preserve the existing operator expectation, but the stored canonical order must remain valid across all Services in the Category. Do not create duplicate positions because an inactive row is hidden.

If the current movement behavior cannot safely reorder hidden inactive rows, choose and document one simple behavior:

```text
- either reorder against the full Category sequence while showing the visible result;
- or require Show inactive services to be enabled for complete manual reordering.
```

Do not leave ambiguous behavior.

### Current-category status

Show safe status such as:

```text
SelectedCategoryResolved
SortMode
QueryRows
DisplayedRows
OrderSaved
ResultCode
```

Do not print private GUIDs or service names into public reports.

## Phase 5 — MainWindow ordering

Find the exact real MainWindow path that loads Services for a selected Category.

Replace any independent/default ordering with the shared canonical policy:

```text
load selected Category
read its persisted sort mode
load eligible Services for that Category
apply shared ordering policy
bind/render buttons/items in that order
```

Requirements:

```text
- local PostgreSQL is authoritative;
- no API prerequisite;
- active/visibility business filters remain unchanged except ordering;
- switching Category applies that Category's own mode;
- returning to a Category preserves its persisted mode/order;
- a management save triggers the existing local refresh path when practical;
- restart still shows the same order.
```

Audit other MainWindow caches/collections. Do not create a second Service cache.

## Phase 6 — synchronization/outbox integrity

Use existing Category and Service CRUD/outbox contracts.

Required proof:

```text
sort-mode change -> one intended Category update/outbox event
custom reorder -> intended Service update/outbox events only
no event for a no-op selection
no duplicate outbox on refresh/reload
price mode changes do not rewrite every Service row
```

If multi-machine sync consumers already apply Category/Service updates, include the new fields in those existing mappings. Do not build a new sync lane.

## Phase 7 — migration workbook interaction

Audit `CategoryMigration.xlsx` and `ServiceMigration.xlsx` mappings only for relevant ordering columns.

Rules:

```text
- importing Services must not reset an already valid target custom order unless the workbook contains an explicit valid order value and the import action is Update/Create according to current policy;
- unknown/blank ordering values use deterministic target defaults;
- Category import may preserve a valid sort mode only when the workbook contract explicitly contains it;
- no change to independent Category/Service migration lane isolation.
```

Do not run a physical workbook import in this task.

## Phase 8 — tests

Add focused tests covering at least:

```text
Category A PriceAscending -> management list ascending by price
Category A PriceDescending -> management list descending by price
Category A Custom -> persisted manual order
Category B has an independent mode/order
price ties use deterministic tie-breakers
nullable price sorts last
radio initialization does not persist accidentally
mode change persists once and refreshes list
Move Up swaps/resequences within Category
Move Down swaps/resequences within Category
Move buttons disabled outside Custom mode
inactive hidden rows do not produce duplicate order positions
new Service appends to end of Category
Service moved to another Category appends to target Category
MainWindow uses the same shared ordering policy
MainWindow restart/reload preserves mode/order
API unavailable does not block local ordering
no-op mode save creates no duplicate outbox
price mode change does not update all Service rows
migration preview/import mapping preserves valid order/defaults safely
`Customer Order` typo has zero active UI/source matches
```

Use disposable/in-memory test infrastructure. Do not mutate `obm_pos_dev_v0_pg` automatically.

## Phase 9 — evidence

Create the next available folder:

```text
E:\Project2026\RecoveryReports\ServiceCategoryMigration\ServiceDisplayOrderingV001
```

Never overwrite an existing version.

Expected evidence:

```text
README.md
SHA256SUMS.txt
ordering-field-inventory.csv
management-order-flow.mmd
mainwindow-order-flow.mmd
sort-mode-matrix.csv
outbox-write-matrix.csv
test-results.txt
```

No service names, private GUIDs, prices tied to named services, credentials, or business row contents in public evidence.

## Documentation updates

Canonical V002 architecture remains unchanged unless a genuine contract change is proven.

Preserve current `CURRENT_TASK.md` and `CURRENT_RESULT.md` under the next versioned history folder before updating them.

Update current docs with:

```text
- Service sort mode is persisted per Category;
- custom order is persisted per Service within Category;
- management and MainWindow share one ordering policy;
- next task is operator physical ordering retest.
```

## Required build/tests

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~ServiceOrder|FullyQualifiedName~ServiceSort|FullyQualifiedName~CatalogServices|FullyQualifiedName~MainWindow|FullyQualifiedName~ServiceCategory|FullyQualifiedName~Migration" -v minimal
```

If schema/model parity requires ApiServer changes, also run the focused ApiServer build/tests for Category/Service mapping. Do not broaden into unrelated failing Payroll tests.

## Physical execution policy

Do not launch WPF automatically and do not mutate the operator DB automatically.

Operator physical retest after PASS:

```text
1. Select one Category with multiple Services.
2. Choose Price Low -> High; verify management order.
3. Open MainWindow; verify the same ascending order.
4. Choose Price High -> Low; verify both screens.
5. Choose Custom Order; Move Up/Down; verify both screens.
6. Close/restart WPF; verify persisted mode/order.
7. Switch to another Category and verify independent order.
8. Keep API unavailable/401 and verify local ordering still works.
```

## Report 069

Create and push:

```text
report/report069.md
```

Required sections:

1. Verdict.
2. DOCS_READ_BEFORE_CODE_GATE proof.
3. Existing schema/order-field audit.
4. Final persistent model and migration decision.
5. Sort-mode enum/value contract.
6. Shared deterministic ordering policy.
7. Management radio-button behavior.
8. Custom Move Up/Down transaction behavior.
9. Inactive-row ordering policy.
10. MainWindow load/order path.
11. Sync/outbox proof.
12. Migration-workbook ordering behavior.
13. Exact files/migrations changed.
14. Build/test counts.
15. Evidence folder/hashes.
16. No DB/process/source-push mutation proof.
17. Exact operator physical retest steps.
18. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_SERVICE_DISPLAY_ORDERING_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```

```text
BLOCKED_SERVICE_ORDER_SCHEMA_CONTRACT_AMBIGUOUS
```

```text
BLOCKED_SERVICE_DISPLAY_ORDERING_BUILD_OR_TEST
```
