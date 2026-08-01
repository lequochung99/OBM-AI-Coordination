# Prompt 026 — Align POS1 runtime privileges, create rollback anchor, and execute physical Phase 2 trial

## Authoritative state

Read completely:

```text
report/report023.md
report/report024.md
report/report025.md
prompt/prompt023.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Current implementation state:

```text
Phase 2 source/build/tests ready
Active label: prompt023
Focused InstallationV0 tests: 37/37 PASS
Approved target: obm_pos_dev_v0_pg
Physical seed: never executed
```

Report025 proved:

```text
current_database = obm_pos_dev_v0_pg
current_user = hung
has_schema_privilege(hung, dbo, USAGE) = false
```

Therefore this is not only a backup issue. The intended WPF runtime role cannot reliably read/write the `dbo` schema and would also fail the physical seed/runtime path.

## Operator-approved correction

The operator approves a one-time, least-privilege alignment for the local Development database only.

Use two separate credential paths:

```text
Backup/admin credential:
LOCALAPPDATA\OBM\Phase2Pos1Trial\pgpass-backup-admin.conf

WPF runtime credential:
LOCALAPPDATA\OBM\Phase2Pos1Trial\pgpass-pos1.conf
```

The admin credential may be used only for:

1. creating and validating the pre-seed rollback dump;
2. inspecting ownership/privileges;
3. granting the approved runtime privileges to role `hung` on exactly `obm_pos_dev_v0_pg`;
4. creating the approved Phase 2 marker schema object through the canonical schema path if it is missing and the source implementation requires it.

The admin credential must never be used to execute the Phase 2 data seed or normal WPF runtime.

Do not read, print, copy, hash, or commit either pgpass file.

## Hard target boundary

Only this database may be changed:

```text
obm_pos_dev_v0_pg
```

Hard reject:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
recovery_api_day16_pg
any other database
production/reference/protected databases
```

Environment must be `Development`.

## Step 1 — Pre-change inspection

Using the temporary admin path, collect sanitized evidence for:

```text
database owner
schema dbo owner
current grants for hung
current table owners in dbo
current sequence owners in dbo
marker table existence
```

Do not report passwords, connection strings, or private row data.

Create a reversible privilege script in the versioned local evidence folder:

```text
GRANT-RUNTIME-PRIVILEGES.sql
REVOKE-RUNTIME-PRIVILEGES.sql
```

Do not commit these local files to the public coordination repository.

## Step 2 — Create rollback anchor before privilege/data mutation

Create the next unused versioned anchor, preserving V001/V002/V003:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV004\PreSeedBackup\
```

If V004 exists, use V005, V006, etc.

Use admin/owner credential for backup only:

```text
pg_dump --format=custom --no-owner --no-privileges
```

Required anchor artifacts:

```text
obm_pos_dev_v0_pg.preseed.dump
pg_restore-list.txt
pre-seed-table-counts.tsv
sanitized-database-metadata.txt
RESTORE-NOTES.md
GRANT-RUNTIME-PRIVILEGES.sql
REVOKE-RUNTIME-PRIVILEGES.sql
SHA256SUMS.txt
```

Validation requirements:

```text
pg_dump exit code = 0
dump size > 0
pg_restore --list exit code = 0
archive listing non-empty
all required files present
SHA-256 manifest valid
```

If the admin backup credential is absent or backup validation fails, stop before any privilege or data mutation:

```text
BLOCKED_PHASE2_POS1_ADMIN_BACKUP_CREDENTIAL
```

## Step 3 — Least-privilege runtime alignment for hung

After the valid rollback anchor exists, use the admin credential to grant only the privileges needed by the WPF runtime on the existing `dbo` schema:

```sql
GRANT CONNECT ON DATABASE obm_pos_dev_v0_pg TO hung;
GRANT USAGE ON SCHEMA dbo TO hung;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA dbo TO hung;
GRANT USAGE, SELECT, UPDATE ON ALL SEQUENCES IN SCHEMA dbo TO hung;
```

Rules:

- Do not grant SUPERUSER.
- Do not grant ownership.
- Do not grant CREATEDB, CREATEROLE, BYPASSRLS, or REPLICATION.
- Do not transfer database/schema/table ownership.
- Do not grant schema CREATE unless the approved marker object is missing and the canonical schema step explicitly requires an admin-created object.
- Do not use `GRANT ALL` broadly.
- Do not alter passwords.
- Do not change privileges on any other database/schema.

If the Phase 2 completion marker object is missing, create it with the admin/schema path required by the prompt023 source contract, then grant normal DML/read access on that marker table to `hung`. Do not let the data-seed transaction silently create unrelated schema.

Keep the runtime grants after the trial because `hung` is the approved WPF Development runtime role. The generated REVOKE script is a recovery artifact, not an automatic cleanup action.

## Step 4 — Verify hung independently

Switch to the runtime pgpass path and prove safely:

```text
current_database = obm_pos_dev_v0_pg
current_user = hung
has_schema_privilege(hung, dbo, USAGE) = true
SELECT privilege on selected baseline tables = true
INSERT/UPDATE/DELETE privilege on selected baseline tables = true
required sequence privilege = true
```

Also run `pg_dump` as `hung` to a temporary disposable path and validate it with `pg_restore --list`. Delete only the temporary validation dump after success.

This proves the same runtime role can both inspect/backup and perform the Phase 2 transaction.

If runtime privilege verification fails:

```text
BLOCKED_PHASE2_POS1_RUNTIME_PRIVILEGE
```

Do not run the seed.

## Step 5 — Preserve the approved seed design

Do not redesign source or change the selected trial plan.

Reuse prompt023 state:

```text
phase2-pos1-legacy-reuse-trial-v001
23 baseline data rows
22 matching TblLocalOutbox rows
1 completion marker
one shared PostgreSQL transaction
marker last
```

Selected order:

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
matching TblLocalOutbox rows
completion marker
```

Keep complete logical printer settings:

```text
CUSTOMER_TERMINAL
MERCHANT_TERMINAL
POS_CUSTOMER
POS_MERCHANT
POS_EMPLOYEE
```

Do not seed employees, services/categories/products, customers, gift cards, invoices, output tables, booking, terminal-payment runtime data, queue/turn/payroll history, or event-delivery history.

## Step 6 — Phase 1 revalidation

Before the physical data transaction:

```text
read existing ApiAuthorized checkpoint
DPAPI-unprotect bootstrap credential in memory
call protected hello endpoint
verify exact hello marker
call /bootstrap/me
verify Tenant/POS/Attempt/LocalInstallation identity
```

Do not redeem another Pairing Code.
Do not modify Phase 1 artifacts.

If Phase 1 credential is expired/invalid, stop with an explicit Phase 1 prerequisite result; do not delete the checkpoint.

## Step 7 — Physical first seed

Execute the existing prompt023 plan using the `hung` runtime credential and exactly one underlying PostgreSQL transaction:

```text
BEGIN
  acquire tenant/POS/version advisory transaction lock
  verify target/marker/current state
  insert-or-verify selected data rows in dependency order
  insert matching TblLocalOutbox rows in the same transaction
  verify baseline/outbox/excluded-table invariants
  write completion marker last
  read back stable keys/counts
COMMIT
```

Any failure must roll back:

```text
all baseline rows
all outbox rows
completion marker
```

Do not auto-restore after a normal transaction rollback. Restore is reserved for a proven non-transactional failure and requires explicit operator approval.

## Step 8 — Physical replay

Run the same version a second time.

Required second-run deltas:

```text
baseline data delta = 0
TblLocalOutbox delta = 0
completion marker delta = 0
runtime/excluded table delta = 0
```

Any difference is a blocker.

## Step 9 — WPF handoff

After seed and replay pass:

- leave WPF not running unless the established operator handoff requires launching it;
- provide Visual Studio Debug steps;
- active visible label remains `prompt023` unless a real source correction was necessary;
- show Phase 2 status and physical DB proof safely.

If no source change is required, do not change the label to prompt026.
If source must be corrected, change label to prompt026 and rerun focused builds/tests.

## Builds/tests

At minimum rerun:

```text
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Build source only if source changed or binaries must be refreshed.

## Report

Create:

```text
report/report026.md
```

Report must include:

1. Verdict.
2. Report025 root-cause confirmation.
3. Admin backup credential presence proof without secret disclosure.
4. Exact versioned rollback-anchor path/files/hashes.
5. pg_dump/pg_restore validation.
6. Database/schema/table ownership summary.
7. Exact privileges granted to hung and forbidden privileges not granted.
8. Marker schema handling.
9. Independent hung read/DML/sequence/temporary-backup proof.
10. Phase 1 revalidation proof.
11. First physical seed before/after counts.
12. Exact baseline/outbox/marker deltas.
13. Runtime/excluded-table zero-delta proof.
14. Same-version replay zero-delta proof.
15. Transaction/rollback/marker-last proof.
16. WPF handoff and active label.
17. Source changes or confirmation none.
18. Build/test evidence.
19. No reference DB mutation/no secret leakage/no source push.
20. Coordination commit SHA.

## Valid verdicts

Full physical pass:

```text
PHASE2_LEGACY_REUSE_TRIAL_V001_POS1_DB_PASS_READY_FOR_WPF_TEST
```

Admin backup credential missing/invalid:

```text
BLOCKED_PHASE2_POS1_ADMIN_BACKUP_CREDENTIAL
```

Runtime privilege alignment failed:

```text
BLOCKED_PHASE2_POS1_RUNTIME_PRIVILEGE
```

Phase 1 prerequisite invalid:

```text
BLOCKED_PHASE2_PHASE1_PREREQUISITE
```

Physical seed/replay failed:

```text
BLOCKED_PHASE2_POS1_PHYSICAL_SEED
```
