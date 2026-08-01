# Prompt 018 — Implement iterative POS1 test baseline seed v001

## Operator decision

The operator explicitly approves a **trial-and-error development lane** for Phase 2 because a disposable/current POS1 test environment already exists.

Do not continue trying to prove a theoretically perfect seed manifest before implementation.

The working method is now:

```text
choose a small reasonable baseline subjectively
-> seed the POS1 test database
-> launch/test WPF
-> observe the first missing table/row/default
-> add only what is proven missing in the next version
-> repeat until startup and required screens pass
-> freeze the latest successful version as the packaged baseline
```

Prompt016 and prompt017 are superseded for execution purposes. Their reports remain historical evidence, but neither blocks this experimental lane.

## Core simplification

Do not reproduce the old fragmented architecture of many special-case methods such as:

```text
SeedParameterSettingAsync
SeedSetupPrinterAsync
SeedEmployeePermissionAsync
SeedPaymentMethodsAsync
...
```

Implement a new, isolated InstallationV0 Phase 2 baseline mechanism based on:

```text
one versioned template dataset
+ one generic identity/value transformer
+ one explicit table order
+ one PostgreSQL transaction executor
+ one verification result
+ one completion marker
```

The template is the source of expected row counts. Do not maintain independent hard-coded counts that can drift from the template.

## First experimental manifest

Create the first version as:

```text
phase2-pos1-trial-seed-v001
```

Source folder:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\v001\
```

Never overwrite this folder in future iterations. Future corrections must create `v002`, `v003`, and so on. Maintain a clear pointer/index identifying the current trial version, but preserve every prior version.

## Approved subjective v001 row selection

Use this exact initial set without further proof gate:

```text
1 TblTenant row
1 TblPosLocal row
1 TblSetupWeird row
1 TblSetupServicesMethod row
3 TblSetupLoginMethod rows
6 TblSetupPaymentMethod rows
3 TblEmployeePermission rows
2 TblParameterSetting rows
5 TblSetupPrinter placeholder rows
1 Phase 2 trial completion marker row
```

Non-marker baseline row total:

```text
23
```

### Roles

Seed exactly:

```text
Owner
Admin
Sub_Manager
```

`Sub_Manager` remains the physical representation of operator-facing `SubAdmin`.

Do not seed employee rows or PINs.

### Parameters

Seed exactly these two initial keys:

```text
Date Hien Tai
EnableTurnEngine
```

Use a canonical current-date/business-date value for `Date Hien Tai` derived at execution time according to the existing WPF business-day convention.

Use a conservative safe default for `EnableTurnEngine` that preserves current InstallationV0 behavior; document the exact value and reason in report018.

Do not seed the other feature flags yet. Missing keys are allowed to fall back to current code behavior. Add a key only in a later version when an observed failure or required feature proves it necessary.

### Printers

Seed placeholder rows for all five currently supported enum types:

```text
CUSTOMER_TERMINAL
MERCHANT_TERMINAL
POS_CUSTOMER
POS_MERCHANT
POS_EMPLOYEE
```

This avoids arbitrarily selecting three of five supported types.

For every placeholder row:

- preserve only safe logical `PrinterType` identity;
- use blank/neutral bind-later values for physical printer name, IP, Windows path, share, driver, image path, machine name, and terminal binding;
- do not copy machine-specific values from any existing salon/reference database.

### Login/payment defaults

Use the safe label sets already proven by report014/current source:

Login methods:

```text
Login-Smart
Login-With-Password
No-Login
```

Payment methods:

```text
Cash
Coupon
Credit Card
Checks
Gift Card
Membership
```

Use the exact physical spelling/casing required by current WPF source/entity constraints after checking existing code. Report the final exact values.

### Singleton settings

Seed one tenant-scoped row each:

```text
TblSetupWeird
TblSetupServicesMethod
```

Use conservative defaults from current WPF source/legacy safe defaults. Do not copy tenant-private values from the reference DB.

### Deferred from v001

Do not seed:

```text
TblSetting
TblTurnSetting
other TblParameterSetting keys
employees/staff
service categories/services/products
customers/gift cards
invoices/output/payment transactions
bookings/appointments
queue/turn/payroll runtime history
terminal credentials
historical TblLocalOutbox rows
```

## Outbox decision for the trial lane

For `phase2-pos1-trial-seed-v001`:

```text
TblLocalOutbox rows = 0
```

The initial goal is to make the local POS database start correctly. Do not emit one outbox event per baseline row merely because the legacy seed did so.

A later trial version may add selective outbox events only when ownership/sync behavior is proven necessary.

The report must explicitly confirm the outbox remains unchanged/empty for this trial seed.

## Identity transformation rules

Do not simply copy old GUIDs.

For every template row classify every identity field as one of:

```text
Preserve canonical constant
Replace from Phase 1 identity
Generate deterministic GUID
Clear to null/blank
Derive at execution time
```

Required rules:

- `TenantGuid` comes from the Phase 1 authorized checkpoint/API identity.
- `PosGuid`, POS name/code, slot/station identity come from Phase 1.
- `LocalInstallationGuid` remains the Phase 1 machine identity.
- Primary GUIDs for template rows must be deterministic from:

```text
trial manifest version + TenantGuid + table name + stable row key
```

- Any FK to a generated template GUID must be remapped to the corresponding deterministic GUID.
- Timestamps are generated at execution time in UTC unless the entity contract requires otherwise.
- Private, terminal, machine-path, credential, customer, employee, and historical fields are cleared or excluded.

## Table order and FK safety

Use live metadata from `report014` / Phase2SeedAuditV003 and current EF mappings.

Candidate tables had no direct FK chain in the audited reference set, but still use this logical order:

```text
1. TblTenant
2. TblPosLocal
3. TblSetupLoginMethod
4. TblSetupPaymentMethod
5. TblEmployeePermission
6. TblSetupWeird
7. TblSetupServicesMethod
8. TblParameterSetting
9. TblSetupPrinter
10. Phase 2 trial marker
```

Before inserting, verify the actual target schema and constraints. If the test schema differs, adjust the order based on real FK metadata and record the change.

## One transaction

All Phase 2 trial mutations must use one underlying Npgsql/EF transaction:

```text
BEGIN
  acquire transaction-scoped advisory lock
  validate target database safety
  validate Phase 1 identity
  validate trial version state
  insert/verify the 23 baseline rows in dependency order
  verify TblLocalOutbox row delta = 0
  verify excluded operational-table row delta = 0
  insert completion marker last
  read back all stable keys/counts
COMMIT
```

Any exception or invariant failure:

```text
ROLLBACK all 23 baseline rows
ROLLBACK marker
Phase 1 checkpoint unchanged
trial version remains incomplete
```

No partial manual insert and no catch-and-continue.

## Trial marker

Use a dedicated InstallationV0 marker table through the canonical migration/schema path:

```text
dbo.TblInstallationV0Phase2SeedVersion
```

Marker version:

```text
phase2-pos1-trial-seed-v001
```

Marker must include enough safe proof for:

```text
ManifestVersion
ManifestHash
SeedBatchGuid
TenantGuid
PosGuid
LocalInstallationGuid
InstallationAttemptGuid
BaselineRowCount = 23
OutboxRowCount = 0
AppliedAtUtc
Status = Complete
```

Marker is inserted last, after readback verification of baseline rows.

## Test target safety

The operator approves mutation only against the existing current POS1 **development/test** database.

Before any mutation, Codex must discover the active database name from the canonical WPF test configuration and prove it is a development/test lane.

Hard reject:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
any production/salon/reference database
any database not clearly classified as dev/test/disposable
```

Do not print credentials or connection strings.

If the active POS1 database cannot be proven safe, stop with:

```text
BLOCKED_PHASE2_POS1_TEST_DATABASE_NOT_PROVEN_SAFE
```

Do not create a new database with administrator credentials unless an existing approved disposable harness already provides that capability.

## Existing-state behavior for the POS1 test DB

This is an iterative test lane, but do not destructively clear unknown data.

Before seed, capture safe row counts for:

```text
all v001 baseline tables
marker table
TblLocalOutbox
excluded operational tables
```

Behavior:

- Empty/eligible baseline tables: run v001.
- Same v001 marker and exact rows: return idempotent success.
- Existing rows matching stable keys and values: treat as verified/no-op.
- Existing conflicting row: block with safe table/stable-key evidence.
- Partial trial rows without marker: block and report the exact safe partial state; do not delete automatically.
- Newer trial marker: block older v001.

## Phase 1 prerequisite

Before Phase 2 execution:

```text
read existing ApiAuthorized checkpoint
DPAPI-unprotect bootstrap credential in memory
call protected hello
verify exact hello marker
call /bootstrap/me
verify Tenant/POS/Attempt/LocalInstallation identity
```

No new Pairing Code and no second redeem.

Do not alter Phase 1 checkpoint or credential on Phase 2 failure.

## UI and prompt label

Prompt018 changes WPF, so update the single canonical build label to:

```csharp
public const string CoordinationPromptLabel = "prompt018";
```

Visible UI:

```text
OBM InstallationV0 Phase 1/2 - prompt018
Build label: prompt018
Phase 1 API Authorization: Complete
Phase 2 Trial Baseline: Not Started / Running / Complete / Blocked
Trial version: phase2-pos1-trial-seed-v001
Install Trial Baseline
```

Do not auto-run the seed on WPF startup. Require explicit operator action.

After execution display safe evidence:

```text
Target DB proven dev/test
Phase 1 identity revalidated
Single transaction started
23 baseline rows verified
TblLocalOutbox delta = 0
Excluded operational-table delta = 0
Marker written last
Transaction committed
```

## Generic implementation requirement

The executor must not contain a large switch of hand-written `SeedXxxAsync` workflows.

Preferred structure:

```text
TrialManifest
TrialTableGroup
TrialRowTemplate
IdentityTransformer
GenericRowWriter or a small typed adapter layer
TrialTransactionExecutor
TrialVerifier
```

A small typed adapter is acceptable where EF/entity constraints require it, but the orchestration, versioning, identity transformation, count verification, and transaction ownership must remain generic and centralized.

Do not call legacy `RunAllAsync` or `RunLegacyDemoSeedAllAsync`.

## Iteration evidence and future versions

Create a versioned local evidence folder:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2TrialSeedV001\
```

Record:

```text
manifest snapshot/hash
pre-counts
post-counts
transaction proof
marker proof
excluded-table deltas
build/test outputs
physical test status
```

Do not overwrite prior audit folders.

Future rule:

```text
Observed missing dependency/default
-> create v002 folder/manifest
-> retain v001 unchanged
-> add only the proven missing rows/transform rules
-> rerun against a clean/recoverable test state
```

## Tests

Add focused tests for:

- prompt018 label;
- manifest has exactly 23 non-marker rows;
- exactly two parameter keys;
- exactly five printer types;
- no TblSetting/TblTurnSetting/outbox rows;
- deterministic GUID generation;
- Tenant/POS identity replacement;
- private/machine binding fields cleared;
- explicit logical table order;
- one transaction ownership;
- marker written last;
- rollback on failure at every table group;
- Phase 1 artifacts unchanged on failure;
- same-version idempotent replay;
- conflict/partial/newer marker blocking;
- reference/protected DB rejection;
- excluded operational table deltas remain zero.

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Run a physical seed only if the active POS1 DB passes the dev/test safety gate.

Do not leave WPF running after Codex testing. Hand it back for Visual Studio physical testing.

## No source Git push

Do not run:

```text
git add .
git add -A
reset
clean
stash
restore/checkout unrelated work
push source
```

List exact source files changed. Commit only:

```text
report/report018.md
```

to the public coordination repository.

## Report018 — 100% operational detail

Create:

```text
report/report018.md
```

Include:

1. Verdict.
2. Confirm prompt016/prompt017 superseded.
3. Active test DB safety classification.
4. Phase 1 revalidation design/proof.
5. Exact v001 template rows and values.
6. Identity transformation matrix.
7. Exact table order/FK findings.
8. Generic architecture and files changed.
9. Marker migration/schema.
10. Single transaction ownership proof.
11. Outbox delta-zero proof.
12. Excluded table delta-zero proof.
13. Rollback/idempotency/conflict behavior.
14. Build/test commands and counts.
15. Physical POS1 test result or exact reason pending.
16. Prompt018 visible label proof.
17. Local evidence path/hashes.
18. No reference/protected DB mutation/no secret proof.
19. Exact operator physical test steps.
20. Recommended v002 delta process.
21. Coordination commit SHA.

## Valid verdicts

Implementation and tests complete, operator physical WPF test pending:

```text
PHASE2_POS1_TRIAL_SEED_V001_READY_FOR_USER_TEST
```

Implementation and approved POS1 test DB physical seed passed:

```text
PHASE2_POS1_TRIAL_SEED_V001_POS1_DB_PASS_READY_FOR_WPF_TEST
```

Unsafe target:

```text
BLOCKED_PHASE2_POS1_TEST_DATABASE_NOT_PROVEN_SAFE
```

Implementation blocker:

```text
BLOCKED_PHASE2_POS1_TRIAL_SEED_V001_IMPLEMENTATION
```
