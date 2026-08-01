# Prompt 023 — Consolidate the proven seed/save path into one atomic POS1 trial transaction

## Supersession and current source state

Read completely before changing anything:

```text
report/report014.md
report/report015.md
report/report016.md
report/report019.md
prompt/prompt021.md
prompt/prompt022.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
E:\Project2026\RecoveryReports\InstallationV0\Phase2SeedAuditV003\
```

This prompt supersedes prompts 020, 021, and 022 as execution instructions.

Prompt019 already created a local Phase 2 scaffold and produced:

```text
Build: PASS
Focused InstallationV0 tests: 34/34 PASS
Visible label: prompt019
Physical PostgreSQL seed: NOT RUN
Verdict: BLOCKED_PHASE2_POS1_TRIAL_TARGET_SAFETY
```

Do not discard useful prompt019 source merely because the physical seed was blocked. Reuse or adapt it where useful.

## Operator’s consolidated seed decisions

The operator clarified the original working model:

1. The WPF seed/save code written approximately two weeks ago was already used successfully to seed a database.
2. That existing row-construction/default-value logic is useful and should be reused first.
3. The old implementation inserted the actual row with the new `TenantGuid` and also created the corresponding `TblLocalOutbox` record.
4. The main correction now is to collect the selected baseline operations under one shared PostgreSQL transaction.
5. Seed ordering must follow physical FK and logical parent/child dependency: parent first, child later.
6. Runtime/business tables such as invoice/output tables must remain empty because their rows arise only after sales or runtime activity.
7. A trial-and-error POS1 development lane is approved. Start with a reasonable baseline, run WPF, observe what is missing, add only the proven missing defaults in v002/v003, then freeze the latest successful version for packaging.

The preferred implementation is therefore:

```text
find the exact prior successful seed/save path
-> reuse proven entity construction, values, mapping, and outbox generation
-> select only installation baseline tables
-> inject Phase 1 Tenant/POS identity
-> order parent before child
-> execute data rows + matching outbox rows + marker in one transaction
-> run POS1
-> record first missing table/row/default
-> create the next version only when needed
```

Do not redesign the already-working runtime setup/property-loading logic.

## Exact approved POS1 development target

The approved local trial target is:

```text
obm_pos_dev_v0_pg
```

This is the only database that prompt023 may mutate.

Hard reject:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
recovery_api_day16_pg
any database not exactly obm_pos_dev_v0_pg
any production/reference/protected database
```

Before mutation, prove all of the following using sanitized evidence:

```text
resolved database name = obm_pos_dev_v0_pg
environment = Development
target is writable by the already-approved WPF development connection path
target is not in the protected/reference deny list
Phase 1 Tenant/POS identity is available
```

Do not print a connection string, password, token, pgpass content, or protected credential.

If the canonical current WPF database configuration does not resolve exactly to `obm_pos_dev_v0_pg`, stop with:

```text
BLOCKED_PHASE2_POS1_TARGET_MISMATCH
```

Do not silently switch targets.

## Mandatory rollback anchor before first mutation

Create a new versioned rollback anchor before seeding:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV001\PreSeedBackup\
```

Do not overwrite any existing backup/artifact. If that folder exists, create `Phase2Pos1TrialV002`, `V003`, etc. and report the chosen version.

The anchor must include, using the existing approved local PostgreSQL credential/config path without revealing secrets:

```text
custom-format pg_dump of obm_pos_dev_v0_pg
sanitized database metadata
pre-seed table counts
SHA256SUMS.txt
RESTORE-NOTES.md
```

Verify the dump command exits successfully and the file is non-empty. Do not perform a restore unless the seed fails and the existing recovery protocol explicitly authorizes it. Transaction rollback remains the first recovery mechanism.

If a safe rollback anchor cannot be created, stop before mutation:

```text
BLOCKED_PHASE2_POS1_ROLLBACK_ANCHOR
```

## Phase 1 prerequisite and identity source

Preserve the accepted Phase 1 artifacts:

```text
E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
InstallationV0\Checkpoints\api-authorized.json
InstallationV0\Secrets\bootstrap-credential.dpapi
LocalInstallationGuid
TenantGuid
PosGuid
POS name/slot
InstallationAttemptGuid
```

Before Phase 2 execution:

```text
read checkpoint
DPAPI-unprotect WpfJwt in memory
call protected hello endpoint
verify exact hello marker
call /bootstrap/me
verify Tenant/POS/attempt/local-installation identity
```

Do not redeem another Pairing Code.

If the bootstrap credential has expired, fail closed with a clear result such as:

```text
PHASE1_CREDENTIAL_REAUTH_REQUIRED
```

Do not erase or rotate Phase 1 artifacts automatically.

## First task — locate and prove the prior successful seed/save implementation

Investigate source/history and reports around approximately 2026-07-14 through 2026-07-20.

Primary areas:

```text
E:\Project2026\4POS\NailSalonNet8\SeedDb
E:\Project2026\4POS\NailSalonNet8\Services
E:\Project2026\4POS\NailSalonNet8\MyData
E:\Project2026\4POS\NailSalonNet8\InstallationV0
E:\Project2026\RecoveryReports
local git history for the date range above
```

Likely methods include:

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
CreateLocalOutboxSingleAsync or equivalent outbox helpers
```

For every relevant method, record:

```text
source path and method
physical table written
row/default values constructed
TenantGuid/PosGuid behavior
PK/FK behavior
SaveChanges behavior
transaction behavior
outbox behavior
whether it was in the prior successful seed path
reuse unchanged / small adaptation / exclude
```

Do not re-audit runtime business modules in depth.

## Reuse-first rule

Preserve proven code for:

```text
entity construction
required non-null values
canonical labels/default flags
column mapping
serialization
normalization
TblLocalOutbox entity/payload construction
```

Allowed adaptations:

```text
pass Phase 1 TenantGuid/PosGuid/POS name/slot
replace demo/hard-coded identity
remap or deterministically generate row GUIDs
order calls by dependency
suppress excluded demo/business tables
make all selected operations use a caller-owned transaction
remove nested independent transaction/commit boundaries
add marker-last and verification
```

Do not replace useful proven table-specific methods with a generic JSON framework merely for architectural purity.

Prompt019’s generic/template scaffold may remain only where it reduces risk and cleanly delegates to proven save/mapping logic. Remove or adapt any scaffold rule that conflicts with this prompt, especially its previous `TblLocalOutbox delta = 0` assumption.

## Selected initial baseline domain

Start with these configuration/identity tables only:

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
TblLocalOutbox support rows
```

### Complete default rows

For default/config tables, reuse the complete safe default row set from the proven successful seed implementation. Do not reduce counts merely to satisfy an earlier estimate.

### Roles

Seed only:

```text
Owner
Admin
Sub_Manager
```

`Sub_Manager` is the physical representation of operator-facing `SubAdmin`.

Do not seed employees or PINs.

### Parameters

Reuse the safe parameter rows from the prior successful clean/dev baseline. At minimum, preserve proven current startup/default rows such as:

```text
Date Hien Tai
EnableTurnEngine
```

Do not copy secrets, gateway credentials, terminal values, private URLs, or salon-specific historical data.

Because this is an approved trial lane, do not block merely because the permanent parameter set may later grow. Record the selected v001 list and add proven missing rows only in v002+.

### Printer settings

Insert the complete default `TblSetupPrinter` row set used by the proven seed path, including all currently supported logical types:

```text
CUSTOMER_TERMINAL
MERCHANT_TERMINAL
POS_CUSTOMER
POS_MERCHANT
POS_EMPLOYEE
```

These are logical print settings, not five physical printers.

Preserve logical values/flags used by existing WPF code, including settings controlling behavior such as:

```text
POS_EMPLOYEE: whether checkout prints an employee copy
POS_CUSTOMER: whether checkout auto-prints a customer copy
MERCHANT_TERMINAL: whether merchant-copy data is sent for terminal-side printing
```

Do not modify the existing `foreach`/static-property/runtime-loading code. Ensure all expected rows exist so that existing code continues to work.

Transform only columns that truly require transformation:

```text
TenantGuid -> Phase 1 TenantGuid
PosGuid -> Phase 1 PosGuid where the entity has it
row GUID -> reuse deterministic/persisted safe identity strategy
FK GUID -> remap to the new parent identity
timestamps -> current canonical execution time
machine-secret/private invalid values -> clear/bind later only when proven necessary
```

Do not blank logical print flags/defaults.

## Dependency-first insertion order

Build the actual dependency map from:

```text
Phase2SeedAuditV003\fk-dependency.tsv
Phase2SeedAuditV003\constraints.tsv
EF model/annotations
current proven save path
```

Distinguish:

```text
physical FK dependency
logical identity dependency
no dependency
```

Expected high-level order, subject to actual metadata:

```text
1. TblTenant
2. TblPosLocal
3. independent lookup/default rows
4. tenant/POS-scoped singleton/config rows
5. TblLocalOutbox rows associated with each seeded row, using the established mapping
6. completion marker last
```

A table-specific method may call `SaveChangesAsync` more than once if required to obtain parent/generated identities, provided every call remains enlisted in the same underlying transaction and no independent commit occurs.

## TblLocalOutbox is part of the atomic seed

The prior successful seed inserted baseline data with the new `TenantGuid` and generated `TblLocalOutbox` rows. Preserve that behavior.

For every selected seed method/table, create an evidence matrix:

```text
data table
stable key
insert/update operation
outbox required by prior proven path? yes/no
outbox entity name
outbox operation
payload source
TenantGuid source
SourceClientId/source POS identity
transaction identity
```

Rules:

- Data row and its required outbox row must be in the same transaction.
- No orphan outbox row.
- No outbox row if the corresponding data mutation rolls back.
- Same-version idempotent replay must not duplicate either data or outbox.
- Reuse the prior safe payload/entity mapping when proven.
- Do not emit private/secret data.
- Do not copy historical outbox rows from another database.

Do not assume outbox delta is zero.
Do not invent outbox for rows that the proven path treats as local-only; report those exceptions explicitly.

## One shared PostgreSQL transaction

All selected data rows, matching outbox rows, verification, and completion marker must use one connection/DbContext transaction:

```text
BEGIN
  revalidate Phase 1 identity
  verify exact target obm_pos_dev_v0_pg
  acquire tenant/POS/version pg_advisory_xact_lock
  verify marker/current-state eligibility
  seed parent rows
  seed dependent/default rows
  create matching TblLocalOutbox rows according to proven policy
  verify row counts/stable keys/outbox mappings
  verify excluded runtime tables unchanged
  write completion marker last
  read back marker and invariants
COMMIT
```

Any error:

```text
ROLLBACK all data rows
ROLLBACK all outbox rows
ROLLBACK marker
leave Phase 1 checkpoint/credential unchanged
surface an explicit safe error
```

Multiple `SaveChangesAsync()` calls are permitted only inside that single shared transaction.

No selected method may open a hidden second connection, begin an independent transaction, or commit independently.

## Versioned trial artifacts

Use version:

```text
phase2-pos1-legacy-reuse-trial-v001
```

Store new versioned source/coordination artifacts without overwriting prompt019 v001 artifacts:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\legacy-reuse-v001\
```

Maintain a `CURRENT.md` or equivalent pointer to the active trial version while preserving prior versions.

Future corrections use:

```text
legacy-reuse-v002
legacy-reuse-v003
...
```

The selected call list/template is the authority for row counts. Do not duplicate counts in unrelated constants.

## Explicit no-seed classifications

### User-created/imported later — no initial rows

```text
TblEmployee and employee PIN/private rows
TblServiceCategory
TblService
TblProduct
TblCustomer*
TblGiftCard*
```

Category/service remains the canonical dependency example:

```text
TblServiceCategory before TblService
```

but neither is part of this initial baseline.

### Runtime/business generated — schema only, zero seeded rows

```text
TblInvoice*
TblOutputInfo*
TblOutputInfoTam*
TblTerminalDejavoo
TblTerminalPaymentInfo
TblInvoiceBookingLink
booking/appointment rows
payment transaction/history tables
queue/turn/payroll runtime history
event delivery/log operational rows
```

These tables receive rows only after sales or runtime activity. Empty after installation is correct.

Do not deeply inspect or copy their real rows. Verify only that prompt023 causes zero delta.

## Current/partial database state behavior

Before mutation, inspect selected stable keys and marker state.

Required behavior:

```text
clean eligible target -> seed normally
same trial version already complete -> verify and return idempotent success
partial selected rows without valid marker -> stop with PHASE2_PARTIAL_BASELINE_RECOVERY_REQUIRED
conflicting stable-key values -> stop with PHASE2_BASELINE_CONFLICT
newer marker/version present -> stop with PHASE2_NEWER_MANIFEST_PRESENT
```

Do not blind-merge conflicting data and do not delete unrelated existing rows.

## WPF Phase 2 flow

Phase 2 must remain an explicit operator action after Phase 1 resume/revalidation.

UI should show:

```text
Phase 1 API Authorization: Complete
Phase 2 Local DB Baseline: Not Started / Running / Complete / Blocked
Target DB: obm_pos_dev_v0_pg (Development/Test)
Trial version: phase2-pos1-legacy-reuse-trial-v001
```

Action:

```text
Install Local Database Baseline
```

Do not auto-seed on WPF startup.

After execution, show safe proofs for:

```text
Phase 1 identity revalidated
target safety passed
rollback anchor verified
shared transaction started
parent/default rows seeded
outbox mappings verified
runtime tables unchanged
completion marker written last
transaction committed
```

## WPF build label

Because prompt023 changes WPF/source, update the canonical single label constant to:

```text
prompt023
```

Expected visible title/label:

```text
OBM InstallationV0 Phase 1/2 - prompt023
Build label: prompt023
```

Tests must prove prompt019 is absent from active title/header/build-info source.

## Build and test requirements

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Focused tests must cover:

- prompt023 label;
- exact approved target name and deny-list guards;
- rollback anchor prerequisite;
- prior successful seed method inventory/call list;
- dependency parent-before-child order;
- Phase 1 TenantGuid/PosGuid injection;
- complete printer default rows and logical flags preserved;
- selected data row + required outbox atomic pairing;
- one shared transaction across all SaveChanges calls;
- no independent/nested commit;
- marker written last;
- failure injection rolls back data + outbox + marker;
- idempotent replay creates no duplicate data/outbox;
- partial/conflicting/newer state blocks;
- zero seed delta for invoice/output/output-tam/runtime tables;
- zero seed delta for employee/catalog/customer/gift-card tables;
- Phase 1 artifacts unchanged on success/failure.

## Physical POS1 trial

After builds/tests pass and rollback anchor is verified, execute the first physical seed against exactly:

```text
obm_pos_dev_v0_pg
```

Never execute against the reference DB.

Collect sanitized before/after counts for:

```text
selected baseline tables
TblLocalOutbox
completion marker
explicit excluded/runtime tables
```

Run the same version a second time to prove idempotency:

```text
second-run baseline delta = 0
second-run outbox delta = 0
second-run marker delta = 0
```

Then launch WPF through the normal Visual Studio Debug handoff or leave it ready for operator launch according to current convention.

Observe and report the first missing table/row/default or runtime error. Do not immediately broaden the seed. That observation becomes the bounded scope for `legacy-reuse-v002`.

## Git safety

The source worktree is shared and dirty.

Do not run:

```text
git add .
git add -A
git reset
git clean
git stash
git checkout/restore unrelated files
```

Do not push OBM source. List exact source files changed.

Commit only:

```text
report/report023.md
```

to the coordination repository.

## Report 023 — 100% detail

Create and push:

```text
report/report023.md
```

Required sections:

1. Verdict.
2. Report019 scaffold assessment: reused/adapted/replaced pieces.
3. Exact approved target proof: `obm_pos_dev_v0_pg`.
4. Rollback anchor path/files/hashes.
5. Phase 1 revalidation/freeze proof.
6. Exact prior successful seed/save path found.
7. Reusable method matrix.
8. Selected baseline method/table call list.
9. Physical/logical dependency order.
10. TenantGuid/PosGuid/PK/FK transformation behavior.
11. Full `TblSetupPrinter` row list and preserved logical settings.
12. Data-to-`TblLocalOutbox` mapping matrix.
13. One shared transaction proof across all SaveChanges.
14. Marker-last and verification-before-commit proof.
15. Rollback/failure-injection evidence.
16. Same-version idempotent replay evidence.
17. Explicit no-seed runtime/user-data table proof.
18. Exact source files changed.
19. Build/test commands and counts.
20. Physical POS1 before/after counts.
21. Second-run zero-delta proof.
22. WPF handoff and prompt023 label proof.
23. First missing table/row/default observed, or confirmation v001 is sufficient for tested startup scope.
24. No reference DB mutation/no secret leakage/source no-push proof.
25. Coordination commit SHA.

## Valid verdicts

Physical database seed and replay passed, ready for WPF trial:

```text
PHASE2_LEGACY_REUSE_TRIAL_V001_POS1_DB_PASS_READY_FOR_WPF_TEST
```

Implementation/tests pass, physical mutation blocked by credential or backup safety:

```text
PHASE2_LEGACY_REUSE_TRIAL_V001_READY_FOR_USER_TEST
```

Target mismatch:

```text
BLOCKED_PHASE2_POS1_TARGET_MISMATCH
```

Rollback anchor blocked:

```text
BLOCKED_PHASE2_POS1_ROLLBACK_ANCHOR
```

Implementation blocked:

```text
BLOCKED_PHASE2_LEGACY_REUSE_TRANSACTION_IMPLEMENTATION
```
