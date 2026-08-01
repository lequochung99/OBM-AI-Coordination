# Report 026 — POS1 runtime privilege alignment and physical seed gate

## 1. Verdict

`BLOCKED_PHASE2_POS1_PHYSICAL_SEED`

Prompt026 successfully corrected the proven runtime privilege blocker for the approved local Development target and created a valid rollback anchor. The physical Phase 2 seed/replay was not executed because the current prompt023 source contains only the in-memory executor and PostgreSQL placeholder script builder, not an executable PostgreSQL Phase 2 operator action.

No seed rows, outbox rows, or completion marker rows were inserted.

## 2. Report025 root-cause confirmation

Report025 root cause was confirmed:

```text
current_database = obm_pos_dev_v0_pg
current_user = hung
has_schema_privilege(hung, dbo, USAGE) = false
```

Prompt026 corrected this for the approved target only. Post-correction runtime proof:

```text
current_database = obm_pos_dev_v0_pg
current_user = hung
has_schema_privilege(hung, dbo, USAGE) = true
```

## 3. Admin backup credential presence proof without secret disclosure

Admin backup credential path class:

```text
LOCALAPPDATA\OBM\Phase2Pos1Trial\pgpass-backup-admin.conf
```

Safe proof:

```text
file exists: true
PGPASSFILE set process-local: true
admin connection user: postgres
admin current_database: obm_pos_dev_v0_pg
admin dbo usage: true
credential content read: false
credential content printed: false
```

Runtime credential path class:

```text
LOCALAPPDATA\OBM\Phase2Pos1Trial\pgpass-pos1.conf
```

No pgpass content was read, printed, hashed, copied, or committed.

## 4. Exact versioned rollback-anchor path/files/hashes

Valid rollback anchor:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV007\PreSeedBackup
```

Earlier partial retry folders V004, V005, and V006 were preserved and not overwritten. V007 is the first complete valid prompt026 anchor.

V007 files:

```text
GRANT-RUNTIME-PRIVILEGES.sql
obm_pos_dev_v0_pg.preseed.dump
pg_dump.stderr.txt
pg_restore-list.stderr.txt
pg_restore-list.txt
pre-seed-table-counts.tsv
RESTORE-NOTES.md
REVOKE-RUNTIME-PRIVILEGES.sql
sanitized-database-metadata.txt
sanitized-prechange-inspection.tsv
SHA256SUMS.txt
```

Anchor SHA-256 manifest exists and was verified. Key entries:

```text
obm_pos_dev_v0_pg.preseed.dump  54E18F6C4DFE66D0F404D9A0FF984DCCCDD4A1B1C5FA01FFEC199AD8AD089684
pg_restore-list.txt             52460058A85AB1480FD91525D614562226572604AADCD197F1C8173EB82C37EA
pre-seed-table-counts.tsv        1F322B2F3CFB83C4700BF5C63C0615018D55EDBDFDF7741B4F16FD64A0BBE3A3
```

Additional post-grant proof file:

```text
hung-runtime-privilege-proof.tsv
```

This proof was written after the anchor manifest and is not part of the pre-change backup hash set.

## 5. pg_dump/pg_restore validation

Admin backup validation:

```text
pg_dump exit code: 0
dump size: 359224 bytes
pg_restore --list exit code: 0
archive listing size: 97722 bytes
```

The dump was created with:

```text
--format=custom
--no-owner
--no-privileges
database: obm_pos_dev_v0_pg
admin user: postgres
```

## 6. Database/schema/table ownership summary

Sanitized ownership summary:

```text
database owner: postgres
schema dbo owner: postgres
dbo table owners: postgres
dbo sequence owners: postgres
```

Pre-change inspection proved the marker table was initially missing:

```text
marker table exists before alignment: false
```

## 7. Exact privileges granted to hung and forbidden privileges not granted

Granted only on approved local Development target:

```sql
GRANT CONNECT ON DATABASE obm_pos_dev_v0_pg TO hung;
GRANT USAGE ON SCHEMA dbo TO hung;
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA dbo TO hung;
GRANT USAGE, SELECT, UPDATE ON ALL SEQUENCES IN SCHEMA dbo TO hung;
```

Forbidden privileges not granted:

```text
SUPERUSER: not granted
ownership transfer: not performed
CREATEDB: not granted
CREATEROLE: not granted
BYPASSRLS: not granted
REPLICATION: not granted
schema CREATE: not granted to hung
GRANT ALL broadly: not used
password changes: none
other database changes: none
```

Recovery-only script produced:

```text
REVOKE-RUNTIME-PRIVILEGES.sql
```

The revoke script was not executed.

## 8. Marker schema handling

The Phase 2 marker table was missing before alignment, and prompt026 allowed admin/schema creation if required by the prompt023 source contract.

Created approved marker object:

```text
dbo."Phase2TrialCompletionMarker"
```

Columns:

```text
MarkerGuid uuid primary key
TenantGuid uuid not null
PosGuid uuid not null
Version text not null unique
Status text not null
CreatedAtUtc timestamptz not null default now()
```

During the DDL step, an incorrectly lower-cased marker table created by command quoting was immediately removed before any data was inserted, then the correct mixed-case marker object was created and granted to `hung`.

Marker row count after alignment:

```text
0
```

## 9. Independent hung read/DML/sequence/temporary-backup proof

Runtime role proof after grants:

```text
current_database = obm_pos_dev_v0_pg
current_user = hung
dbo usage = true
selected baseline table SELECT/INSERT/UPDATE/DELETE = true
dbo sequence USAGE = true for 2/2 visible sequences
```

Selected verified tables:

```text
Phase2TrialCompletionMarker
TblEmployeePermission
TblLocalOutbox
TblParameterSetting
TblPosLocal
TblSetupLoginMethod
TblSetupPaymentMethod
TblSetupPrinter
TblSetupServicesMethod
TblSetupWeird
TblTenant
```

Temporary runtime-role backup validation:

```text
pg_dump --schema=dbo as hung exit code: 0
dump size: 354294 bytes
pg_restore --list exit code: 0
listing size: 96722 bytes
temporary dump deleted: true
```

A whole-database runtime-role dump encountered an out-of-scope table privilege outside the approved `dbo` alignment. No broader schema privilege was granted because prompt026 approved runtime alignment on the existing `dbo` schema only.

## 10. Phase 1 revalidation proof

Not executed.

Reason: physical seed execution was blocked before Phase 1 revalidation because no executable PostgreSQL Phase 2 seed path exists in the current source. No Pairing Code was redeemed, and no bootstrap token or protected credential was read or printed.

## 11. First physical seed before/after counts

Physical seed was not executed.

Sanitized post-alignment counts before any seed:

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

No seed after-count exists because the executable physical seed action is missing.

## 12. Exact baseline/outbox/marker deltas

Physical seed deltas:

```text
baseline data delta: not run
TblLocalOutbox delta: not run
completion marker delta: not run
```

Schema/privilege alignment delta:

```text
Phase2TrialCompletionMarker table: created
Phase2TrialCompletionMarker rows: 0
runtime grants for hung on dbo: applied
```

## 13. Runtime/excluded-table zero-delta proof

Because no physical seed ran, no selected, runtime, business, or excluded table rows were inserted by Phase 2.

Post-alignment sampled excluded/runtime counts remained zero for:

```text
TblEmployee
TblServiceCategory
TblService
TblProduct
TblInvoice
TblInvoiceBookingLink
TblOutputInfo
TblOutputInfoTam
```

Existing non-seed runtime state was not altered.

## 14. Same-version replay zero-delta proof

Not executed.

Reason: first physical seed did not run because the source lacks an executable PostgreSQL Phase 2 path.

## 15. Transaction/rollback/marker-last proof

Source/test proof retained:

```text
focused InstallationV0 tests: 37 passed, 0 failed, 0 skipped
in-memory executor rollback proof: present
script builder marker-last proof: present
```

Physical transaction proof:

```text
not executed
```

Concrete blocker:

```text
InstallationV0 currently has InMemoryPhase2TrialTransactionExecutor and Phase2PostgreSqlTransactionScriptBuilder placeholder output, but no PostgreSQL executor/operator action that can execute the prompt023 plan against obm_pos_dev_v0_pg.
```

## 16. WPF handoff and active label

No WPF process was launched for Phase 2.

Active label remains:

```text
prompt023
```

Operator handoff:

```text
Do not attempt Phase 2 physical seed from WPF until a real PostgreSQL executor/operator action exists.
The WPF UI currently shows the Phase 2 action as disabled.
```

## 17. Source changes or confirmation none

No OBM source files were changed by prompt026.

The source worktree remains shared/dirty from prior prompt023 work and was not committed or pushed.

## 18. Build/test evidence

Focused test command:

```text
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Result:

```text
Passed: 37
Failed: 0
Skipped: 0
```

No broad rebuild was run because prompt026 made no source changes.

## 19. No reference DB mutation/no secret leakage/no source push

Confirmed:

- only approved database target touched: `obm_pos_dev_v0_pg`;
- no reference/protected/production database mutation;
- no seed data mutation;
- approved runtime grants applied only on `dbo` in the approved local Development database;
- approved marker schema object created with zero rows;
- no pgpass content read, printed, copied, hashed, or committed;
- no password, token, connection string, or protected credential printed;
- no source commit or source push;
- no restore attempted.

## 20. Coordination commit SHA

This report is intended to be committed as:

```text
report/report026.md
```

The exact coordination commit SHA is reported by Codex after commit and push because a Git commit cannot contain its own final hash without changing that hash.

