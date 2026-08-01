# Prompt 022 — Reuse proven insert path with TenantGuid injection and same-transaction TblLocalOutbox

## Operator correction

Prompt021 is superseded and must not be executed as written.

The operator clarified the exact behavior of the previously successful seed implementation:

```text
build/execute INSERT data
-> inject the installation TenantGuid into each applicable row
-> write the matching TblLocalOutbox row at the same time
```

This path already seeded a database successfully approximately two weeks ago. Therefore:

- the existing insert/entity construction code is the preferred implementation source;
- the existing per-row outbox behavior is useful and must be investigated for reuse;
- Phase 2 should not default to `TblLocalOutbox delta = 0` when the proven path deliberately emits outbox records;
- the main refactor is to select the correct baseline inserts, order them by dependency, substitute Phase 1 identity, and execute both data inserts and matching outbox inserts inside one shared PostgreSQL transaction.

## Objective

Implement the first InstallationV0 Phase 2 POS1 trial by reusing the proven seed/save path.

Required shape:

```text
Phase 1 identity revalidation
-> select approved baseline insert methods/rows
-> parent tables first, dependent tables second
-> inject TenantGuid/PosGuid from Phase 1
-> insert each baseline row
-> insert its matching TblLocalOutbox row through the proven mapping
-> verify all selected baseline rows and outbox mappings
-> write completion marker last
-> commit once
```

Any failure must roll back:

```text
all baseline rows
all matching TblLocalOutbox rows
completion marker
```

Phase 1 checkpoint and DPAPI credential remain unchanged.

## First step — locate the exact proven implementation

Search read-only before modifying source:

```text
E:\Project2026\4POS\NailSalonNet8\SeedDb
E:\Project2026\4POS\NailSalonNet8\Services
E:\Project2026\4POS\NailSalonNet8\InstallationV0
E:\Project2026\RecoveryReports
E:\Project2026\CanonicalInstallationDocs
local git history around 2026-07-14 through 2026-07-20
```

Likely methods/helpers include:

```text
SeedDbProvider.RunAllAsync
SeedDbProvider.RunLegacyDemoSeedAllAsync
MainServices.ExecuteInTransactionAsync
SeedTenantAsync
SeedPosAsync
SeedPaymentMethodsAsync
SeedParameterSetingAsync
SeedSetupPrinterAsync
SeedSetupLoginMethodAsync
SeedSetupWeirdAsync
SeedSetupServicesMethodAsync
SeedEmployeePermissionAsync
CreateLocalOutboxSingleAsync or equivalent
```

For every candidate method, report:

```text
path/class/method
physical table written
whether it injects TenantGuid
whether it injects PosGuid
whether it generates row GUID
whether it calls SaveChanges
whether it opens/commits its own transaction
whether it writes TblLocalOutbox
outbox entity/operation/payload mapping
whether it participated in the prior successful seed
reuse unchanged / small adaptation / exclude
```

Do not rewrite proven row construction/default values unless required for identity or transaction safety.

## Selected initial baseline

Reuse the previously working insert logic for only these initial Phase 2 tables:

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
Phase 2 completion marker
```

### Roles

Seed only:

```text
Owner
Admin
Sub_Manager
```

Do not seed employee rows or PINs.

### Parameters

Use the proven existing insert/default logic. Initial minimum:

```text
Date Hien Tai
EnableTurnEngine
```

Additional safe non-secret defaults may be reused when they clearly belonged to the prior successful baseline. Do not copy private URLs, gateway credentials, terminal credentials, tokens, passwords, or salon-specific historical values.

### Printer settings

Insert the complete canonical `TblSetupPrinter` default row set already consumed by the existing foreach/static-property code:

```text
CUSTOMER_TERMINAL
MERCHANT_TERMINAL
POS_CUSTOMER
POS_MERCHANT
POS_EMPLOYEE
```

Preserve logical default flags and behavior fields.

Replace/remap only identity or machine-specific fields:

```text
TenantGuid <- Phase 1 TenantGuid
PosGuid <- Phase 1 PosGuid if present
row GUID <- deterministic or safely persisted new GUID
FK GUID <- remapped
physical printer binding/private machine fields <- safe default or bind later
```

Do not change the runtime foreach/static-property loading logic.

## Dependency-first order

Build the actual dependency map from:

```text
Phase2SeedAuditV003\fk-dependency.tsv
Phase2SeedAuditV003\constraints.tsv
EF model/entity configuration
existing successful seed call order
```

Distinguish:

```text
physical FK dependency
logical identity dependency
no dependency
```

Use parent-before-child order. Expected logical order, subject to verified metadata:

```text
TblTenant
-> TblPosLocal
-> independent lookup/default rows
-> tenant/POS-scoped settings/default rows
-> matching TblLocalOutbox rows as each baseline mutation is staged
-> completion marker last
```

Do not add `TblServiceCategory` or `TblService` to the initial baseline. Their parent-before-child example only defines the general dependency method.

## Identity injection

The Phase 1 authorized identity is the only source for installation identity.

For each reused insert method:

```text
TenantGuid <- Phase 1 checkpoint/API identity
PosGuid/PosName/slot <- Phase 1 POS identity where applicable
LocalInstallationGuid/InstallationAttemptGuid <- Phase 1 proof where marker requires them
```

Never retain a demo/reference TenantGuid or PosGuid.

For all row/FK GUIDs, either:

- reuse an existing deterministic helper; or
- add deterministic generation based on manifest version + TenantGuid + PosGuid/scope + table + stable key.

Same-version replay must not create duplicate identities.

## TblLocalOutbox — required proven behavior

For every selected baseline insert/update that the prior successful seed emitted to `TblLocalOutbox`, preserve the matching outbox creation.

Each outbox row must:

```text
use TenantGuid from Phase 1
use source client/POS identity from Phase 1 where required
use the same caller-owned DbContext/connection/transaction
reference the inserted/updated entity deterministically
use the proven entity name and operation contract
contain deterministic/sanitized payload
```

Do not emit outbox rows for excluded tables.

Do not emit historical/demo outbox rows.

Do not include:

```text
password/PIN
Pairing Code/WpfJwt
terminal/payment credentials
customer/employee private data
reference DB dumps
private machine values
```

### Outbox count rule

Do not hard-code an arbitrary global count.

Expected outbox count is derived from the approved selected call list and proven existing outbox behavior:

```text
for each actual baseline insert/update that contractually emits outbox
-> exactly one matching outbox row
```

No outbox for a no-op same-version replay.

If an existing selected seed method did not emit outbox, do not add one without evidence that synchronization requires it. Report the exact per-table policy.

## One shared transaction

Refactor orchestration so all selected data inserts, their matching outbox rows, verification, and completion marker participate in one underlying PostgreSQL transaction.

Allowed:

```text
multiple SaveChangesAsync calls inside the same caller-owned transaction
```

Forbidden:

```text
nested independent transaction
independent commit inside a seed method
outbox written through another DbContext/connection
catch-and-continue after a failed table
marker committed before final verification
```

Required flow:

```text
BEGIN
  revalidate Phase 1 protected hello and bootstrap/me identity
  verify approved POS1 dev/test target and rollback anchor
  acquire tenant/POS/version advisory transaction lock
  execute selected parent inserts
  execute selected dependent/default inserts
  create matching TblLocalOutbox rows through proven helper/mapping
  verify baseline stable keys/counts
  verify outbox-to-baseline mapping
  verify excluded/runtime tables unchanged/empty
  write completion marker last
  read back marker and invariants
COMMIT
```

Any exception or invariant failure:

```text
ROLLBACK
no partial baseline rows
no orphan outbox rows
no completion marker
Phase 1 unchanged
```

## Explicit no-seed tables

Do not seed:

```text
TblInvoice*
TblOutputInfo*
TblOutputInfoTam*
TblTerminalDejavoo
TblTerminalPaymentInfo
payment transaction/history
bookings/appointments
queue/turn/payroll runtime history
event delivery/log operational history
TblEmployee and PIN/private employee data
TblServiceCategory
TblService
TblProduct
TblCustomer*
TblGiftCard*
```

These tables receive schema only and must remain empty/unchanged after initial installation.

## POS1 trial boundary

The operator approves direct trial-and-error only on the existing POS1 development/test target.

Before mutation, prove the target is not:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
production/reference/protected DB
```

Verify or create an explicit rollback anchor before mutation.

If target/rollback safety cannot be proven, stop with:

```text
BLOCKED_PHASE2_POS1_TRIAL_TARGET_SAFETY
```

## WPF label

Because prompt022 changes WPF/source, update the single canonical label to:

```text
prompt022
```

Visible UI:

```text
OBM InstallationV0 Phase 1/2 - prompt022
Build label: prompt022
```

Phase 2 remains an explicit operator action after Phase 1 resume/revalidation. Do not auto-seed on application startup.

## Tests

Build and run focused InstallationV0 tests proving:

- exact prior successful seed/save path identified;
- selected methods reuse proven entity/default construction;
- dependency order is correct;
- all selected methods use one caller-owned transaction;
- TenantGuid/PosGuid are injected from Phase 1;
- matching outbox rows are created through the proven mapping;
- baseline row and outbox row roll back together;
- no orphan outbox rows;
- same-version replay creates no duplicate baseline/outbox rows;
- marker is last;
- runtime/generated tables receive zero seed rows;
- excluded user/business tables receive zero seed rows;
- Phase 1 artifacts remain unchanged;
- label is prompt022.

Required commands:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Perform the POS1 trial seed only after safety and rollback guards pass.

## Report

Create:

```text
report/report022.md
```

Report must include:

1. Verdict.
2. Confirmation prompt021 was superseded.
3. Exact prior successful seed/save call chain and evidence date/path.
4. Per-method reuse/exclude matrix.
5. Exact selected baseline tables/rows.
6. Physical/logical dependency order.
7. TenantGuid/PosGuid injection proof.
8. Per-table TblLocalOutbox policy and exact mapping/helper reused.
9. One-transaction ownership proof.
10. SaveChanges versus commit boundary proof.
11. Rollback/no-orphan-outbox proof.
12. Same-version replay proof.
13. Completion-marker-last proof.
14. Explicit runtime/excluded-table zero-delta proof.
15. Exact source files changed.
16. Build/test commands and counts.
17. POS1 target classification and rollback anchor.
18. POS1 physical seed result and WPF handoff.
19. Prompt022 visible label proof.
20. First missing table/row/default observed, or confirmation current trial is sufficient.
21. No reference DB mutation/no secret leakage.
22. Coordination commit SHA.

## Valid verdicts

POS1 seed and outbox physically passed:

```text
PHASE2_REUSED_INSERT_OUTBOX_V001_POS1_DB_PASS_READY_FOR_WPF_TEST
```

Implementation/tests ready, physical POS1 seed pending:

```text
PHASE2_REUSED_INSERT_OUTBOX_V001_READY_FOR_USER_TEST
```

Target safety blocked:

```text
BLOCKED_PHASE2_POS1_TRIAL_TARGET_SAFETY
```

Implementation blocked:

```text
BLOCKED_PHASE2_REUSED_INSERT_OUTBOX_IMPLEMENTATION
```
