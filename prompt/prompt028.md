# Prompt 028 — Build the physical Phase 2 seed from the complete reference POS1 database

## Operator direction

The operator confirms that this database is the most complete and useful reference source:

```text
enailsalon_phasee1_pos1_pg
```

Prompt027 is superseded as an execution instruction.

Do not continue building the physical seed from guessed row counts or incomplete hard-coded placeholders. Use the reference database to determine the actual default/config rows and column values, while preserving the existing proven WPF save/outbox logic and consolidating the selected seed into one PostgreSQL transaction.

This does **not** authorize cloning the entire database. The reference database contains real salon/runtime/business data. It is a read-only template source for approved baseline/default/config rows only.

## Authoritative state

Read completely:

```text
report/report014.md
report/report023.md
report/report024.md
report/report025.md
report/report026.md
prompt/prompt023.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
E:\Project2026\RecoveryReports\InstallationV0\Phase2SeedAuditV003\
```

Current physical state:

```text
Reference DB: enailsalon_phasee1_pos1_pg — read-only source
Target DB: obm_pos_dev_v0_pg — only approved mutation target
Rollback anchor: Phase2Pos1TrialV007 — valid
Runtime role hung: dbo DML/sequence privileges PASS
Marker table: dbo."Phase2TrialCompletionMarker" exists, row count 0
Focused InstallationV0 tests: 37/37 PASS
Physical seed: never executed
```

## Core implementation model

Use this model:

```text
read exact selected default/config rows from reference DB
-> classify each column
-> transform Tenant/POS/row identity
-> reuse proven entity/save/outbox mapping
-> order parent before child
-> execute all selected data + outbox + marker in one target transaction
-> verify
-> commit once
```

Do not recreate a fragmented multi-transaction seed system.
Do not replace proven save/outbox code with a speculative framework when existing mapping can be adapted.

## Reference DB access — read only

Reference database:

```text
enailsalon_phasee1_pos1_pg
```

Use only the approved read-only credential path already used by prompt014, if still present:

```text
LOCALAPPDATA\OBM\Phase2SeedAudit\pgpass.conf
```

Allowed user:

```text
hung
```

Every reference query must use:

```text
PGOPTIONS=-c default_transaction_read_only=on
psql -X -v ON_ERROR_STOP=1
BEGIN TRANSACTION READ ONLY
ROLLBACK
```

Required proof:

```text
transaction_read_only = on
current_database = enailsalon_phasee1_pos1_pg
current_user = hung
```

Do not mutate the reference DB, call side-effect functions, change sequence values, change privileges, or export private business rows.

If the old reference pgpass path is missing or unusable, stop with:

```text
BLOCKED_PHASE2_REFERENCE_READ_ACCESS
```

Do not fallback to administrator credentials for the reference DB.

## Approved target boundary

Only mutate:

```text
obm_pos_dev_v0_pg
```

Hard reject:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
recovery_api_day16_pg
any other database
production/reference/protected database
```

Use role `hung` for Phase 2 data seed/runtime.
Admin credential is not used for seed.

Before mutation verify the existing valid rollback anchor:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV007\PreSeedBackup\
```

Verify its dump, pg_restore list, and SHA256SUMS before seed.

## Selected baseline/default tables

Use the reference DB to obtain the complete approved row set for these tables:

```text
TblSetupLoginMethod
TblSetupPaymentMethod
TblEmployeePermission
TblSetupWeird
TblSetupServicesMethod
TblParameterSetting
TblSetupPrinter
```

Create/verify local identity rows from Phase 1, not by copying reference identity:

```text
TblTenant
TblPosLocal
```

Completion marker:

```text
dbo."Phase2TrialCompletionMarker"
```

### Roles

From `TblEmployeePermission`, include only the operator-approved default permission rows:

```text
Owner
Admin
Sub_Manager
```

Use the exact non-secret values/flags/required columns from the matching reference rows.
Do not seed employee rows or PINs.

### Printer settings

Copy the complete canonical `TblSetupPrinter` logical configuration row set from the reference DB, including all logical types that exist there, expected to include:

```text
CUSTOMER_TERMINAL
MERCHANT_TERMINAL
POS_CUSTOMER
POS_MERCHANT
POS_EMPLOYEE
```

Preserve the logical flags and values used by the existing foreach/static-property loading code.

Do not modify that runtime logic.

For each printer column classify:

```text
preserve logical default
replace TenantGuid
replace PosGuid if present
generate/remap row GUID
remap FK
clear machine-private binding
derive timestamp
```

Clear only fields proven to be machine-specific/private for a new machine, such as physical Windows printer binding, path, IP, driver, share, machine name, or terminal credential. Do not clear logical print-control flags.

### Parameters

Use the complete **safe baseline/default subset** from `TblParameterSetting` in the reference DB.

Do not force exactly 2 or 6 rows.
Do not copy all 110 rows blindly.

For every reference parameter row classify:

```text
A. safe default required for startup/common behavior — include
B. safe optional feature default — include only when clearly canonical
C. salon historical/custom value — exclude
D. environment/machine-specific value — clear/derive/exclude
E. secret/private/gateway/terminal credential — exclude
F. duplicate/ambiguous scope — resolve by exact scope or exclude
```

At minimum preserve proven safe reads such as:

```text
Date Hien Tai
EnableTurnEngine
```

The final parameter count is determined by the sanitized reference-driven manifest, not by an arbitrary constant.

Public report may list safe key names and non-sensitive defaults, but never private URLs, credentials, tokens, terminal identifiers, passwords, or salon-private values.

## Explicit no-seed tables

Do not seed or copy rows from:

```text
TblEmployee and employee private/PIN rows
TblServiceCategory
TblService
TblProduct
TblCustomer*
TblGiftCard*
TblInvoice*
TblOutputInfo*
TblOutputInfoTam*
TblTerminalDejavoo
TblTerminalPaymentInfo
TblInvoiceBookingLink
booking/appointment rows
payment transaction/history
queue/turn/payroll runtime history
event log/delivery operational rows
historical TblLocalOutbox rows
```

These tables are schema-only for the initial installation. Runtime/business tables must remain unchanged/empty according to the Development target state.

The category-before-service example remains the dependency rule for future catalog import, not permission to seed catalog now.

## Dependency-first mapping

Build the actual physical/logical order using:

1. reference PostgreSQL FK metadata;
2. target schema FK metadata;
3. EF model configuration;
4. proven legacy seed call order.

For every selected table report:

```text
PK
FK parent(s)
logical parent dependency
Tenant/POS scope
stable key
reference row count selected
insert/adopt order
outbox rule
```

Parent rows must be established before child rows.

Expected high-level order:

```text
Phase 1 identity revalidation
TblTenant
TblPosLocal
independent lookup/default tables
Tenant-scoped singleton/config tables
matching TblLocalOutbox rows
completion marker last
```

Follow actual metadata if more specific ordering is required.

## Column transformation contract

Do not merely replace `TenantGuid` and copy everything else.

Every selected reference column must be classified as exactly one:

```text
Preserve canonical logical value
Replace from Phase 1 identity
Generate deterministic row GUID
Remap FK to deterministic/new parent GUID
Clear private/environment value
Derive at execution time
Exclude column/row from manifest
```

Mandatory rules:

```text
TenantGuid <- Phase 1 TenantGuid
PosGuid/POS name/slot <- Phase 1 POS identity where applicable
row GUID <- deterministic from version + target identity + table + stable key
FK GUID <- remapped to selected target parent row
Created/Updated timestamps <- execution time or approved canonical behavior
reference Tenant/POS/row GUIDs <- never copied as target identities
```

## Reuse proven save/outbox path

Reuse the proven concepts/mapping in:

```text
SeedDbProvider.RunLegacyDemoSeedAllAsync
MainServices.ExecuteInTransactionAsync
CreateLocalOutboxSingle / CreateLocalOutboxSingleAsync
```

Reuse entity construction, required field mapping, serialization, and outbox payload logic where safe.

Do not call the legacy broad demo orchestrator directly.
Build a selected InstallationV0 call list only.

## Outbox policy

For each selected data row, follow the proven legacy policy:

```text
TblTenant                 -> outbox
TblPosLocal               -> no outbox, local-only exception
TblSetupLoginMethod       -> outbox
TblSetupPaymentMethod     -> outbox
TblEmployeePermission     -> outbox
TblSetupWeird             -> outbox
TblSetupServicesMethod    -> outbox
TblParameterSetting       -> outbox
TblSetupPrinter           -> outbox
Completion marker         -> no outbox
```

Do not copy historical outbox rows from the reference DB.
Create new deterministic target outbox rows using Phase 1 Tenant/POS source identity.

If a compatible target data row already exists and the corresponding baseline outbox is absent, create one deterministic baseline insert event only when the legacy policy requires it.

No duplicate outbox on same-version replay.

## Existing target state policy

The target currently contains some prior Development baseline rows.

For each selected stable key:

```text
absent -> insert transformed reference row + required outbox
present and compatible -> adopt/verify; add required missing deterministic outbox
present but conflicting -> rollback with PHASE2_BASELINE_CONFLICT
extra rows outside selected manifest -> preserve; do not delete or modify
```

Do not truncate or wipe selected tables.
Do not delete target data merely to match the reference DB.

## Real PostgreSQL executor — required

Implement the missing executable path, not only an in-memory executor or script string builder.

Create a real executor equivalent to:

```text
PostgreSqlPhase2ReferenceSeedExecutor
```

It must use:

```text
one DbContext / Npgsql connection
one PostgreSQL transaction
one tenant/POS/version advisory transaction lock
selected data rows
matching outbox rows
marker written last
verification before commit
one final COMMIT
```

Multiple `SaveChangesAsync()` calls are allowed only when all remain inside the same underlying transaction.

Forbidden:

```text
separate transaction per table
separate outbox connection
commit between seed groups
placeholder script without execution
admin credential for seed
```

Any failure:

```text
ROLLBACK selected data changes
ROLLBACK outbox changes
ROLLBACK marker
Phase 1 artifacts unchanged
```

## WPF explicit operator action

Wire a real explicit Phase 2 action in InstallationV0.

Do not auto-run seed on WPF startup.

Enable only when:

```text
Phase 1 checkpoint exists
DPAPI credential can be unprotected
protected hello PASS
/bootstrap/me identity PASS
target = obm_pos_dev_v0_pg
environment = Development
V007 rollback anchor validation PASS
runtime role hung privilege proof PASS
marker is not already conflicting/newer
```

UI safe proof should show:

```text
Reference template DB: enailsalon_phasee1_pos1_pg (read-only)
Target DB: obm_pos_dev_v0_pg
Reference rows selected
Existing rows adopted
Rows inserted
Outbox rows inserted
Marker written
Transaction committed/rolled back
Runtime tables unchanged
```

## Version and label

Create a new immutable trial version:

```text
phase2-reference-driven-trial-v001
```

Store versioned source artifacts under:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\reference-driven-v001\
```

Preserve prior trial folders and current pointer history.

Because this prompt changes WPF/source, set:

```text
Build label: prompt028
Window title: OBM InstallationV0 Phase 1/2 - prompt028
```

## Testing and physical execution

Build/test:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Tests must prove:

- reference DB path is read-only and never target;
- target hard guard;
- sanitized reference row selection;
- exact column transformation classifications;
- parent-before-child ordering;
- deterministic GUID/FK remap;
- compatible existing-row adoption;
- conflict rollback;
- data + outbox + marker share one transaction;
- marker last and readback before commit;
- same-version replay zero delta;
- explicit no-seed tables absent from plan;
- prompt028 label.

After tests pass, execute the physical seed on exactly `obm_pos_dev_v0_pg` using role `hung`.

Collect:

```text
before counts
reference-selected manifest counts
inserted/adopted data counts
inserted outbox counts
marker count
runtime/excluded table deltas
transaction user/database proof
```

Run the same version a second time and prove:

```text
selected data delta = 0
TblLocalOutbox delta = 0
marker delta = 0
runtime/excluded delta = 0
```

Then launch or hand WPF to the operator for runtime testing and record the first missing default/table, if any, for v002.

## Report 028

Create and push:

```text
report/report028.md
```

Required sections:

1. Verdict.
2. Confirmation prompt027 superseded.
3. Reference read-only proof.
4. Valid V007 rollback-anchor proof.
5. Exact sanitized reference-driven manifest and row counts.
6. Parameter classification and final included safe keys.
7. Printer complete row/type list and column transformation matrix.
8. Permission/login/payment/singleton exact selected rows.
9. Dependency/FK order.
10. Exact identity/GUID/FK transformation rules.
11. Reused legacy save/outbox methods.
12. Real PostgreSQL executor/operator action implementation.
13. Exact source files changed.
14. One-transaction/marker-last proof.
15. Before/after physical target counts.
16. Inserted versus adopted rows.
17. Outbox mapping and physical deltas.
18. Runtime/excluded-table zero-delta proof.
19. Same-version replay zero-delta proof.
20. Build/test counts.
21. WPF prompt028 handoff.
22. First missing default/table observed or v001 sufficient.
23. No reference mutation/no secret leakage/no source push.
24. Coordination commit SHA.

## Valid verdicts

Physical reference-driven seed PASS:

```text
PHASE2_REFERENCE_DRIVEN_TRIAL_V001_POS1_DB_PASS_READY_FOR_WPF_TEST
```

Implementation ready, physical action pending operator click:

```text
PHASE2_REFERENCE_DRIVEN_TRIAL_V001_READY_FOR_USER_TEST
```

Reference read blocked:

```text
BLOCKED_PHASE2_REFERENCE_READ_ACCESS
```

Implementation/physical seed blocked:

```text
BLOCKED_PHASE2_REFERENCE_DRIVEN_PHYSICAL_SEED
```
