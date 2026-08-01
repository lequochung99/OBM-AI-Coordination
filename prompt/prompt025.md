# Prompt 025 — Use a dedicated POS1 pgpass entry, create rollback anchor V003, and run the physical seed

## Current state

Read completely:

```text
report/report023.md
report/report024.md
prompt/prompt023.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Current implementation state:

```text
InstallationV0 build: PASS
NailSalonNet8 build: PASS
Focused InstallationV0 tests: 37/37 PASS
Active WPF label: prompt023
Approved seed target: obm_pos_dev_v0_pg
Physical seed: not run
```

Prompt024 stopped safely with:

```text
BLOCKED_PHASE2_POS1_ROLLBACK_ANCHOR
```

The known credential-path file used for the earlier reference-database audit was created for a different database name. PostgreSQL pgpass matching includes the database field, so it must not be assumed to authorize `obm_pos_dev_v0_pg`.

Prompt025 uses a dedicated operator-created pgpass file for the POS1 development target. Do not read or print its contents.

## Dedicated pgpass path

Use exactly:

```powershell
$pos1Pgpass = Join-Path $env:LOCALAPPDATA "OBM\Phase2Pos1Trial\pgpass-pos1.conf"
```

Required safe checks:

```text
file exists = true
file content read = false
file content printed = false
```

Set process-local only:

```powershell
$env:PGPASSFILE = $pos1Pgpass
```

Do not use the earlier `Phase2SeedAudit\pgpass.conf` file for this task.

Do not:

- `Get-Content`, `type`, `cat`, `more`, hash, copy, or expose the pgpass file;
- print its full expanded path in the public report;
- ask for or print a password;
- alter PostgreSQL roles/passwords/privileges;
- fall back to `postgres` or administrator credentials.

If the file is absent or PostgreSQL clients still report `no password supplied`, stop with:

```text
BLOCKED_PHASE2_POS1_DEDICATED_PGPASS
```

## Approved backup connection

Use only:

```text
host: 127.0.0.1
port: 5432
database: obm_pos_dev_v0_pg
user: hung
```

Before creating the anchor, run a sanitized non-mutating proof:

```sql
SELECT current_database(), current_user;
```

Required result:

```text
current_database = obm_pos_dev_v0_pg
current_user = hung
```

Do not set `default_transaction_read_only=on` globally for the later WPF seed process. The pgpass is for PostgreSQL client backup/count commands only. The WPF physical seed must continue to use its canonical Development database configuration and transaction code.

## Target safety

Only this database may be mutated by the Phase 2 trial:

```text
obm_pos_dev_v0_pg
```

Hard reject:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
recovery_api_day16_pg
any database not exactly obm_pos_dev_v0_pg
```

No target inference or fallback.

## Create rollback anchor V003

Preserve V001 and V002.

Create:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV003\PreSeedBackup\
```

If V003 already exists, use the next unused version and report it. Never overwrite an existing anchor.

Required files:

```text
obm_pos_dev_v0_pg.preseed.dump
pg_restore-list.txt
pre-seed-table-counts.tsv
sanitized-database-metadata.txt
RESTORE-NOTES.md
SHA256SUMS.txt
```

Use PostgreSQL 18 tools already installed. Prefer PATH resolution, otherwise use:

```text
C:\Program Files\PostgreSQL\18\bin\pg_dump.exe
C:\Program Files\PostgreSQL\18\bin\pg_restore.exe
C:\Program Files\PostgreSQL\18\bin\psql.exe
```

### Backup requirements

Run `pg_dump` in custom format against exactly `obm_pos_dev_v0_pg` using role `hung` and the dedicated process-local `PGPASSFILE`.

Required proof:

```text
pg_dump exit code = 0
dump file exists = true
dump file size > 0
```

Validate:

```text
pg_restore --list <dump>
exit code = 0
listing file non-empty
```

Generate SHA-256 for all anchor files except `SHA256SUMS.txt` itself.

Do not include passwords, connection strings, tokens, or private row data in metadata/count files.

If the role can connect but lacks sufficient privileges to dump, stop with a sanitized exact privilege error classification. Do not change privileges.

## Reuse prompt023 implementation

Do not redesign the Phase 2 seed.

Reuse the current prompt023 implementation:

```text
phase2-pos1-legacy-reuse-trial-v001
23 baseline data rows
22 matching TblLocalOutbox rows
1 completion marker
one shared transaction
marker last
runtime/business tables excluded
```

Reuse the proven legacy seed/save mapping and selected method/table list from report023.

Active WPF label remains:

```text
prompt023
```

Do not change source or label unless a real implementation defect is found.

## Phase 1 prerequisite

Before physical Phase 2 execution:

```text
read ApiAuthorized checkpoint
DPAPI-unprotect WpfJwt in memory
call protected hello
call bootstrap/me
verify Tenant/POS/Attempt/LocalInstallation identity
```

Do not redeem another Pairing Code.
Do not print the token or protected credential.

## Physical first run

After the V003 anchor is valid, execute the existing Phase 2 operator action against exactly `obm_pos_dev_v0_pg`.

Collect sanitized before/after counts for:

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
Phase 2 completion marker
```

Also verify zero seed delta for:

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
queue/turn/payroll runtime/history tables
event delivery/log operational tables
```

Expected new trial plan when target is in the eligible pre-seed state:

```text
baseline data rows inserted/verified: 23
TblLocalOutbox rows created according to plan: 22
completion marker: 1
```

Use actual physical counts as source of truth. If existing compatible rows make some operations no-op, report exact safe deltas and whether eligibility/idempotency rules accepted them.

## Transaction proof

Prove physically that one transaction owns:

```text
baseline rows
matching TblLocalOutbox rows
completion marker
```

Any failure must leave:

```text
no partial baseline rows
no orphan outbox rows
no completion marker
Phase 1 checkpoint unchanged
```

Do not automatically restore the backup after a normal transaction rollback. Preserve the anchor for explicit recovery.

## Same-version replay

Run the same Phase 2 version a second time.

Required result:

```text
baseline delta = 0
outbox delta = 0
marker delta = 0
runtime/excluded delta = 0
```

If replay produces duplicates or conflict, stop and report the exact safe stable key/table. Do not manually patch rows.

## WPF handoff

After database seed/replay PASS:

- leave WPF closed unless runtime testing is required by the current implementation;
- provide exact Visual Studio Debug steps;
- operator should see `Build label: prompt023`;
- Phase 2 should show Complete for `phase2-pos1-legacy-reuse-trial-v001`;
- run normal POS startup and record the first missing table/row/default, if any.

That observation will drive trial v002 rather than modifying v001.

## Build/test rule

No source change:

- retain report023 build/test evidence;
- rerun the 37 focused tests before physical mutation when practical.

Source correction required:

- update label to `prompt025`;
- rebuild InstallationV0 and NailSalonNet8;
- rerun focused InstallationV0 tests;
- report exact source defect and changed files.

## Report

Create:

```text
report/report025.md
```

Required sections:

1. Verdict.
2. Dedicated pgpass presence/process-local proof without content disclosure.
3. Exact target/current user proof.
4. V003-or-later rollback-anchor path.
5. `pg_dump` exit code and file size.
6. `pg_restore --list` validation.
7. Anchor filenames and SHA-256.
8. Confirmation V001/V002 preserved.
9. Phase 1 revalidation proof.
10. Physical first-run before/after counts.
11. Exact baseline/outbox/marker deltas.
12. Runtime/excluded-table zero-delta proof.
13. One-transaction/rollback proof.
14. Same-version replay zero-delta proof.
15. Active WPF label and source-change status.
16. WPF operator handoff.
17. First missing table/row/default observed, or confirmation v001 is sufficient.
18. No reference DB mutation/no secret leakage/no source push.
19. Coordination commit SHA.

## Verdicts

Full physical PASS:

```text
PHASE2_LEGACY_REUSE_TRIAL_V001_POS1_DB_PASS_READY_FOR_WPF_TEST
```

Dedicated pgpass absent or unmatched:

```text
BLOCKED_PHASE2_POS1_DEDICATED_PGPASS
```

Backup privilege issue:

```text
BLOCKED_PHASE2_POS1_BACKUP_PRIVILEGE
```

Seed transaction issue after valid anchor:

```text
BLOCKED_PHASE2_POS1_PHYSICAL_SEED
```
