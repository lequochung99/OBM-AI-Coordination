# Prompt 030 — Seed all reference employees as a versioned Phase 2 v002 upgrade

## Operator correction

Prompt029 is superseded and must not be executed.

The operator clarified that, for the current POS1 trial lane, it is faster to seed **all employee rows** from the complete reference database and rename/edit them later in the WPF management UI than to seed only Owner/Admin/SubAdmin.

This is an approved trial-and-error decision for the current POS1 Development target.

The intent is:

```text
copy all useful employee starter rows
-> preserve EmployeeType and UI classification
-> replace Tenant identity
-> remap employee GUIDs/dependencies
-> remove private/credential/history fields
-> write matching TblLocalOutbox rows
-> open WPF management UI
-> rename/edit/deactivate employees as needed
```

Do not interpret this as approval to copy employee private data, PINs, payroll history, or salon secrets.

## Important packaging distinction

This v002 is a **tenant-specific/reference-driven trial profile** for the current POS1 Development lane.

Do not silently freeze real employee names from `enailsalon_phasee1_pos1_pg` into a universal baseline for every future tenant.

For this trial, local display names may be copied so the operator can rename them quickly. Public reports must not print those names. Before a generic production installer is finalized, employee bootstrap must be classified explicitly as one of:

```text
optional tenant bootstrap/import
sanitized generic placeholders
operator-provided template
```

Prompt030 does not make that final packaging decision.

## Authoritative state

Read completely:

```text
report/report026.md
report/report028.md
prompt/prompt028.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
E:\Project2026\RecoveryReports\InstallationV0\Phase2SeedAuditV003\
```

Current accepted state:

```text
v001 physical seed on obm_pos_dev_v0_pg: PASS
v001 rows verified: 23
v001 TblLocalOutbox delta: 21
v001 marker written last: true
active source label before this task: prompt028
```

Reference database:

```text
enailsalon_phasee1_pos1_pg
```

Approved target database:

```text
obm_pos_dev_v0_pg
```

## New version

Create a new immutable upgrade version:

```text
phase2-reference-driven-trial-v002-employees
```

Source folder:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\reference-driven-v002-employees\
```

Do not overwrite v001. Preserve all prior trial folders and maintain the current/latest pointer.

## Backup before v002 mutation

The V007 backup predates the v001 physical seed and is not the correct rollback anchor for this upgrade.

Before any v002 mutation, create the next unused versioned backup of the current post-v001 target state, for example:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV008\PreV002Backup\
```

If V008 exists, use V009/V010/etc.

Required:

```text
custom-format pg_dump
pg_restore --list validation
pre-v002 selected/runtime table counts
sanitized metadata
RESTORE-NOTES.md
SHA256SUMS.txt
```

Use the approved admin backup credential only for backup/schema inspection. Use role `hung` for the actual v002 data transaction.

## Reference read-only rule

Read `enailsalon_phasee1_pos1_pg` only with:

```text
BEGIN TRANSACTION READ ONLY
SELECT-only queries
ROLLBACK
```

No reference mutation, no sequence mutation, no privilege changes.

Do not export employee private data into public artifacts.

## Employee scope

Select **all `TblEmployee` rows** from the reference database that are valid employee starter rows, including:

```text
Staff
Owner/Admin/SubAdmin-style non-staff
Manager/non-staff rows
other current EmployeeType rows that are legitimate UI employee records
```

Preserve the reference EmployeeType classification so existing UI filtering continues to work:

```text
management UI -> non-staff/management rows
checkout/queue/service staff UI -> staff rows only
```

Do not change the existing filtering logic unless a physical test proves a bug.

## TblEmployee column transformation policy

For every employee column, classify and report one of:

```text
preserve local trial display value
replace from Phase 1 identity
generate deterministic target GUID
remap FK
clear private/security value
derive at execution time
exclude from v002
```

### Preserve for local trial

Preserve only fields useful for quickly editing employees in WPF, when present and non-sensitive:

```text
display/name fields
EmployeeType
active/deleted status needed by current UI
sort/display order
management/checkout visibility flags
basic UI color or harmless presentation fields
safe commission/service flags only when they are true defaults and not payroll history
```

Names may exist in the local target trial DB, but must not appear in the public report.

### Replace/remap

```text
TenantGuid <- Phase 1 TenantGuid
EmployeeGuid <- deterministic from v002 + TenantGuid + reference stable employee identity
Permission/role FK <- remap to target TblEmployeePermission row
other included employee-parent FK <- remap to target deterministic parent
CreatedAt/UpdatedAt <- derive at execution time
```

Do not copy reference TenantGuid, row GUID, POS identity, or source database identity values as target identities.

### Clear/exclude private or security fields

Do not copy into the target trial seed:

```text
plaintext PIN/password
password hash or login credential from the real salon
phone/email/address/private notes
SSN/tax/payroll/bank fields
biometric/device credentials
terminal/API credentials
private image/file paths tied to the old machine
personal schedule/history if not required for UI startup
clock-in/timecard/payroll history
sales/commission history
```

If a PIN/credential column is non-null or required, inspect the existing safe WPF seed/bootstrap behavior and implement a **non-secret reset/unconfigured state**, not a copied real credential.

Do not hard-code one shared known PIN in source. Existing Phase 1/owner bootstrap access must remain available so the operator can configure employee PINs later.

## Employee dependency-first audit

Before inserting employees, inspect actual FK metadata and current entity configuration.

At minimum determine:

```text
TblEmployeePermission -> TblEmployee
TblTenant -> TblEmployee
```

Also inspect possible child/dependent tables such as employee working hours, capability/setup, pictures, service mappings, and queue/turn/payroll tables.

Classify each employee-related table:

```text
A. required parent for TblEmployee
B. required child/default for management UI startup
C. dependent on service/catalog not yet seeded -> defer
D. runtime/history/private -> exclude
E. optional -> add only after physical WPF test proves needed
```

Hard exclusions for v002 unless a non-null FK makes a minimal safe row mandatory:

```text
employee-service mappings when TblService is not seeded
turn/queue runtime state
clock/timecard rows
payroll runs/paychecks/history
sales/commission history
private contact/payroll detail tables
```

If management UI requires a safe default child row, include it after TblEmployee in the same transaction and document why.

## Existing target behavior

The target may already contain employee rows from manual testing or prior work.

Use stable-key behavior:

```text
absent employee stable key -> insert transformed row
compatible existing employee -> adopt/verify
same stable key but incompatible identity/type -> rollback PHASE2_EMPLOYEE_BASELINE_CONFLICT
extra target employees outside v002 -> preserve, do not delete
```

Do not truncate or replace all employee rows blindly.

## Outbox policy

Reuse the proven legacy employee save/outbox behavior.

For each employee row actually inserted or materially updated by v002:

```text
insert matching TblLocalOutbox row
TenantGuid = Phase 1 TenantGuid
SourceClientId = Phase 1 POS source identity
Entity/table = canonical TblEmployee name
Operation = I or U
Payload = deterministic sanitized employee payload
```

Do not include PIN/password/private contact/payroll fields in outbox payload.

No outbox for no-op compatible adoption when a deterministic v002 employee outbox event already exists.

If an adopted employee row is compatible but missing the required deterministic v002 outbox event, add that event in the same transaction.

All employee rows, included child/default rows, matching outbox rows, and v002 marker must share one PostgreSQL transaction.

## v002 marker and startup hydration

Do not overwrite the v001 marker.

Use a new immutable marker version:

```text
phase2-reference-driven-trial-v002-employees
```

Marker must be written last after readback verification.

Fix startup status hydration so WPF reads markers from `dbo."Phase2TrialCompletionMarker"` after Phase 1 resume.

Expected UI states:

```text
v001 complete, v002 not present -> Phase 2 v001 Complete; Employee v002 Upgrade Available
v002 marker present and verified -> Phase 2 v002 Complete
marker absent -> Not Started
marker/data conflict -> Blocked with safe result code
```

The UI must no longer reset to `Not Started` merely because the process restarted.

## One transaction

Use one target connection/transaction as role `hung`:

```text
BEGIN
  revalidate Phase 1 protected identity
  verify target = obm_pos_dev_v0_pg Development
  verify new pre-v002 backup anchor
  acquire advisory transaction lock for tenant/POS/v002
  verify v001/current marker state
  read reference employees via separate read-only reference connection
  verify/remap parent permissions
  insert/adopt TblEmployee rows
  insert only required safe employee child/default rows
  insert deterministic matching TblLocalOutbox rows
  verify management/staff classification counts
  verify excluded runtime/history tables unchanged
  write v002 marker last
  read back invariants
COMMIT
```

Any failure:

```text
ROLLBACK employees
ROLLBACK included child/default rows
ROLLBACK outbox
ROLLBACK v002 marker
preserve v001 data/marker
preserve Phase 1 checkpoint
```

## Physical UI acceptance

After physical v002 seed, launch the normal WPF/POS UI and verify:

```text
management employee UI lists all seeded employee rows
non-staff/management employees appear in management screens
staff employees appear in checkout/staff selection paths
non-staff employees do not appear in checkout staff list
employee names can be edited successfully
active/inactive behavior follows existing UI logic
no PIN/private credential was copied from reference DB
```

Do not seed services merely to satisfy employee-service mappings. Record a missing service dependency as a later v003 item.

## Same-version replay

Run or expose a second execution of v002 and prove:

```text
TblEmployee delta = 0
included employee-child/default delta = 0
TblLocalOutbox delta = 0
v002 marker delta = 0
runtime/history table delta = 0
```

## WPF label

Because this prompt changes WPF/source, set:

```text
Build label: prompt030
Window title: OBM InstallationV0 Phase 1/2 - prompt030
```

Focused tests must prove prompt028/prompt029 are absent from the active title/header/build-info source.

## Build and tests

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Add focused tests for:

```text
all reference employee types selected
private fields excluded
TenantGuid/GUID/FK remap
permission parent before employee
staff/non-staff UI filtering remains intact
employee outbox mapping
one transaction and marker last
rollback
existing compatible adoption
conflict behavior
startup marker hydration
same-version zero delta
excluded employee history/runtime tables
prompt030 label
```

## Report

Create and push:

```text
report/report030.md
```

Report must include:

1. Verdict.
2. Confirmation prompt029 superseded.
3. Exact pre-v002 backup path and hashes.
4. Reference employee count and EmployeeType distribution, sanitized.
5. Employee column transformation matrix.
6. Private/security fields excluded.
7. Employee dependency/child-table classification.
8. Exact inserted/adopted/conflicting counts.
9. Employee outbox mapping and delta.
10. One-transaction/marker-last proof.
11. Startup marker hydration proof.
12. Management UI and checkout filtering physical proof.
13. Same-version replay zero-delta proof.
14. Excluded runtime/history table proof.
15. Exact source files changed.
16. Build/test commands and counts.
17. Prompt030 label proof.
18. No reference mutation/no secret leakage/no source push.
19. Remaining missing employee/default dependency for v003, if any.
20. Coordination commit SHA.

Do not print employee names, PINs, contacts, payroll values, or private row dumps in the public report.

## Valid verdicts

Physical v002 employee upgrade passed:

```text
PHASE2_REFERENCE_DRIVEN_V002_ALL_EMPLOYEES_POS1_DB_PASS_READY_FOR_WPF_TEST
```

Implementation ready, physical operator action pending:

```text
PHASE2_REFERENCE_DRIVEN_V002_ALL_EMPLOYEES_READY_FOR_USER_TEST
```

Blocked by employee schema/dependency conflict:

```text
BLOCKED_PHASE2_V002_EMPLOYEE_DEPENDENCY
```

Blocked implementation:

```text
BLOCKED_PHASE2_V002_ALL_EMPLOYEES_IMPLEMENTATION
```
