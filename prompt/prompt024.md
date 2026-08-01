# Prompt 024 — Complete rollback anchor and run the physical POS1 Phase 2 trial

## Current state

Read completely:

```text
report/report023.md
prompt/prompt023.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Prompt023 completed implementation and focused verification:

```text
InstallationV0 build: PASS
NailSalonNet8 build: PASS
Focused InstallationV0 tests: 37/37 PASS
Visible label: prompt023
Approved target: obm_pos_dev_v0_pg
Physical DB mutation: not run
Safety stop: BLOCKED_PHASE2_POS1_ROLLBACK_ANCHOR
```

The only current blocker is the missing successful rollback anchor:

```text
custom-format pg_dump
non-empty dump
validated archive
SHA256SUMS.txt
```

Do not redesign the seed manifest, table list, outbox mapping, transaction boundary, or runtime logic in this prompt unless a concrete physical defect is discovered.

## Objective

1. Diagnose and correct the rollback-anchor creation path.
2. Create a valid versioned pre-seed backup for `obm_pos_dev_v0_pg`.
3. Execute the already-implemented `phase2-pos1-legacy-reuse-trial-v001` against that exact database.
4. Prove data rows, matching outbox rows, and completion marker commit atomically.
5. Run the same version a second time and prove zero delta.
6. Leave WPF ready for operator runtime testing and report the first missing table/default only after a real physical run.

## Hard database boundary

The only writable database is:

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

Do not infer or silently switch targets.

## Preserve failed V001 anchor

Do not overwrite or delete:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV001\
```

Create the next available version, expected:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV002\PreSeedBackup\
```

If V002 exists, use V003, then V004, etc.

Maintain prior artifacts for audit/rollback.

## Diagnose `pg_dump` safely

Resolve PostgreSQL tools in this order:

```powershell
(Get-Command pg_dump -ErrorAction SilentlyContinue).Source
C:\Program Files\PostgreSQL\18\bin\pg_dump.exe
```

Resolve `pg_restore` similarly:

```powershell
(Get-Command pg_restore -ErrorAction SilentlyContinue).Source
C:\Program Files\PostgreSQL\18\bin\pg_restore.exe
```

Do not install tools or alter system PATH.

Use only the already-approved local credential path. A fixed local pgpass may be used process-locally without reading its content:

```powershell
$pgpassPath = Join-Path $env:LOCALAPPDATA "OBM\Phase2SeedAudit\pgpass.conf"
```

Allowed checks:

```text
path resolved
file exists
PGPASSFILE set process-local
```

Forbidden:

```text
Get-Content/type/cat/more on pgpass
printing password or connection string
copying/committing credential file
fallback to postgres/admin
changing database role/password/privilege
```

If no approved credential path exists, stop with:

```text
BLOCKED_PHASE2_POS1_BACKUP_CREDENTIAL_PATH
```

Report only sanitized failure classification, not secret/path contents.

## Required rollback anchor

Create a custom-format dump:

```text
obm_pos_dev_v0_pg.preseed.dump
```

Use:

```text
pg_dump
--format=custom
--no-owner
--no-privileges
--host=127.0.0.1
--port=5432
--username=hung
--dbname=obm_pos_dev_v0_pg
```

Do not include password on the command line.

Required validation before any seed mutation:

1. `pg_dump` exit code = 0.
2. Dump file exists and length > 0.
3. `pg_restore --list <dump>` exits 0 and returns a non-empty archive listing.
4. Create sanitized database metadata.
5. Create exact pre-seed counts for selected baseline, outbox, marker, and excluded runtime tables.
6. Create `RESTORE-NOTES.md` with exact sanitized restore procedure and explicit warning not to restore over another DB accidentally.
7. Create `SHA256SUMS.txt` covering every anchor file except itself.
8. Recompute and verify hashes.

Do not continue to mutation unless all eight checks pass.

## Seed implementation to execute

Use the existing prompt023 implementation:

```text
phase2-pos1-legacy-reuse-trial-v001
```

Selected baseline data rows:

```text
TblTenant: 1
TblPosLocal: 1
TblSetupLoginMethod: 3
TblSetupPaymentMethod: 6
TblEmployeePermission: 3
TblSetupWeird: 1
TblSetupServicesMethod: 1
TblParameterSetting: 2
TblSetupPrinter: 5
completion marker: 1
```

Expected matching outbox plan from report023:

```text
22 TblLocalOutbox rows on a fresh insert
```

Do not change these counts or rows in prompt024 unless a physical constraint proves the current implementation invalid. Any correction must be minimal, documented, tested, and versioned.

## Atomic transaction requirement

Use one PostgreSQL transaction and one final commit:

```text
BEGIN
  revalidate Phase 1 authorization/identity
  verify exact Development target obm_pos_dev_v0_pg
  acquire tenant/POS/version advisory transaction lock
  verify eligible pre-seed state
  insert/verify parent identity rows
  insert/verify selected default/config rows
  insert matching TblLocalOutbox rows using the proven legacy policy
  verify selected data rows
  verify outbox mapping
  verify excluded runtime/user tables unchanged
  write completion marker last
  read back marker and invariants
COMMIT
```

All `SaveChangesAsync` calls must use the same DbContext, connection, and transaction.

Any failure:

```text
ROLLBACK all data rows
ROLLBACK all outbox rows
ROLLBACK marker
Phase 1 artifacts unchanged
```

Do not automatically restore the dump after a normal transaction rollback. Preserve it as the operator recovery anchor. Restore only if a concrete non-transactional corruption is proven and the report explains why.

## Physical evidence — first run

Capture before and after counts for:

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
Phase 2 marker table
```

Also prove zero seed delta for:

```text
TblEmployee
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
booking/appointment tables
queue/turn/payroll runtime history
event delivery/log operational tables
```

Do not expose private business rows; counts and safe stable keys only.

## Physical evidence — same-version replay

Run the exact same v001 installation action a second time.

Required second-run deltas:

```text
baseline data rows: 0
TblLocalOutbox rows: 0
completion marker rows: 0
excluded/runtime rows: 0
```

The second run must return an explicit idempotent-success result, not silently insert duplicates.

## Phase 1 prerequisite

Before physical Phase 2 execution:

```text
read ApiAuthorized checkpoint
DPAPI-unprotect WpfJwt in memory
call protected hello
call /bootstrap/me
verify Tenant/POS/Attempt/LocalInstallation identity
```

Do not redeem another Pairing Code.

If the bootstrap credential is expired, fail closed and report the exact safe result. Do not delete or overwrite Phase 1 artifacts.

## Source-change and label rule

Prefer no source changes: prompt023 implementation is already build/test ready.

If no WPF source changes are necessary, active label remains:

```text
prompt023
```

If a real source correction is required, update the canonical label to:

```text
prompt024
```

Report which case applies. Never change the label merely because a new coordination prompt exists.

## Build/test rule

If source is unchanged, do not repeat broad builds unnecessarily; rerun the focused InstallationV0 tests needed to prove the executable path used for the physical run.

If source changes, run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

## WPF handoff

After physical DB first-run and replay pass:

- do not leave multiple WPF processes running;
- leave the latest WPF ready for Visual Studio Debug/operator launch;
- report the exact visible build label;
- report Phase 2 status and target DB classification;
- operator will test startup and screens to identify the first missing default/table, if any.

Do not automatically create v002 seed content before the operator observes a real missing requirement.

## Git safety

Do not run:

```text
git add .
git add -A
git reset
git clean
git stash
git checkout/restore unrelated files
```

Do not push OBM source. Commit only:

```text
report/report024.md
```

## Report 024 — 100% detail

Create:

```text
report/report024.md
```

Required sections:

1. Verdict.
2. Prompt023 state reused.
3. Exact pg_dump failure root cause from V001, sanitized.
4. New versioned rollback-anchor path.
5. pg_dump executable/path classification.
6. Credential-path presence proof without content disclosure.
7. Dump exit code, size, pg_restore-list validation.
8. Anchor filenames and SHA-256.
9. Exact target/environment proof.
10. Phase 1 prerequisite proof.
11. Source changes or confirmation none.
12. Active WPF build label.
13. First-run before/after counts.
14. Exact baseline and outbox deltas.
15. Completion-marker-last proof.
16. Excluded/runtime zero-delta proof.
17. Same-version replay zero-delta proof.
18. Transaction/rollback proof.
19. WPF operator handoff.
20. First missing requirement, only if physically observed.
21. No reference DB mutation/no secret leakage/no source push.
22. Coordination commit SHA.

## Valid verdicts

Full physical seed and replay passed:

```text
PHASE2_LEGACY_REUSE_TRIAL_V001_POS1_DB_PASS_READY_FOR_WPF_TEST
```

Backup credential path blocked:

```text
BLOCKED_PHASE2_POS1_BACKUP_CREDENTIAL_PATH
```

Rollback anchor still invalid:

```text
BLOCKED_PHASE2_POS1_ROLLBACK_ANCHOR
```

Phase 1 credential expired/invalid:

```text
BLOCKED_PHASE2_PHASE1_PREREQUISITE
```

Physical seed defect:

```text
BLOCKED_PHASE2_POS1_PHYSICAL_SEED
```
