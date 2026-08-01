# Prompt 029 — Add canonical non-staff management employees as Phase 2 v002

## Operator decision

The operator has refined the baseline policy:

```text
TblEmployee must include canonical non-staff management employee rows because management UI depends on them.
```

This changes the earlier broad rule that all employee rows were deferred.

The new rule is:

```text
Seed only canonical non-staff management employees required by the management UI.
Do not seed salon staff/technicians.
Do not copy real employee identities or private data from the reference database.
```

The existing checkout policy remains:

```text
checkout UI shows Staff only
management UI shows non-Staff only
```

The seeded management rows must therefore appear in management screens and must not appear as service-performing staff in checkout/queue flows.

## Authoritative state

Read completely:

```text
report/report028.md
prompt/prompt028.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
E:\Project2026\RecoveryReports\InstallationV0\Phase2SeedAuditV003\
```

Current accepted physical state:

```text
phase2-reference-driven-trial-v001 physically seeded on obm_pos_dev_v0_pg
rows verified: 23
outbox delta: 21
marker last: true
active WPF label: prompt028
```

Do not overwrite v001 source or marker. Implement an additive version:

```text
phase2-reference-driven-trial-v002
```

Source folder:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\reference-driven-v002\
```

Preserve all prior versions and update the current/latest pointer explicitly.

## Objective

Implement and physically test a v002 upgrade that adds the minimal canonical non-staff `TblEmployee` rows required for management UI operation.

The preferred canonical management set is:

```text
Owner
Admin
SubAdmin
```

Physical database permission name remains:

```text
Sub_Manager
```

Before inserting, verify the exact current source/schema mapping between:

```text
TblEmployee
TblEmployeePermission
EmployeeType
PermissionName
management UI filters
checkout UI filters
login/PIN behavior
```

Do not guess enum values or relationship columns. Use actual source/model/schema evidence.

## Reference database policy

Reference source:

```text
enailsalon_phasee1_pos1_pg
```

Read-only only:

```text
BEGIN TRANSACTION READ ONLY
SELECT-only inspection
ROLLBACK
```

Use the reference DB only to learn:

- exact `TblEmployee` schema/required columns;
- non-staff row shape;
- permission relationship shape;
- default boolean/status fields;
- any directly required parent/child configuration rows.

Do not copy:

- real employee names;
- phone/email/address;
- employee GUIDs;
- PIN/password/hash;
- payroll/commission/wage data;
- schedules/hours;
- service capability mappings;
- biometric/device/private identifiers;
- historical timestamps;
- salon-specific notes.

Public report must remain sanitized.

## Canonical v002 employee identities

Create synthetic, tenant-default management rows only.

Preferred operator-facing display identities:

```text
Owner
Admin
SubAdmin
```

Permission mapping:

```text
Owner    -> TblEmployeePermission.PermissionName = Owner
Admin    -> TblEmployeePermission.PermissionName = Admin
SubAdmin -> TblEmployeePermission.PermissionName = Sub_Manager
```

Use Phase 1 `TenantGuid`.

Use deterministic employee GUIDs derived from:

```text
phase2-reference-driven-trial-v002
+ TenantGuid
+ TblEmployee
+ canonical management key
```

Do not use reference employee GUIDs.

## EmployeeType rules

Verify exact enum/value semantics in current code.

Required outcome:

```text
all three seeded rows are non-Staff
all three appear in management UI
none appears in checkout employee selection
none participates as service/turn staff
```

Do not create `VirtualAnyTechnician` or any other virtual/staff employee in v002.

If the current schema uses only one non-staff EmployeeType value plus permission rows, use that proven design.

If Owner/Admin/SubAdmin require different type values, use the exact current code mapping.

## PIN / login safety

Known operator policy:

```text
Staff PIN length = 4 digits
Non-Staff PIN length = 6 digits
```

However, do not invent or copy a known default PIN.

Inspect current WPF login/bootstrap behavior and apply this hierarchy:

1. If `TblEmployee` PIN is nullable and management UI only needs the row, leave it null/empty as supported.
2. If the existing owner/manager bootstrap PIN remains code-owned, preserve that behavior and do not persist a duplicate DB PIN.
3. If a DB PIN is structurally required, use an existing secure generation/activation path and do not print the generated value.
4. Never seed a universal known PIN such as `000000`, `123456`, or equivalent.

If no safe compatible PIN state can be implemented without inventing a credential, stop with:

```text
BLOCKED_PHASE2_V002_NONSTAFF_PIN_POLICY
```

Do not weaken authentication to complete the seed.

## Direct dependencies only

Inspect real FK/logical dependencies for `TblEmployee`.

Expected parent order to verify:

```text
TblTenant
-> TblEmployeePermission
-> TblEmployee
```

Add only directly required parent/default rows that are missing.

Do not seed unrelated employee-domain child tables unless the management UI hard-fails without them.

Default exclusions remain:

```text
employee working hours
service capability/commission mappings
payroll rows
turn/queue state
employee devices/biometrics
staff schedules
```

If a required dependent row is proven by actual UI/runtime failure, include it explicitly in v002 and report the evidence.

## Outbox policy

The proven legacy seed/save path emitted `TblLocalOutbox` for `TblEmployee`.

For each newly inserted canonical management employee:

```text
insert TblEmployee row
insert matching deterministic TblLocalOutbox row
same transaction
operation = I
TenantGuid = Phase 1 TenantGuid
SourceClientId = Phase 1 POS source identity
```

For an already-compatible canonical row:

- adopt/verify the data row;
- create the missing deterministic baseline outbox only when absent;
- do not duplicate an existing deterministic outbox.

Do not emit employee PIN/private fields in the outbox payload.

## v001 and v002 marker behavior

Preserve the existing v001 marker.

Create a distinct v002 completion marker using the existing marker table contract:

```text
dbo."Phase2TrialCompletionMarker"
Version = phase2-reference-driven-trial-v002
```

v002 marker is written last, after employee and outbox verification.

Do not replace or mutate the v001 marker.

## One transaction

Use one target connection and one PostgreSQL transaction:

```text
BEGIN
  revalidate Phase 1 identity
  verify target = obm_pos_dev_v0_pg and Development
  verify v001 marker is complete
  acquire advisory transaction lock for v002
  verify Owner/Admin/Sub_Manager permission rows
  insert/adopt canonical non-staff TblEmployee rows
  insert/verify deterministic TblLocalOutbox rows
  verify management/non-staff invariants
  verify checkout/staff exclusion invariants
  write v002 marker last
  read back stable keys/counts
COMMIT
```

Any failure:

```text
ROLLBACK employee rows
ROLLBACK outbox rows
ROLLBACK v002 marker
preserve v001 marker
preserve Phase 1 artifacts
```

## New rollback anchor

V007 predates the physical v001 seed and is not sufficient as the pre-v002 anchor.

Before any v002 mutation, create a new versioned backup of the current post-v001 target state:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV008\PreV002Backup\
```

If V008 exists, use the next unused version.

Required:

- custom-format `pg_dump` using the approved admin backup credential;
- `pg_restore --list` validation;
- pre-v002 selected table counts;
- SHA256SUMS.txt;
- restore notes;
- sanitized metadata.

Do not overwrite V007 or earlier anchors.

## UI state hydration correction

After restart, prompt028 UI showed `Phase 2 Local DB Baseline: Not Started` even though v001 marker existed.

Correct startup/status hydration so WPF reads marker state and displays one of:

```text
v001 Complete — v002 Upgrade Available
v002 Complete
Blocked/Conflict
```

The Phase 2 action must not auto-run.

Before v002:

```text
Apply Local Database Baseline v002
```

After v002 completion:

```text
Verify Local Database Baseline
```

or disable the mutation action with clear `Complete` status.

## Target and safety boundary

Only mutate:

```text
obm_pos_dev_v0_pg
```

Hard reject all reference/protected/production DBs.

Reference DB remains read-only.

Use runtime role `hung` for v002 data mutation. Admin credential is backup/schema-recovery only and must not perform employee seed data inserts.

## Explicit no-seed policy remains

Do not add rows to:

```text
Staff/technician TblEmployee rows
TblServiceCategory
TblService
TblProduct
TblCustomer*
TblGiftCard*
TblInvoice*
TblOutputInfo*
TblOutputInfoTam*
terminal transaction tables
booking/payment/queue/turn/payroll history
```

## WPF build label

Because this prompt changes WPF/source, set:

```text
Build label: prompt029
Window title: OBM InstallationV0 Phase 1/2 - prompt029
```

Focused tests must prove prompt028 is absent from the active title/header/build-info path.

## Testing requirements

Build and test:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Required focused proof:

- exact non-staff EmployeeType mapping;
- Owner/Admin/SubAdmin management UI visibility;
- no checkout visibility;
- no Staff rows inserted;
- deterministic employee GUIDs;
- exact permission mapping;
- safe PIN handling;
- one transaction owns employee + outbox + v002 marker;
- marker last;
- rollback on failure;
- v001 marker preserved;
- same-version v002 replay zero delta;
- runtime/business tables unchanged;
- startup marker hydration displays correct status.

## Physical test

After backup and build/tests pass:

1. Apply v002 to `obm_pos_dev_v0_pg` using runtime role `hung`.
2. Record before/after counts for `TblEmployee`, `TblLocalOutbox`, and marker table.
3. Verify exactly the canonical non-staff management set is present.
4. Open management UI and verify Owner/Admin/SubAdmin rows appear.
5. Open checkout employee selection and verify those rows do not appear.
6. Run v002 again and prove zero delta.
7. Restart WPF and prove marker status hydrates as `v002 Complete`.

## Report 029

Create and push:

```text
report/report029.md
```

Required sections:

1. Verdict.
2. Exact operator policy change.
3. Reference DB read-only proof.
4. Exact `TblEmployee` schema/dependencies.
5. EmployeeType/permission mapping.
6. PIN/login safety decision.
7. Canonical non-staff employee rows and deterministic IDs.
8. Outbox mapping.
9. New pre-v002 rollback anchor path/hashes.
10. One-transaction/marker-last proof.
11. Exact files changed.
12. Build/test counts.
13. Physical before/after counts.
14. Management UI visibility proof.
15. Checkout exclusion proof.
16. Replay zero-delta proof.
17. Startup marker hydration proof.
18. Explicit no-seed/runtime zero-delta proof.
19. No reference mutation/no secret leakage/no source push.
20. Coordination commit SHA.

## Valid verdicts

Full physical v002 success:

```text
PHASE2_REFERENCE_DRIVEN_V002_NONSTAFF_EMPLOYEES_PASS
```

Implementation ready, operator UI test pending:

```text
PHASE2_REFERENCE_DRIVEN_V002_NONSTAFF_EMPLOYEES_READY_FOR_USER_TEST
```

PIN policy blocked:

```text
BLOCKED_PHASE2_V002_NONSTAFF_PIN_POLICY
```

Implementation blocked:

```text
BLOCKED_PHASE2_V002_NONSTAFF_EMPLOYEE_IMPLEMENTATION
```
