# Prompt 021 — Reuse proven legacy seed/save code and consolidate into one transaction

## Operator correction

Prompt020 is superseded and must not be executed as written.

The operator clarified an important implementation fact:

- the WPF seed/save code written roughly two weeks ago was already used successfully to seed a database once;
- therefore that row-construction and save logic is useful and should not be discarded merely because it is split across several methods;
- the main correction now is to inspect the proven path, select only the required baseline methods/tables, order them by dependency, and run them under **one shared PostgreSQL transaction**.

Preferred approach:

```text
find the exact prior successful seed/save path
-> preserve proven row values and entity mapping
-> remove demo/runtime/business tables from the selected call list
-> order selected methods by parent/child dependency
-> make every selected method use the same DbContext/connection/transaction
-> commit once after verification
```

Do not redesign a new generic framework unless the existing code is genuinely unusable. Reuse first; refactor only the transaction boundary, identity inputs, selected table list, and safety checks.

## Task objective

Implement the first POS1 Phase 2 trial by reusing the previously successful seed/save implementation.

The goal is not to rewrite every `SeedXxxAsync` method. Small, readable table-specific methods are acceptable and preferred when they already work.

The hard requirements are:

1. one approved call list;
2. actual dependency order;
3. one shared transaction;
4. one final commit;
5. complete rollback on any failure;
6. no rows in runtime/generated business tables;
7. Phase 1 identity substituted correctly.

## Historical successful implementation investigation

Before modifying source, locate the exact seed implementation that successfully populated a database approximately two weeks before this prompt.

Search:

```text
E:\Project2026\4POS\NailSalonNet8\SeedDb
E:\Project2026\4POS\NailSalonNet8\Services
E:\Project2026\4POS\NailSalonNet8\InstallationV0
E:\Project2026\RecoveryReports
E:\Project2026\CanonicalInstallationDocs
local git history around 2026-07-14 through 2026-07-20
```

Likely relevant paths include, but are not limited to:

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
```

Do not assume all of these belong in the new baseline. Determine which exact methods produced the previously accepted clean/dev seed and which were only demo/sample data.

Report for every reusable method:

```text
method/path
rows/table written
input identity source
whether it calls SaveChanges
whether it opens its own transaction
whether it emits TblLocalOutbox
whether it was part of the prior successful seed
reuse unchanged / reuse with small adaptation / exclude
```

## Reuse rule

Preserve working code for:

- entity construction;
- exact default values;
- column mapping;
- required non-null fields;
- stable logical labels;
- existing safe normalization;
- existing save behavior that is compatible with a shared transaction.

Do not replace proven code with a new JSON/template/generic writer merely for architectural purity.

Allowed refactoring:

- pass the Phase 1 `TenantGuid`, `PosGuid`, POS name/slot, and related identity into existing methods;
- replace old/demo GUID values;
- ensure deterministic or safely persisted row GUID behavior;
- suppress excluded demo/business tables;
- suppress legacy outbox creation for the trial unless immediately required;
- make methods participate in a caller-owned transaction;
- remove nested transaction/independent commit boundaries;
- add verification and marker-last behavior.

## Selected initial baseline tables

Start from the previously discussed configuration/default set:

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

Use the complete proven default rows from the old successful code for these tables, subject to privacy/environment cleanup and Phase 1 identity replacement.

### Printer settings

Insert the full default `TblSetupPrinter` row set used by the working code.

Preserve logical configuration flags and printer-purpose rows. Do not change the existing runtime code that iterates these rows and loads static/runtime properties.

Only clear or replace machine-specific/private values that are invalid for a new installation.

### Parameters

Reuse the parameter rows from the previously successful clean/dev seed rather than inventing an arbitrary count.

For trial v001, it is acceptable to use the proven prior parameter set even if later POS1 testing shows more rows are needed. Add missing rows in v002+.

Do not copy secrets, private URLs, terminal credentials, or salon historical values.

### Roles

Initial permission rows remain:

```text
Owner
Admin
Sub_Manager
```

Do not seed employee/PIN rows.

## Dependency-first call order

Inspect actual FK metadata and logical dependencies.

At minimum, verify whether this order is valid:

```text
TblTenant
-> TblPosLocal
-> independent lookup/default tables
-> tenant-scoped singleton/config tables
-> completion marker
```

If no physical FK exists, still respect logical identity order.

The previous category/service example remains the rule for any future import:

```text
parent category before child service
```

But category/service are not part of this initial baseline.

## Explicit no-seed tables

Do not call legacy methods that seed:

```text
TblEmployee or employee PIN/private data
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
booking/appointment rows
queue/turn/payroll runtime history
event delivery/log operational history
```

`TblInvoice`, `TblOutputInfo`, `TblOutputInfoTam`, and related tables only receive rows after sales/business activity. Correct initial row count is zero.

Do not investigate or copy their business rows.

## One shared transaction — central requirement

The selected existing seed methods must run under one caller-owned transaction.

Acceptable pattern:

```text
open one DbContext/connection
BEGIN transaction
  revalidate Phase 1 authorization and identity
  verify POS1 dev/test target
  acquire tenant/POS/version advisory lock
  call selected existing seed/save methods in dependency order
  each method uses the same DbContext and transaction
  SaveChanges may occur within the shared transaction when required
  verify selected default rows
  verify runtime/excluded tables remain empty
  verify outbox policy
  write completion marker last
  final readback
COMMIT once
```

Important distinction:

```text
multiple SaveChanges inside one shared transaction = allowed
multiple independent transactions/commits = forbidden
```

Any exception or failed invariant:

```text
ROLLBACK the shared transaction
no baseline rows committed
no marker committed
Phase 1 checkpoint unchanged
```

Do not create nested transactions unless PostgreSQL savepoints are intentionally used only for tests and do not permit partial success.

## Outbox trial policy

Default trial expectation:

```text
TblLocalOutbox delta = 0
```

If reused legacy methods automatically emit outbox rows, do not accept that silently. Either:

- call/adapt the proven row-save portion without the old outbox write; or
- prove a specific row requires immediate outbox behavior.

Do not emit outbox for employees/catalog/customers or other excluded data because those seed methods must not run.

## Versioning

Create the first implementation under a versioned lane such as:

```text
phase2-pos1-reused-seed-v001
```

Store any new coordination/manifest wrapper under:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\reused-v001\
```

Do not duplicate working seed values unnecessarily. The versioned artifact may reference/compose proven code while recording:

- selected method list;
- dependency order;
- expected tables/rows;
- identity transformation;
- exclusion list;
- marker version.

Future corrections use `reused-v002`, `reused-v003`, etc. Never overwrite v001.

## Phase 1 and target safety

Revalidate Phase 1 protected hello and `/bootstrap/me` before enabling Phase 2.

Do not redeem another Pairing Code.

Use only the approved POS1 dev/test target. Never mutate:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
production/reference/protected databases
```

Verify or create a rollback anchor before physical trial mutation.

## WPF label

Because this prompt changes WPF/source, update the canonical label to:

```text
prompt021
```

Visible title/label:

```text
OBM InstallationV0 Phase 1/2 - prompt021
Build label: prompt021
```

Phase 2 remains an explicit operator action; no automatic seed on application startup.

## Tests and physical trial

Build and run focused InstallationV0 tests.

Required proof:

- exact prior successful seed/save path identified;
- selected old methods and excluded old methods documented;
- old proven row mappings/defaults preserved;
- dependency order verified;
- all selected methods share one DbContext/transaction;
- multiple SaveChanges, if present, remain inside the same transaction;
- one final commit only;
- failure injection rolls back every selected table and marker;
- same-version replay is idempotent;
- complete printer default rows present and logical flags preserved;
- runtime tables remain zero;
- invoice/output/output-temp tables remain zero;
- employee/catalog/customer/gift-card tables remain zero;
- outbox delta remains zero unless specifically justified;
- Phase 1 artifacts unchanged;
- visible label is prompt021.

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
report/report021.md
```

Report must include:

1. Verdict.
2. Confirmation prompt020 was superseded.
3. Exact successful seed/save implementation from roughly two weeks ago.
4. Evidence that it previously seeded a database successfully.
5. Complete reusable/excluded method matrix.
6. Exact selected tables and row counts from proven code.
7. Dependency and call order.
8. Identity transformation details.
9. Shared DbContext/transaction ownership proof.
10. SaveChanges-versus-commit explanation.
11. Outbox suppression/justification.
12. Marker-last and verification proof.
13. Rollback/failure-injection proof.
14. Idempotent replay proof.
15. Runtime/excluded-table zero-row proof.
16. Exact source files changed.
17. Build/test commands and counts.
18. POS1 physical seed result and rollback anchor.
19. WPF handoff and prompt021 label proof.
20. First missing table/row/default observed, or confirmation v001 is sufficient.
21. No reference DB mutation/no secret leakage.
22. Coordination commit SHA.

## Valid verdicts

Physical POS1 seed passed:

```text
PHASE2_REUSED_SEED_V001_POS1_DB_PASS_READY_FOR_WPF_TEST
```

Implementation ready, physical test pending:

```text
PHASE2_REUSED_SEED_V001_READY_FOR_USER_TEST
```

Target safety blocked:

```text
BLOCKED_PHASE2_POS1_TRIAL_TARGET_SAFETY
```

Implementation blocked:

```text
BLOCKED_PHASE2_REUSED_SEED_IMPLEMENTATION
```
