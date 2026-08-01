# Prompt 066 — Verify whether Services were imported, then repair the Service UI load/refresh path if needed

## Operator request

After the Service/Category migration lanes were separated, verify the real current state in this exact order:

```text
1. Check whether Services were actually added to the current local database.
2. If Services exist in the database, trace why they are not visible in the Service management UI.
3. If Services do not exist, identify whether Import Services did not run, rolled back, was blocked, or wrote to the wrong tenant/database.
```

Do not assume the UI reflects database state.

## Mandatory documentation gate

Before editing source, tests, or current documentation, read completely:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report063.md
report/report064.md
report/report065.md
```

Record before the first edit:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V002
CanonicalDocSha256=<actual current hash>
```

## Current expected environment

Target runtime database:

```text
obm_pos_dev_v0_pg
```

Runtime PostgreSQL user:

```text
hung
```

Workbook:

```text
E:\NailSallon_RS\MigrationDb\ServiceCategory\ServiceMigration.xlsx
```

Expected source export count:

```text
158 Services
```

Do not print passwords, full connection strings, private GUIDs, or business row contents.

## Phase 1 — read-only physical database verification

Use the approved protected local credential path and query only inside:

```sql
BEGIN TRANSACTION READ ONLY;
...
ROLLBACK;
```

Determine safely:

```text
active TenantGuid present = true/false
TblService total rows for active tenant
TblService rows matching workbook source GUIDs/count
TblService rows grouped by active/inactive/deleted/status flags
TblService rows with resolvable TblServiceCategory relation
orphan service-category count
duplicate service GUID count
duplicate normalized service-name count, informational only
latest relevant TblLocalOutbox count by entity/action, sanitized
```

Do not print Service names or raw GUIDs in the report.

Read the workbook and compare by stable source keys/count only:

```text
workbook Service rows = 158
matching target rows = N
missing target rows = N
differing target rows = N
```

Choose exactly one primary physical classification:

```text
SERVICES_IMPORTED_COMPLETE
SERVICES_IMPORTED_PARTIAL
SERVICES_NOT_IMPORTED
SERVICES_IMPORTED_WRONG_TENANT_OR_DATABASE
SERVICES_PRESENT_BUT_FILTERED_BY_STATUS
OTHER_PROVEN_DATABASE_STATE
```

## Phase 2A — when Services are present in DB

Trace the complete UI load path for the existing Service management tab:

```text
CategoryServiceManager_UC / Services_aMain_UC entry
-> Service tab Selected/Loaded event
-> ViewModel/control initialization
-> local catalog/query service
-> EF query/filter
-> collection assignment
-> ItemsSource/DataGrid/list binding
-> refresh after migration import
```

Inventory every filter that can exclude imported Services:

```text
TenantGuid
ServiceCategoryGuid
IsActive
IsDeleted
Status
Enable/Visible flag
ServiceType
Category active state
search text
selected category
cached collection
one-time Loaded guard
stale DbContext tracking
```

Report exact source file/method/line and actual pre-change query predicate.

### Required UI behavior

When the database contains valid Services for the active tenant:

```text
- opening the Services tab loads them;
- switching away and back reloads or uses a valid refreshed cache;
- successful Import Services refreshes the Services list automatically;
- successful Import Services refreshes Category counts/grouping when needed;
- no app restart is required;
- no API connection is required;
- API HTTP 401 must not block local Service display.
```

Prefer reusing the existing Service-management reload method. Do not create a second Service catalog/cache.

If migration currently updates DB but does not notify the UI, add one explicit post-import refresh/event using the existing local UI architecture. Do not add a remote event bus or platform dependency.

## Phase 2B — when Services are not present in DB

Trace the Service import path:

```text
ServiceCategoryMigration_UC.ImportServices
-> IServiceCategoryMigrationService.ImportServicesAsync
-> preview fingerprint/stale-state validation
-> target category resolution
-> create/update/skip/error loop
-> transaction
-> existing Service CRUD/local service
-> SaveChanges
-> TblLocalOutbox/audit
-> commit/result
```

Determine exactly whether the physical import:

```text
was never clicked
was disabled
was blocked by preview errors
returned success but rolled back
wrote to another database
wrote to another tenant
processed all rows as Skip/Error
failed before commit
```

Do not import automatically in this prompt.

Fix source/tests only when a proven defect exists. Leave the next physical Import Services action to the operator.

## Phase 3 — import result and UI refresh contract

Ensure `ImportServicesAsync` returns structured, sanitized fields such as:

```text
Succeeded
StageId
ResultCode
CreateCount
UpdateCount
SkipCount
ErrorCount
Committed
TargetTenantResolved
ServiceRowsAfterCommitCount, optional safe count
RequiresUiRefresh
```

The UI must display a clear result and refresh the existing Services view only when `Committed=true`.

Do not expose raw row values in public logs/reports.

## Phase 4 — tests

Add focused tests for the proven classification and at least:

```text
Services exist in target DB -> Service UI load query returns them
active tenant filter correct
valid active imported Services not excluded by legacy status predicate
inactive/deleted Services follow existing intended UI policy
successful Import Services triggers existing Service list refresh
failed/rolled-back import does not show false success or refresh as committed
Service import writes zero Categories
Service display works while API is unavailable/401
switching to Services tab after import shows refreshed collection
no duplicate UI catalog/cache introduced
```

If Services are not yet imported, add tests proving the import transaction and result contract, but do not mutate the operator DB.

## Phase 5 — evidence

Create the next available folder:

```text
E:\Project2026\RecoveryReports\ServiceCategoryMigration\ServiceDbUiVerificationV001
```

Never overwrite an existing version.

Expected artifacts:

```text
README.md
SHA256SUMS.txt
physical-db-service-counts.json
workbook-vs-db-counts.json
service-ui-load-path.mmd
service-ui-filter-inventory.csv
import-to-ui-refresh-flow.mmd
safe-test-results.txt
```

No Service names, raw GUIDs, credentials, or business rows.

## Safety

Do not:

```text
mutate source/reference DB
mutate obm_pos_dev_v0_pg during investigation
run workbook import automatically
launch/click WPF automatically
change API/PIN/DB credentials
create another migration framework
commit/push OBM source
print business row contents
```

Read-only DB inspection, source fixes, tests, builds, docs, and evidence are allowed.

## Required build/tests

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~ServiceCategoryMigration|FullyQualifiedName~ServiceMigration|FullyQualifiedName~ServiceManager|FullyQualifiedName~ServiceLoad|FullyQualifiedName~ServiceRefresh|FullyQualifiedName~IndependentMigration" -v minimal
```

## Documentation

Preserve current task/result files under the next versioned history folder before updating them.

Update `CURRENT_RESULT.md` with the physical DB classification and source correction, if any.

Update `CURRENT_TASK.md` to the exact next operator action:

```text
- if DB complete: physical UI reload verification;
- if DB empty/partial: Preview Services then Import Services, followed by UI refresh verification.
```

Do not change canonical V002 architecture.

## Report 066

Create and push:

```text
report/report066.md
```

Required sections:

1. Verdict.
2. DOCS_READ_BEFORE_CODE_GATE proof.
3. Read-only physical DB classification.
4. Workbook-versus-target safe counts.
5. Exact Service UI load path.
6. Exact query/filter predicates.
7. Import path classification when applicable.
8. Root cause.
9. Source correction and UI refresh behavior.
10. Structured import/result contract.
11. Exact files changed.
12. Build/test counts.
13. Evidence folder/hashes.
14. No DB/import/process/source-push mutation proof.
15. Exact operator retest steps.
16. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_SERVICES_PRESENT_UI_RELOAD_READY_FOR_PHYSICAL_RETEST
```

```text
OBM_POS_SERVICES_NOT_IMPORTED_IMPORT_FLOW_READY_FOR_PHYSICAL_RETEST
```

```text
OBM_POS_SERVICES_PARTIAL_IMPORT_REVIEW_REQUIRED
```

```text
BLOCKED_SERVICE_TARGET_DATABASE_OR_TENANT_AMBIGUOUS
```

```text
BLOCKED_SERVICE_DB_UI_VERIFICATION_BUILD_OR_TEST
```
