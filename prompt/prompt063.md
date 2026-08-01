# Prompt 063 — Export TblServiceCategory/TblService to Excel and add Service/Category Migration UI

## Operator priority

Defer prompt062 and the unrelated Payroll migration-test blocker.

Do not revert the prompt061 Canonical V002 or Phase 2 V003 PIN seed work. That work remains present but is not the priority of this task.

The operator now authorizes this higher-priority feature:

```text
1. Create two Excel workbooks under:
   E:\NailSallon_RS\MigrationDb\ServiceCategory

2. Export data from:
   TblServiceCategory -> CategoryMigration.xlsx
   TblService         -> ServiceMigration.xlsx

3. In the existing Service/Category management UI, add a Migration tab
   that previews and imports both workbooks into the current local tenant DB.
```

Keep this implementation practical. Do not create another platform service, remote API, migration server, or complex identity/authentication layer.

## Mandatory documentation gate

Before editing source, tests, documentation, project files, or creating Excel files, read completely:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report060.md
report/report061.md
```

Record before the first source edit:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V002
CanonicalDocSha256=<actual current hash>
```

Canonical V002 does not need a new architecture version for this feature. Preserve current task/result files under the next versioned history folder before updating them.

## Source and target database boundary

The intended migration flow is legacy/reference salon data -> current OBM-POS tenant database.

Audit the existing project configuration and prove the exact source connection before export.

Expected source candidate from the current project context:

```text
enailsalon_phasee1_pos1_pg
```

Expected current target runtime database:

```text
obm_pos_dev_v0_pg
```

Rules:

```text
- source export connection is read-only;
- do not export from the target DB by accident;
- do not mutate the source database;
- do not import into the target during this prompt automatically;
- UI import is left for operator physical testing;
- do not print passwords or full connection strings;
- report only sanitized database names, table counts, and equality booleans.
```

If project evidence shows a different canonical source DB, use the proven source and report the reason. Do not guess silently.

If more than one tenant exists in the source tables, resolve the intended salon through existing explicit tenant configuration/context. Do not export mixed-tenant data and do not hard-code a private TenantGuid in public artifacts.

## Phase 1 — audit existing models, CRUD paths, and Excel dependencies

Inspect exact source/schema/UI for:

```text
TblServiceCategory
TblService
service/category entities and mappings
service/category create/edit/delete UI
service/category repositories/services
TblLocalOutbox/event creation for manual CRUD
current active-tenant/TenantGuid resolution
existing Customer/GiftCard Excel migration implementations
existing Excel library/package such as ClosedXML, EPPlus, OpenXML, or equivalent
```

Report:

```text
- exact ServiceCategory primary/stable key;
- exact Service primary/stable key;
- exact Service -> ServiceCategory foreign key;
- TenantGuid columns and scope;
- soft-delete/active fields;
- display/order/color/price/duration/tax/bookable fields;
- audit/concurrency/generated fields;
- exact manual CRUD path and outbox behavior;
- exact existing management window/page/tab where Migration must be added.
```

Reuse the existing Excel package and migration patterns when available. Do not introduce a second spreadsheet stack without necessity.

## Phase 2 — create the migration folder and preserve existing files

Required folder:

```text
E:\NailSallon_RS\MigrationDb\ServiceCategory
```

Required current filenames:

```text
CategoryMigration.xlsx
ServiceMigration.xlsx
```

If either file already exists:

```text
- do not overwrite it silently;
- move/copy it into a versioned backup folder such as history\V001;
- use the next available version folder;
- record SHA-256 before replacement.
```

Do not create alternate current filenames such as `CategoryMigration_new.xlsx`.

## Phase 3 — Excel workbook contract

Each workbook must contain exactly these visible worksheets unless the existing Excel pattern requires a small justified variation:

```text
Data
Manifest
```

### Data sheet

Requirements:

```text
- row 1 contains exact source database column names;
- freeze the header row;
- enable filters;
- format the data region as an Excel table;
- preserve text leading zeros;
- use invariant/explicit formats for numbers, booleans, dates, times, money, and GUIDs;
- write strings as data, not formulas;
- protect against accidental formula execution for text beginning with =, +, -, or @;
- do not include credentials, tokens, or unrelated customer/employee data.
```

The export must include all source table columns needed to faithfully understand the source rows. Technical columns may remain in the workbook for evidence, but the importer must explicitly whitelist writable business fields and must not blindly write database-generated/audit/concurrency columns.

### Manifest sheet

Include sanitized metadata:

```text
WorkbookContractVersion
SourceDatabaseName
SourceTableName
ExportedUtc
SourceTenantSelectionMode
ExportedRowCount
ColumnCount
DataSheetName
PrimaryKeyColumn
CategoryForeignKeyColumn when applicable
SHA256 or deterministic content fingerprint when practical
```

Do not include raw connection strings or private credentials.

### Category workbook

```text
CategoryMigration.xlsx
Data source: TblServiceCategory
```

Must preserve the stable category key required by `TblService` relationships.

### Service workbook

```text
ServiceMigration.xlsx
Data source: TblService
```

Must preserve the service stable key and exact category foreign key.

## Phase 4 — perform the read-only export

Use the canonical source database connection in a read-only transaction/session.

Export:

```text
TblServiceCategory -> CategoryMigration.xlsx
TblService         -> ServiceMigration.xlsx
```

Required proof:

```text
- exported category row count equals sanitized source query count;
- exported service row count equals sanitized source query count;
- every non-null service category foreign key exists in CategoryMigration.xlsx;
- workbook can be reopened and parsed by the selected Excel library;
- Manifest counts match Data counts;
- no source DB mutation;
- no target DB mutation.
```

Do not print service/category business names in the public report. Counts and schema/column names are sufficient.

## Phase 5 — Migration tab in the existing management UI

Locate the existing management surface that creates/edits Service and Service Category.

Add one sibling tab:

```text
Migration
```

Do not create a separate top-level application unless the current UI architecture makes a tab impossible.

### Migration tab layout

Provide two clear sections.

#### Category Migration

```text
File path textbox
Browse button
Default path: E:\NailSallon_RS\MigrationDb\ServiceCategory\CategoryMigration.xlsx
Preview button
Import Categories button
Preview/result grid
Summary counts
```

#### Service Migration

```text
File path textbox
Browse button
Default path: E:\NailSallon_RS\MigrationDb\ServiceCategory\ServiceMigration.xlsx
Preview button
Import Services button
Preview/result grid
Summary counts
```

Status values:

```text
Create
Update
Skip
Error
```

Summary:

```text
Total
Create
Update
Skip
Error
Imported
```

The UI must remain touch-friendly and must not hide action buttons inside a scrolling area where touch selection can clear state.

## Phase 6 — import order and relationship rules

Canonical order:

```text
1. Preview/import categories.
2. Preview/import services.
```

Service import must block rows whose category key cannot resolve in the current tenant.

Do not auto-create an arbitrary category and do not silently assign services to the first category.

The workbook `TenantGuid` is source evidence only. Import always targets the currently active local tenant resolved by the application.

Do not allow a workbook to switch the target tenant.

## Phase 7 — matching and idempotency

Use the audited stable GUID/key strategy.

Preferred bounded behavior for same-salon migration:

```text
- preserve source CategoryGuid/ServiceGuid when there is no collision;
- match existing target row by current TenantGuid + stable source key/GUID;
- existing identical row -> Skip;
- existing row with supported changed business fields -> Update;
- missing row -> Create;
- key collision with another tenant or incompatible row -> Error during Preview;
- do not silently generate a replacement GUID when that would break relationships;
- do not create duplicates on repeated import;
- do not delete target rows missing from the workbook.
```

If existing source models use another canonical key/mapping contract, follow it and document the exact reason.

## Phase 8 — writable fields and validation

Audit the current manual CRUD rules and reuse them.

Importer must validate at least:

```text
required names/descriptions
GUID/key shape
Tenant scope
category relationship
price/rate ranges
duration ranges
sort/display order
active/bookable/tax flags
nullable fields
string lengths
duplicate stable keys
duplicate business names when the application treats them as unique
```

Whitelist supported writable fields.

Do not write:

```text
database-generated identity values when not part of the stable key
rowversion/concurrency tokens
created/modified audit timestamps unless the canonical model explicitly requires source preservation
source TenantGuid into a different target tenant
unrecognized workbook columns
```

Unknown columns may be shown as warnings but must not be written.

## Phase 9 — transaction and outbox behavior

Each confirmed import runs transactionally per workbook:

```text
Category import transaction
Service import transaction
```

If any blocking row error remains at confirmation time:

```text
- do not partially commit;
- keep the preview visible;
- show row-specific safe errors.
```

Reuse the same domain validation, repository, and `TblLocalOutbox`/event composition used by manual Service/Category CRUD so migrated changes can sync correctly.

Required idempotency:

```text
first import -> intended Create/Update rows + corresponding outbox rows
same workbook rerun -> all unchanged rows Skip
same workbook rerun -> no duplicate rows
same workbook rerun -> no duplicate outbox events for skipped rows
```

Do not add a new migration database table unless an existing generic migration-audit framework already requires it. Prefer the existing migration pattern and stable keys.

## Phase 10 — preview safety

Preview must not mutate PostgreSQL.

Preview performs:

```text
file/schema validation
manifest validation
row parsing
business validation
target lookup
Create/Update/Skip/Error classification
relationship resolution
summary counts
```

Import buttons remain disabled until the corresponding preview is current and contains zero blocking errors.

Changing the file path invalidates the preview.

## Phase 11 — tests

Add focused tests for at least:

```text
Category export source count == workbook data count
Service export source count == workbook data count
Manifest counts == Data counts
workbooks reopen successfully
all service category FKs resolve in exported category workbook
formula-like text is exported safely
leading zeros/text values are preserved
category preview Create/Update/Skip/Error
service preview Create/Update/Skip/Error
source TenantGuid cannot change target tenant
category import idempotency
service import idempotency
service missing category -> Error and no commit
key collision -> Error and no commit
unknown columns are not written
technical/audit columns are not blindly written
transaction rollback on blocking error
manual CRUD validation rules are reused
outbox emitted for Create/Update only
no duplicate outbox on identical rerun
Migration tab exists beside Service/Category management tabs
file path change invalidates preview
import disabled until successful preview
prompt063/current documentation gate
```

Use disposable/in-memory/test databases only. Do not point tests at `obm_pos_dev_v0_pg` or the source reference database except for the explicitly read-only export operation.

Do not use the broad prompt061 test filter that pulls in the unrelated Payroll migration artifact. Run the focused feature tests plus the normal WPF build.

## Phase 12 — evidence and documentation

Create the next available evidence folder:

```text
E:\Project2026\RecoveryReports\ServiceCategoryMigration\V001
```

Never overwrite a prior version.

Expected artifacts:

```text
README.md
SHA256SUMS.txt
source-schema-inventory.md
workbook-contract.md
export-counts.json
category-import-mapping.csv
service-import-mapping.csv
ui-flow.mmd
safe-test-results.txt
```

Do not copy raw service/category business rows into the evidence folder.

Preserve current `CURRENT_TASK.md` and `CURRENT_RESULT.md` under the next history version, then update current docs:

```text
- prompt062/Payroll test blocker deferred;
- CategoryMigration.xlsx and ServiceMigration.xlsx created;
- source DB/table counts exported;
- Migration tab implemented;
- physical import into current DB not yet performed;
- next task is operator preview/import test with a backup/disposable target first.
```

Canonical remains V002.

## Build label

This feature does not require changing the InstallationV0 build label unless InstallationV0 source is modified. Report the decision.

## Required builds/tests

Run sequentially:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~ServiceCategory|FullyQualifiedName~ServiceMigration|FullyQualifiedName~CategoryMigration|FullyQualifiedName~ExcelMigration|FullyQualifiedName~MigrationTab" -v minimal
```

If the feature spans another existing focused test project, run that project too and report it.

## Physical execution policy

Allowed during this prompt:

```text
- create/backup the two Excel files;
- read source TblServiceCategory/TblService;
- build/test source;
- create local evidence.
```

Not allowed during this prompt:

```text
- import the workbooks into obm_pos_dev_v0_pg automatically;
- mutate the source DB;
- run WPF automatically;
- click the Migration UI;
- change current employee PINs;
- resolve the unrelated Payroll artifact by inventing SQL;
- commit/push OBM source;
- print credentials or business row contents.
```

## Report 063

Create and push:

```text
report/report063.md
```

Required sections:

1. Verdict.
2. DOCS_READ_BEFORE_CODE_GATE proof.
3. Deferred prompt062/Payroll blocker statement.
4. Exact source and target DB boundary proof.
5. Source schema/model/CRUD/outbox audit.
6. Exact Excel library reused.
7. Folder/files created and prior-file backup/versioning.
8. Category workbook contract, columns, row count, hash.
9. Service workbook contract, columns, row count, hash.
10. Service-category relationship validation.
11. Exact management UI location and Migration tab implementation.
12. Preview/Create/Update/Skip/Error behavior.
13. Tenant and stable-key matching behavior.
14. Transaction/idempotency/outbox behavior.
15. Exact source/docs/tests files changed.
16. Build/test commands and counts.
17. Evidence folder and hashes.
18. No source/target DB mutation beyond read-only export proof.
19. Exact operator physical preview/import steps.
20. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_SERVICE_CATEGORY_EXCEL_MIGRATION_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_SOURCE_DATABASE_OR_TENANT_AMBIGUOUS
```

```text
BLOCKED_SERVICE_CATEGORY_SCHEMA_OR_RELATIONSHIP_AMBIGUOUS
```

```text
BLOCKED_SERVICE_CATEGORY_MIGRATION_BUILD_OR_TEST
```
