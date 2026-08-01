# Report 025 — Dedicated POS1 pgpass rollback-anchor attempt

## 1. Verdict

`BLOCKED_PHASE2_POS1_BACKUP_PRIVILEGE`

The dedicated POS1 pgpass matched the required database/user and fixed the previous "no password supplied" class of blocker. The workflow then stopped safely at the backup privilege gate because `pg_dump` could not dump schema `dbo` as role `hung`.

No physical Phase 2 seed, replay, WPF launch, restore, or database mutation was performed.

## 2. Dedicated pgpass presence/process-local proof without content disclosure

Dedicated credential path class used:

```text
LOCALAPPDATA\OBM\Phase2Pos1Trial\pgpass-pos1.conf
```

Safe proof:

```text
file exists: true
PGPASSFILE set process-local: true
file content read: false
file content printed: false
file copied/hashed/committed: false
```

The earlier `Phase2SeedAudit\pgpass.conf` path was not used for this task.

## 3. Exact target/current user proof

Sanitized non-mutating proof:

```text
current_database = obm_pos_dev_v0_pg
current_user = hung
```

Additional read-only privilege proof:

```text
has_schema_privilege(current_user, 'dbo', 'USAGE') = false
```

This confirms the dedicated pgpass authenticates the required role, but that role lacks the required schema privilege for backup.

## 4. V003-or-later rollback-anchor path

Chosen rollback-anchor path:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV003\PreSeedBackup
```

V003 was selected because V001 and V002 already existed.

## 5. `pg_dump` exit code and file size

`pg_dump` attempted against exactly:

```text
host: 127.0.0.1
port: 5432
database: obm_pos_dev_v0_pg
user: hung
format: custom
no-owner: true
no-privileges: true
```

Sanitized result:

```text
pg_dump: failed
safe failure class: permission denied for schema dbo
dump file exists: true
dump file size: 0 bytes
```

Because the dump was zero bytes and invalid, the rollback anchor is not valid.

## 6. `pg_restore --list` validation

Not reached.

Reason:

```text
pg_dump failed before a valid custom-format archive was produced.
```

## 7. Anchor filenames and SHA-256

V003 files present:

```text
obm_pos_dev_v0_pg.preseed.dump    0 bytes
pg_dump.stderr.txt                0 bytes
pre-seed-table-counts.tsv         1180 bytes
```

Required files not produced:

```text
pg_restore-list.txt
sanitized-database-metadata.txt
RESTORE-NOTES.md
SHA256SUMS.txt
```

SHA-256 manifest was not created because the required backup archive was invalid.

## 8. Confirmation V001/V002 preserved

Preserved anchors:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV001
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV002
```

Neither V001 nor V002 was overwritten or deleted.

## 9. Phase 1 revalidation proof

Not reached.

Prompt025 requires a valid rollback anchor before the physical Phase 2 seed. Since the backup privilege gate failed, Phase 1 bootstrap credential unprotect and API revalidation were not executed.

No Pairing Code was redeemed. No token or protected credential was printed.

## 10. Physical first-run before/after counts

Not executed.

Reason:

```text
BLOCKED_PHASE2_POS1_BACKUP_PRIVILEGE
```

Pre-seed discovery was attempted, but the role's schema privilege prevented reliable backup completion. The discovered selected-table count file shows the selected tables hidden/missing to the backup role, consistent with missing `dbo` usage privilege.

## 11. Exact baseline/outbox/marker deltas

Not executed.

Expected v001 plan remains:

```text
baseline data rows: 23
matching TblLocalOutbox rows: 22
completion marker: 1
```

Actual physical deltas:

```text
baseline delta: not run
outbox delta: not run
marker delta: not run
```

## 12. Runtime/excluded-table zero-delta proof

No runtime or excluded table was mutated because the seed did not run.

Physical before/after delta proof was not collected after the failed backup gate.

## 13. One-transaction/rollback proof

Source/test evidence retained from prompt023:

```text
focused InstallationV0 tests: 37 passed, 0 failed, 0 skipped
```

Physical transaction proof was not executed because mutation was blocked before Phase 2 execution.

No restore was attempted because no mutation occurred.

## 14. Same-version replay zero-delta proof

Not executed.

The first physical seed did not run, so same-version replay was not attempted.

## 15. Active WPF label and source-change status

Active WPF label remains:

```text
prompt023
```

Prompt025 made no WPF/POS/API source changes and did not change the build label.

The OBM source worktree remains shared/dirty from prior work, but no prompt025 source commit or push was made.

## 16. WPF operator handoff

No WPF operator handoff was performed.

The next safe action is to correct the backup role privilege/backup path for `hung` on `obm_pos_dev_v0_pg` without exposing secrets, then rerun the rollback-anchor gate before any Phase 2 mutation.

## 17. First missing table/row/default observed, or confirmation v001 is sufficient

No WPF runtime observation occurred because the database seed was blocked before execution.

First concrete missing requirement:

```text
role hung must be able to create the required custom-format pre-seed backup for schema dbo on obm_pos_dev_v0_pg.
```

## 18. No reference DB mutation/no secret leakage/no source push

Confirmed:

- no mutation to `obm_pos_dev_v0_pg`;
- no mutation to reference/protected/production databases;
- no fallback database/user was used;
- no PostgreSQL password, pgpass content, connection string, token, or protected credential was printed;
- no PostgreSQL roles/passwords/privileges were changed;
- no restore was attempted;
- no OBM source commit or push was made.

Only this coordination report is committed.

## 19. Coordination commit SHA

This report is intended to be committed as:

```text
report/report025.md
```

The exact coordination commit SHA is reported by Codex after commit and push because a Git commit cannot contain its own final hash without changing that hash.

