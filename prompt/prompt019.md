# Prompt 019 — Implement POS1 trial seed using full default setting rows

## Operator correction

Prompt018 is superseded and must not be executed as written.

The operator clarified that printer behavior is already implemented in WPF. The printer-related rows are configuration/default-setting rows consumed by existing code, which iterates the table and loads values into static/runtime properties.

Do not spend time re-designing or re-auditing the meaning of individual printer types. Do not choose an arbitrary subset.

Use this simpler rule:

```text
For each approved default/config table:
  copy the complete canonical default row set
  preserve logical values
  replace/remap Tenant/POS identity fields
  regenerate/remap row GUIDs and FK references as required
  insert in dependency order
```

For the printer setting table specifically:

```text
Insert the complete canonical TblSetupPrinter default row set.
```

This includes all currently supported logical rows such as:

```text
CUSTOMER_TERMINAL
MERCHANT_TERMINAL
POS_CUSTOMER
POS_MERCHANT
POS_EMPLOYEE
```

These are logical configuration purposes, not a requirement for five physical printers.

Examples already handled by existing WPF logic:

- `POS_EMPLOYEE`: optionally print an employee copy at checkout.
- `POS_CUSTOMER`: optionally auto-print a customer copy.
- `MERCHANT_TERMINAL`: send merchant-copy information to the terminal for terminal-side printing.

Do not modify that runtime logic. Only ensure the complete default setting rows exist so the existing foreach/static-property loading code can operate.

## Trial-and-error lane

The operator explicitly approves an iterative POS1 test lane:

```text
choose a reasonable canonical baseline
-> seed POS1 test
-> run WPF
-> observe missing row/table/default
-> create next version v002/v003
-> repeat until complete
-> package the latest successful version
```

Do not block implementation because a theoretically perfect manifest cannot be proven in advance.

## Architecture

Do not recreate the legacy fragmented seed design with one special method per table.

Implement:

```text
one versioned template dataset
+ one generic identity/value transformer
+ one explicit table dependency order
+ one PostgreSQL transaction executor
+ one verification result
+ one completion marker
```

Template row counts are authoritative. Do not maintain independent hard-coded counts that can drift.

## First trial version

Create:

```text
phase2-pos1-trial-seed-v001
```

Store under:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\v001\
```

Never overwrite this version. Future changes use v002, v003, and so on.

Maintain a clear latest/current pointer while preserving all prior versions.

## Approved initial tables

Use a practical initial baseline drawn from the canonical/default rows already identified in report014/report015/source:

```text
TblTenant
TblPosLocal
TblSetupWeird
TblSetupServicesMethod
TblSetupLoginMethod
TblSetupPaymentMethod
TblEmployeePermission
TblParameterSetting
TblSetupPrinter
Phase 2 trial completion marker
```

### Complete-row rule

For each selected default/config table, use the full canonical default row set from the approved template source. Do not arbitrarily reduce counts merely to fit an earlier estimate.

The initial template may use the existing safe/default rows from the known test/reference baseline, but must exclude private, historical, machine-secret, credential, customer, employee-personal, transaction, and salon-specific data.

### Roles

Include:

```text
Owner
Admin
Sub_Manager
```

Do not seed employee rows or PINs.

### Parameters

Start with a small practical set that is directly needed by current startup/default behavior. The minimum accepted starting set is:

```text
Date Hien Tai
EnableTurnEngine
```

Additional safe default rows may be included when they are part of the canonical default template and are clearly non-secret/non-historical.

Do not block implementation merely because the final permanent parameter set is not yet perfect. Missing settings can be added in v002+ after POS1 testing.

### Printers

Insert the complete canonical `TblSetupPrinter` default row set.

Preserve its logical settings/default booleans and type rows.

Transform only identity/environment fields as required:

- replace `TenantGuid` with Phase 1 TenantGuid;
- replace/remap `PosGuid` if present;
- regenerate deterministic row GUIDs;
- remap internal FKs;
- derive timestamps;
- clear only fields proven to be machine-secret/private or invalid for a new machine.

Do not blank logical configuration flags that control printing behavior.

Do not modify WPF foreach/static-property loading logic.

## Identity transformation

For every copied template column classify and implement one of:

```text
Preserve canonical value
Replace from Phase 1 identity
Generate deterministic GUID
Remap FK
Clear private/environment value
Derive at runtime
```

Do not simply copy old GUIDs.

At minimum:

```text
TenantGuid <- Phase 1 TenantGuid
PosGuid/slot/name <- Phase 1 POS identity where applicable
row GUID <- deterministic from version + tenant + table + stable key
FK GUID <- remapped to new deterministic parent GUID
created/updated timestamps <- current UTC or canonical execution time
```

## Dependency order

Inspect actual FK metadata and use the minimal correct order.

Do not invent separate seed sessions per table. All selected tables must be inserted through one transaction executor.

## Transaction

Use one PostgreSQL transaction:

```text
BEGIN
  revalidate Phase 1 authorization/identity
  verify POS1 test target and schema eligibility
  acquire tenant/POS/version advisory transaction lock
  insert/verify selected template rows in dependency order
  verify operational/runtime tables unchanged
  write completion marker last
  read back template row counts and stable keys
COMMIT
```

Any error:

```text
ROLLBACK everything
no partial seed
no completion marker
Phase 1 checkpoint unchanged
```

## Outbox trial policy

For trial v001:

```text
Do not create TblLocalOutbox rows unless an existing database constraint or immediate runtime requirement proves they are necessary.
```

Default expectation:

```text
TblLocalOutbox delta = 0
```

A future trial version may add selective outbox rows after ownership/sync behavior is proven.

## POS1 test boundary

The operator has an existing POS1 test environment and approves direct trial-and-error testing there.

Before mutation, identify and report the exact sanitized target DB classification and prove it is not:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
production/reference/protected database
```

Use only the approved dev/test POS1 target.

Take or verify an existing rollback anchor before the first trial mutation.

Do not mutate the reference DB.

## WPF label

Because this prompt changes WPF/source, update the canonical visible label to:

```text
prompt019
```

Expected UI:

```text
OBM InstallationV0 Phase 1/2 - prompt019
Build label: prompt019
```

## Testing

Build and run focused InstallationV0 tests.

Prove:

- template-driven row counts;
- complete printer default row set inserted;
- printer logical flags preserved;
- Tenant/POS identity transformed;
- deterministic GUID/FK remap;
- one transaction owns all baseline rows and marker;
- rollback leaves zero partial rows;
- same-version replay is idempotent;
- outbox delta is zero unless explicitly proven otherwise;
- excluded operational tables unchanged;
- Phase 1 artifacts unchanged;
- visible label is prompt019.

Perform the first POS1 test seed when the target/rollback guard passes.

Then launch WPF for operator testing or leave it ready for Visual Studio Debug, depending on the current runtime handoff convention.

## Explicit exclusions

Do not seed:

```text
employees/staff/PINs
services/categories/products
customers/gift cards
invoices/output/payment transactions
bookings/appointments
queue/turn/payroll runtime history
terminal credentials
historical outbox rows
private machine/salon data
```

## Report

Create:

```text
report/report019.md
```

Report must include:

1. Verdict.
2. Confirmation prompt018 was superseded.
3. Exact POS1 test target classification and rollback anchor.
4. Exact selected tables and template row counts.
5. Full canonical printer default row/type list.
6. Which printer columns were preserved/replaced/remapped/cleared/derived.
7. Exact dependency order.
8. Generic transformer design.
9. One-transaction proof.
10. Completion-marker-last proof.
11. Idempotent replay proof.
12. Outbox delta proof.
13. Excluded-table delta proof.
14. Exact source files changed.
15. Build/test commands and counts.
16. POS1 physical DB result and WPF handoff.
17. Prompt019 label proof.
18. No reference DB mutation/no secret leakage.
19. Next observed missing table/row/default, or confirmation v001 is sufficient.
20. Coordination commit SHA.

## Valid verdicts

Ready for operator WPF test:

```text
PHASE2_POS1_TRIAL_SEED_V001_READY_FOR_USER_TEST
```

POS1 DB seed physically passed:

```text
PHASE2_POS1_TRIAL_SEED_V001_POS1_DB_PASS_READY_FOR_WPF_TEST
```

Blocked only by test target/rollback safety:

```text
BLOCKED_PHASE2_POS1_TRIAL_TARGET_SAFETY
```

Implementation blocked:

```text
BLOCKED_PHASE2_POS1_TRIAL_SEED_IMPLEMENTATION
```
