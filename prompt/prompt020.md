# Prompt 020 — Dependency-first Phase 2 POS1 trial seed

## Operator correction

Prompt019 is superseded and must not be executed.

The operator clarified the preferred seed method from the earlier POS implementation:

```text
inspect table dependencies and foreign keys
-> insert parent tables first
-> insert child tables after their parents
-> replace TenantGuid/POS identity for the new installation
-> leave runtime-generated tables empty
```

Canonical example:

```text
TblServiceCategory must exist before TblService when service/catalog seed is ever performed.
```

However, service categories and services are not part of the initial InstallationV0 baseline. The example defines the dependency-first method, not approval to seed catalog data now.

## Core rule

Before implementing any seed row, classify every relevant table into one of:

```text
A. Required parent/default table — seed first
B. Required dependent/default table — seed after its parent
C. User-created/imported later — schema only, no initial rows
D. Runtime/transaction generated — schema only, must start empty
E. Optional/deferred default — add only after POS1 trial proves it is needed
```

The seed engine must follow actual FK/dependency order, not the historical order of fragmented `SeedXxxAsync` methods.

## Runtime tables — explicit no-seed decision

The following are generated only when business activity occurs and must not receive baseline rows:

```text
TblInvoice*
TblOutputInfo*
TblOutputInfoTam*
TblTerminalDejavoo
TblTerminalPaymentInfo
TblInvoiceBookingLink
payment transaction/history tables
booking/appointment rows
queue/turn/payroll runtime history
event delivery/log operational rows
```

These tables require schema only. Empty row count after installation is the correct state.

Do not investigate their real business rows deeply. Only verify that the trial seed leaves them unchanged/empty.

Also do not seed initially:

```text
TblEmployee and employee PIN/private rows
TblServiceCategory
TblService
TblProduct
TblCustomer*
TblGiftCard*
```

These are created/imported later through application workflows.

## Task mode

Prompt020 is a narrow dependency-first investigation plus implementation task.

It must not repeat broad architecture audits. Perform one focused dependency pass, record the result, then implement the POS1 trial baseline when the target safety guard passes.

Read completely:

```text
report/report014.md
report/report015.md
report/report016.md
prompt/prompt019.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
E:\Project2026\RecoveryReports\InstallationV0\Phase2SeedAuditV003\
```

## Step 1 — Build the real dependency map first

Use, in priority order:

1. `Phase2SeedAuditV003\fk-dependency.tsv` and `constraints.tsv`;
2. EF Core model configuration and entity annotations;
3. PostgreSQL catalog inspection only if an approved read-only path is already available;
4. current WPF lookup/read paths.

For every selected baseline table, report:

```text
Table
PK
FK parent(s)
Tenant/POS scope
Required before startup?
Seed classification A-E
Insert order
Reason
```

Do not invent dependencies. If no physical FK exists, distinguish:

```text
physical FK dependency
logical identity dependency
no dependency
```

Expected logical starting order should be verified, not blindly assumed:

```text
TblTenant
-> TblPosLocal
-> lookup/default tables
-> tenant-scoped singleton/default tables
-> completion marker
```

If actual metadata requires another order, follow actual metadata.

## Step 2 — Initial selected baseline tables

Start from this practical set:

```text
TblTenant
TblPosLocal
TblSetupLoginMethod
TblSetupPaymentMethod
TblEmployeePermission
TblSetupWeird
TblSetupServicesMethod
TblParameterSetting
TblSetupPrinter
Phase 2 trial completion marker
```

Do not add catalog, employee, customer, invoice, output, booking, terminal transaction, or history tables merely because they exist in the schema.

## Complete default-row rule

For tables that are genuinely default/config tables, insert the complete approved canonical default row set rather than choosing arbitrary subsets.

### `TblSetupPrinter`

Insert the complete canonical default rows, including all supported logical types:

```text
CUSTOMER_TERMINAL
MERCHANT_TERMINAL
POS_CUSTOMER
POS_MERCHANT
POS_EMPLOYEE
```

Preserve logical settings/boolean flags because existing WPF code iterates these rows and loads static/runtime properties.

Do not modify that existing runtime logic.

Transform only identity/environment columns as appropriate:

```text
TenantGuid -> Phase 1 TenantGuid
PosGuid -> Phase 1 PosGuid when present
row GUID -> deterministic generated GUID
FK GUID -> remapped parent GUID
created/updated timestamps -> trial execution timestamp
physical/private/machine-secret fields -> clear or bind later
```

Do not blank logical flags that control printing behavior.

### Login/payment/permissions/singletons

Use the complete safe canonical default rows for:

```text
TblSetupLoginMethod
TblSetupPaymentMethod
TblEmployeePermission: Owner, Admin, Sub_Manager only
TblSetupWeird
TblSetupServicesMethod
```

Do not seed employees or PINs.

### Parameters

Use a small safe trial set sufficient for startup and current proven reads. At minimum:

```text
Date Hien Tai
EnableTurnEngine
```

Additional safe non-secret canonical defaults may be included when clearly part of the test/default template.

Do not copy:

```text
passwords
tokens
private URLs
gateway/terminal credentials
salon-specific historical values
```

Missing optional parameters may be added in v002+ after POS1 testing.

## Versioned trial artifact

Create:

```text
phase2-pos1-dependency-trial-v001
```

Source folder:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\dependency-v001\
```

Never overwrite this version. Later corrections use `dependency-v002`, `dependency-v003`, etc.

Maintain a clear current/latest pointer while preserving prior versions.

The template itself is the authority for expected row counts. Do not maintain independent constants that can drift.

## Identity transformation

Do not merely replace `TenantGuid` and copy all other identities.

Every template column must be classified as:

```text
preserve canonical value
replace from Phase 1 identity
generate deterministic GUID
remap FK
clear private/environment value
derive at execution time
```

Use deterministic row GUIDs based on:

```text
trial version + TenantGuid + PosGuid if scoped + table + stable key
```

This allows same-version replay without duplicate identities.

## Generic executor

Do not rebuild the old fragmented design with one method per table.

Implement:

```text
one versioned template
one dependency/order definition
one generic identity transformer
one transaction executor
one verifier
one completion marker
```

Small table-specific adapters are allowed only where entity shapes require them, but they must not become independent seed sessions or independent commits.

## One transaction

Use one underlying PostgreSQL transaction:

```text
BEGIN
  revalidate Phase 1 authorization and identity
  verify POS1 dev/test target and schema eligibility
  acquire tenant/POS/version advisory transaction lock
  verify no conflicting/newer/partial marker state
  insert/verify parent tables first
  insert/verify dependent/default tables next
  verify runtime-generated tables remain unchanged/empty
  verify TblLocalOutbox delta
  write completion marker last
  read back template stable keys/counts
COMMIT
```

Any failure:

```text
ROLLBACK all seeded rows
ROLLBACK marker
no partial seed
Phase 1 checkpoint unchanged
```

## Outbox trial policy

Default for dependency trial v001:

```text
TblLocalOutbox delta = 0
```

Do not emit one event per seed row unless an immediate schema/runtime requirement proves it necessary.

Outbox behavior can be added selectively in later trial versions after ownership/sync requirements are proven.

## POS1 test target safety

The operator approves trial-and-error only on the existing POS1 development/test environment.

Before mutation, prove the target is not:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
production/reference/protected DB
```

Use only the approved dev/test POS1 target. Take or verify a rollback anchor before the first mutation.

If target classification or rollback safety cannot be proven, stop with:

```text
BLOCKED_PHASE2_POS1_TRIAL_TARGET_SAFETY
```

Do not mutate any reference database.

## WPF flow and label

Because prompt020 changes WPF/source, set the canonical label to:

```text
prompt020
```

Expected visible title/label:

```text
OBM InstallationV0 Phase 1/2 - prompt020
Build label: prompt020
```

Phase 2 must remain an explicit operator action after Phase 1 resume/revalidation. Do not auto-seed merely because WPF starts.

## Testing

Build and run focused InstallationV0 tests.

Required proof:

- dependency map created before seed implementation;
- parent-before-child order follows actual metadata;
- `TblTenant`/`TblPosLocal` use Phase 1 identity;
- complete `TblSetupPrinter` default rows inserted;
- logical printer flags preserved;
- deterministic GUID/FK remap;
- one transaction owns all selected seed rows and marker;
- rollback leaves no partial seed;
- same-version replay is idempotent;
- runtime-generated tables remain empty/unchanged;
- `TblInvoice*`, `TblOutputInfo*`, `TblOutputInfoTam*` receive zero seed rows;
- catalog/employee/customer/gift-card tables receive zero seed rows;
- outbox delta is zero unless explicitly proven otherwise;
- Phase 1 artifacts remain unchanged;
- visible label is `prompt020`.

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Perform the POS1 trial seed only after target and rollback guards pass.

## Report

Create:

```text
report/report020.md
```

Report must include:

1. Verdict.
2. Confirmation prompt019 was superseded.
3. Exact physical and logical dependency map.
4. A-E classification for relevant tables.
5. Explicit parent-before-child insertion order.
6. Explicit no-seed classification for `TblInvoice*`, `TblOutputInfo*`, `TblOutputInfoTam*`, and other runtime tables.
7. Exact selected template tables and row counts.
8. Complete printer default row/type list and logical flags handling.
9. Identity/FK transformation map.
10. Generic executor design and exact files changed.
11. One-transaction and marker-last proof.
12. Rollback/idempotent replay proof.
13. Outbox delta proof.
14. Runtime/excluded-table delta proof.
15. POS1 target classification and rollback anchor.
16. Build/test commands and counts.
17. POS1 physical seed result and WPF handoff.
18. Prompt020 label proof.
19. First missing table/row/default observed, or confirmation v001 is sufficient.
20. No reference DB mutation/no secret leakage.
21. Coordination commit SHA.

## Valid verdicts

POS1 database seed passed and ready for WPF test:

```text
PHASE2_DEPENDENCY_TRIAL_V001_POS1_DB_PASS_READY_FOR_WPF_TEST
```

Implementation ready but physical POS1 seed pending:

```text
PHASE2_DEPENDENCY_TRIAL_V001_READY_FOR_USER_TEST
```

Target safety blocked:

```text
BLOCKED_PHASE2_POS1_TRIAL_TARGET_SAFETY
```

Implementation blocked:

```text
BLOCKED_PHASE2_DEPENDENCY_TRIAL_IMPLEMENTATION
```
