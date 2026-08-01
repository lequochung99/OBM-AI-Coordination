# Prompt 027 — Implement the real PostgreSQL Phase 2 executor and run the POS1 physical trial

## Authoritative state

Read completely before changing source:

```text
report/report023.md
report/report024.md
report/report025.md
report/report026.md
prompt/prompt023.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Current proven state:

```text
Approved target DB: obm_pos_dev_v0_pg
Environment: Development
Runtime DB role: hung
Rollback anchor: valid
Rollback anchor path:
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV007\PreSeedBackup

Runtime dbo privileges for hung: PASS
Marker table exists: dbo."Phase2TrialCompletionMarker"
Marker row count: 0
Focused InstallationV0 tests: 37/37 PASS
Active WPF label: prompt023
Physical Phase 2 seed: never executed
```

The only remaining blocker is concrete:

```text
InstallationV0 has an in-memory executor and a PostgreSQL script builder,
but it does not have a real executable PostgreSQL transaction executor
or an enabled WPF/operator action that runs the Phase 2 plan.
```

## Objective

Implement the missing production-shaped Development executor and operator action, then execute the first physical Phase 2 POS1 trial against exactly:

```text
obm_pos_dev_v0_pg
```

The implementation must:

1. reuse the useful prompt023 plan, identity transformation, target guard, outbox plan, and marker-last design;
2. reuse the proven WPF legacy seed/save entity construction and `TblLocalOutbox` mapping where practical;
3. execute through one real `Npgsql`/EF Core connection and one caller-owned PostgreSQL transaction;
4. support compatible existing rows already present in the Development DB;
5. insert only missing approved rows, block conflicting rows, and create deterministic missing outbox rows;
6. write the completion marker last;
7. run the physical seed and same-version replay;
8. leave all runtime/business tables unchanged.

Do not redesign the Phase 2 architecture again. This is the implementation/closure task for the current v001 trial.

## Hard database boundary

Only this database may be mutated:

```text
obm_pos_dev_v0_pg
```

Hard reject:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
recovery_api_day16_pg
any database not exactly obm_pos_dev_v0_pg
any production/reference/protected database
```

Use the canonical WPF Development database configuration path. Do not hard-code or report a connection string/password.

The data seed must execute as:

```text
current_user = hung
current_database = obm_pos_dev_v0_pg
```

Do not use the postgres/admin credential for seed data execution. The admin credential is no longer needed for normal prompt027 seed execution because privileges and marker schema are already aligned.

## Rollback anchor rule

Use and verify the existing valid V007 anchor before mutation:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV007\PreSeedBackup
```

Verify:

- dump exists and is non-empty;
- `pg_restore-list.txt` exists and is non-empty;
- `SHA256SUMS.txt` exists;
- recorded hashes match current files.

Do not create another backup unless V007 validation fails or the DB changed materially outside the known prompt026 privilege/marker alignment.

Transaction rollback is the first recovery mechanism. Do not automatically restore V007 unless a non-transactional failure occurs and the operator explicitly approves restore.

## Existing Development DB state — adopt-compatible policy

Report026 proved the current pre-seed state:

```text
TblTenant: 1
TblPosLocal: 1
TblSetupLoginMethod: 0
TblSetupPaymentMethod: 0
TblEmployeePermission: 0
TblSetupWeird: 0
TblSetupServicesMethod: 0
TblParameterSetting: 6
TblSetupPrinter: 3
TblLocalOutbox: 0
Phase2TrialCompletionMarker: 0
```

This database was previously provisioned for Development and therefore is not a perfectly empty target.

For this approved trial, do not classify all compatible existing baseline rows as an unsafe partial installation merely because the Phase 2 marker is absent.

Use these rules:

### Compatible existing row

A selected row is compatible when:

- its stable key matches the v001 plan;
- Tenant/POS identity matches Phase 1 identity where scoped;
- required logical values match the canonical selected plan, or the existing value is explicitly accepted by the reused legacy seed contract;
- no secret/private/reference identity is present.

Action:

```text
adopt/verify row
insert no duplicate data row
create only the missing deterministic outbox row when the table's legacy policy requires outbox
```

### Missing selected row

Action:

```text
insert the row
insert its required matching TblLocalOutbox row
```

### Conflicting selected row

Action:

```text
rollback and return PHASE2_BASELINE_CONFLICT
```

Report only table and safe stable key; do not print private values.

### Extra rows not selected by the current v001 plan

Do not delete or mutate them during this trial.

Examples may include additional safe parameter rows created by earlier Development provisioning.

Record them as pre-existing extras and leave unchanged. The selected v001 stable keys, not exact whole-table counts, define compatibility.

## Selected v001 baseline plan

Keep the current prompt023 trial identity/version unless a source defect requires an explicit version correction:

```text
phase2-pos1-legacy-reuse-trial-v001
```

Selected tables/method groups:

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
TblLocalOutbox
Phase2TrialCompletionMarker
```

Explicitly excluded:

```text
TblEmployee and employee PIN/private rows
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
payment transaction/history tables
queue/turn/payroll runtime history
event delivery/log operational rows
```

These excluded tables must receive zero Phase 2 row delta.

## Real PostgreSQL executor — required implementation

Add a real implementation of the existing executor abstraction, for example:

```text
PostgreSqlPhase2TrialTransactionExecutor
```

Use the exact project naming conventions after inspecting current source.

The executor must:

1. receive the resolved Development DB configuration through the existing configuration boundary;
2. open exactly one EF Core/Npgsql connection;
3. begin exactly one database transaction;
4. acquire a transaction-scoped advisory lock derived from TenantGuid + PosGuid + trial version;
5. use one DbContext/connection/transaction for every selected data row, outbox row, verification query, and marker row;
6. commit exactly once;
7. rollback on every exception, conflict, cancellation, failed invariant, or marker failure;
8. never execute the generated SQL as an unreviewed free-form external script if typed EF/Npgsql operations are available.

Small table-specific adapters are acceptable because the legacy entity shapes differ. They must all participate in the same caller-owned transaction and must not start nested transactions or independent commits.

Multiple `SaveChangesAsync()` calls are allowed inside the one transaction.

## Reuse the proven legacy save/outbox path

Inspect and reuse/adapt:

```text
SeedDbProvider.RunLegacyDemoSeedAllAsync
MainServices.ExecuteInTransactionAsync
SeedTenantAsync
SeedPosAsync
SeedSetupLoginMethodAsync
SeedPaymentMethodsAsync
SeedEmployeePermissionAsync
SeedSetupWeirdAsync
SeedSetupServicesMethodAsync
SeedParameterSetingAsync
SeedSetupPrinterAsync
CreateLocalOutboxSingle / CreateLocalOutboxSingleAsync
```

Preserve useful proven behavior:

- entity construction;
- exact physical field names;
- required non-null values;
- logical default values;
- TenantGuid insertion;
- serialization/payload mapping;
- source-client and transaction identity conventions.

Adapt only what is required:

- Phase 1 TenantGuid/PosGuid/POS name/slot inputs;
- deterministic stable identities for replay;
- selected baseline call list;
- adopt-compatible existing-row behavior;
- one shared transaction;
- deterministic missing-outbox behavior;
- marker-last verification.

Do not call legacy demo methods for employees, services, customers, or other excluded domains.

## Outbox rules

Legacy policy established by report023:

```text
TblTenant: outbox required
TblPosLocal: no outbox, local-only exception
TblSetupLoginMethod: outbox required
TblSetupPaymentMethod: outbox required
TblEmployeePermission: outbox required
TblSetupWeird: outbox required
TblSetupServicesMethod: outbox required
TblParameterSetting: outbox required
TblSetupPrinter: outbox required
Completion marker: no outbox
```

Implement deterministic outbox identity/replay detection using the actual `TblLocalOutbox` entity/schema and existing helper conventions.

For each selected row requiring outbox:

- if the data row is newly inserted and no matching outbox exists, create one `I` outbox;
- if a compatible data row pre-exists but its matching v001 outbox is absent, create one deterministic baseline `I` outbox for this approved Development adoption trial;
- if the deterministic matching outbox already exists, create no duplicate;
- if a conflicting outbox identity/payload exists, rollback and block.

Use one trial `SeedBatchGuid`/transaction identity consistently.

Do not include secrets, Pairing Code, WpfJwt, PIN, terminal credentials, customer/employee private data, or historical reference rows in payloads.

Do not hard-code the expected outbox count separately from the selected plan. Derive it from the selected data rows and outbox policy.

## Identity transformation

Use Phase 1 identity as source of truth:

```text
TenantGuid <- Phase 1 authorized identity
PosGuid <- Phase 1 authorized identity where applicable
POS name/slot <- Phase 1 identity
SourceClientId <- POS:{PosGuid}
```

For row identities:

- preserve an existing compatible row identity when adopting it;
- generate deterministic identity for a missing row based on version + TenantGuid + PosGuid if scoped + table + stable key;
- remap dependent identities to the adopted/generated parent identity;
- never copy reference tenant/POS GUIDs.

## Transaction order

Use actual physical FK metadata and the approved logical order:

```text
BEGIN
  prove target/current_user/environment
  verify V007 anchor
  revalidate Phase 1 protected identity
  acquire pg_advisory_xact_lock
  inspect marker/newer/conflict state

  adopt-or-insert TblTenant
  adopt-or-insert TblPosLocal
  adopt-or-insert lookup/default rows
  adopt-or-insert singleton/config rows
  create missing deterministic TblLocalOutbox rows

  verify selected stable keys
  verify outbox mappings
  verify excluded/runtime table deltas = 0
  insert Phase2TrialCompletionMarker last
  verify marker and all invariants
COMMIT
```

Any failure:

```text
ROLLBACK all new data rows
ROLLBACK all new outbox rows
ROLLBACK marker
preserve pre-existing compatible rows
preserve Phase 1 checkpoint/credential
```

## Marker behavior

Use the existing table:

```text
dbo."Phase2TrialCompletionMarker"
```

Do not recreate or rename it in prompt027 unless its current schema is incompatible with the source contract.

Marker version:

```text
phase2-pos1-legacy-reuse-trial-v001
```

Write marker only after all selected rows, outboxes, excluded-table delta checks, and readback invariants pass.

Same-version marker present:

- verify all selected data/outbox invariants;
- return idempotent success;
- add zero rows.

Newer marker present:

```text
PHASE2_NEWER_MANIFEST_PRESENT
```

## Real WPF/operator action

Wire the Phase 2 UI button/action to the real PostgreSQL executor.

The action must remain explicit; do not auto-seed on application startup.

Enable only when:

- Phase 1 checkpoint exists;
- DPAPI credential can be unprotected;
- protected hello and `/bootstrap/me` revalidation pass;
- exact target is `obm_pos_dev_v0_pg`;
- environment is Development;
- V007 rollback anchor validates;
- runtime role proof passes;
- Phase 2 is not currently running.

UI must display safe stages:

```text
Phase 1 prerequisite revalidated
Target database verified
Rollback anchor verified
Runtime role verified
Advisory lock acquired
Transaction started
Existing rows adopted
Missing rows inserted
Outbox mappings verified
Runtime tables unchanged
Completion marker written
Transaction committed
```

On failure, show safe result code and confirmation that transaction rolled back.

## WPF label

Prompt027 changes WPF/source, therefore update the canonical build label to:

```text
prompt027
```

Expected visible title:

```text
OBM InstallationV0 Phase 1/2 - prompt027
Build label: prompt027
```

Focused tests must prove `prompt023` is absent from the active title/header/build-info source.

## Testing requirements

Add focused tests for:

- real executor selected through the WPF/operator action;
- exact target/environment/current-user guard;
- V007 anchor validation gate;
- one real transaction abstraction across data/outbox/marker operations;
- existing compatible row adoption;
- missing row insertion;
- extra unselected row preservation;
- conflicting row rollback;
- deterministic missing-outbox creation for adopted rows;
- no duplicate outbox on replay;
- marker written last;
- same-version replay zero delta;
- newer marker block;
- excluded/runtime tables unchanged;
- Phase 1 artifacts unchanged after failure;
- prompt027 label.

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

## Physical execution — required

After builds/tests pass, run the physical seed as role `hung` against exactly `obm_pos_dev_v0_pg`.

Before first run, capture sanitized counts for:

```text
selected baseline tables
TblLocalOutbox
Phase2TrialCompletionMarker
excluded/runtime proof tables
```

Run 1 must prove:

- Phase 1 revalidation passed;
- compatible existing rows were adopted, not duplicated;
- missing selected rows were inserted;
- required deterministic outbox rows were created;
- marker was written last;
- excluded/runtime deltas are zero;
- commit succeeded.

Run the exact same action a second time.

Run 2 must prove:

```text
selected data delta = 0
TblLocalOutbox delta = 0
marker delta = 0
excluded/runtime delta = 0
```

If physical execution fails inside the transaction, verify rollback with before/after counts and do not restore V007 automatically.

## WPF handoff

After physical seed/replay PASS:

- leave no background WPF process unless the established Visual Studio handoff requires it;
- provide exact operator steps to start WPF in Debug;
- operator should see `prompt027`;
- Phase 1 should resume without second redeem;
- Phase 2 should show Complete/idempotent;
- record the first missing table/row/default observed during WPF startup, or state that v001 is sufficient for the tested startup surface.

## Source/Git safety

The OBM source worktree is shared and dirty.

Do not run:

```text
git add .
git add -A
git reset
git clean
git stash
checkout/restore unrelated changes
```

Do not push OBM source.

Commit only:

```text
report/report027.md
```

to the coordination repository.

## Report 027 — 100% detail

Create:

```text
report/report027.md
```

Required sections:

1. Verdict.
2. Exact blocker from report026 and correction.
3. V007 anchor validation proof.
4. Exact source files changed.
5. Real PostgreSQL executor design and call chain.
6. WPF operator action wiring/enabling rules.
7. Existing-row adoption rules and physical findings.
8. Selected stable-key plan and extra-row handling.
9. Data-to-outbox mapping and deterministic replay behavior.
10. One-transaction/one-commit proof.
11. Marker-last proof.
12. Rollback/conflict/newer-marker behavior.
13. Build/test commands and counts.
14. Physical run-1 before/after/deltas.
15. Physical run-2 zero-delta proof.
16. Excluded/runtime zero-delta proof.
17. Phase 1 revalidation/no-second-redeem proof.
18. Prompt027 label proof.
19. WPF operator handoff and first runtime observation.
20. No reference DB mutation/no secret leakage/no source push.
21. Coordination commit SHA.

## Valid verdicts

Physical seed and replay passed:

```text
PHASE2_LEGACY_REUSE_TRIAL_V001_POS1_DB_PASS_READY_FOR_WPF_TEST
```

Implementation/build/tests complete but physical execution blocked by a new concrete safety/runtime issue:

```text
BLOCKED_PHASE2_POS1_PHYSICAL_SEED
```

Implementation blocked:

```text
BLOCKED_PHASE2_POSTGRESQL_EXECUTOR_IMPLEMENTATION
```
