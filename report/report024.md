# Report 024 — Rollback anchor retry for physical POS1 Phase 2 trial

## 1. Verdict

`BLOCKED_PHASE2_POS1_ROLLBACK_ANCHOR`

The physical POS1 Phase 2 mutation was not executed. The retry reached the rollback-anchor gate and failed before creating a valid custom-format backup, so no database seed, replay, or WPF runtime handoff was attempted.

## 2. Prompt023 state reused

Prompt023 implementation state reused without source redesign:

```text
InstallationV0 build: PASS from report023
NailSalonNet8 build: PASS from report023
Focused InstallationV0 tests: 37/37 PASS from report023
Active label: prompt023
Approved target: obm_pos_dev_v0_pg
Trial version: phase2-pos1-legacy-reuse-trial-v001
Physical DB mutation: not previously run
```

Prompt024 did not change the seed manifest, table list, outbox mapping, transaction boundary, or runtime logic.

## 3. Exact pg_dump failure root cause from V001, sanitized

Preserved V001 anchor path:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV001\PreSeedBackup
```

V001 sanitized metadata contained:

```text
PG_DUMP_FAILED
```

V001 did not contain:

```text
custom-format pg_dump
non-empty dump file
pg_restore archive listing
SHA256SUMS.txt
```

Prompt024 diagnosis found the same class of credential handoff failure before dump creation: PostgreSQL client authentication did not receive a password from the approved process-local pgpass path for the required `127.0.0.1:5432 / hung / obm_pos_dev_v0_pg` connection. No pgpass content was read.

## 4. New versioned rollback-anchor path

New retry path:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2Pos1TrialV002\PreSeedBackup
```

V001 was preserved and not overwritten or deleted.

V002 files present:

```text
sanitized-database-metadata.txt
```

V002 sanitized metadata:

```text
PSQL_COUNTS_DISCOVERY_FAILED
```

## 5. pg_dump executable/path classification

PostgreSQL tool resolution:

```text
pg_dump: resolved
pg_restore: resolved
psql: resolved
```

Resolved PostgreSQL 18 tool location class:

```text
C:\Program Files\PostgreSQL\18\bin
```

No tools were installed and system PATH was not modified.

## 6. Credential-path presence proof without content disclosure

Approved local credential path class:

```text
LOCALAPPDATA\OBM\Phase2SeedAudit\pgpass.conf
```

Presence check:

```text
credential path exists: yes
PGPASSFILE set process-local: yes
credential content disclosed: no
```

Forbidden actions avoided:

```text
no Get-Content/type/cat/more on pgpass
no password printed
no connection string printed
no fallback to postgres/admin
no database role/password/privilege change
```

## 7. Dump exit code, size, pg_restore-list validation

The anchor workflow stopped during pre-seed count discovery before `pg_dump` could run successfully.

Sanitized PostgreSQL client result:

```text
psql count discovery: failed
safe error class: fe_sendauth / no password supplied
pg_dump exit code: not reached
dump file size: not available
pg_restore --list: not reached
archive listing: not available
```

Because the required dump and archive validation were not produced, mutation was blocked.

## 8. Anchor filenames and SHA-256

V002 filenames:

```text
sanitized-database-metadata.txt
```

Required but missing:

```text
obm_pos_dev_v0_pg.preseed.dump
pg_restore-list.txt
pre-seed-table-counts.tsv
RESTORE-NOTES.md
SHA256SUMS.txt
```

SHA-256 manifest: not created because required anchor files were not complete.

## 9. Exact target/environment proof

The intended target for the anchor and trial remained:

```text
database: obm_pos_dev_v0_pg
host: 127.0.0.1
port: 5432
user: hung
environment classification: Development
```

No alternate database target was inferred or used.

Hard-rejected targets remained untouched:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
recovery_api_day16_pg
production/reference/protected databases
```

## 10. Phase 1 prerequisite proof

Phase 1 credential revalidation was not reached because rollback-anchor validation is required before mutation.

No Pairing Code was redeemed. No bootstrap credential or token was printed. Phase 1 artifacts were not deleted, rotated, or overwritten.

## 11. Source changes or confirmation none

No WPF/POS/API source changes were made for prompt024.

Reason: the prompt023 implementation was already build/test ready, and the current blocker is the rollback-anchor credential handoff, not seed source behavior.

## 12. Active WPF build label

Active label remains:

```text
prompt023
```

This follows prompt024's label rule: do not change the source label unless a real source correction is required.

## 13. First-run before/after counts

Not executed.

Reason:

```text
BLOCKED_PHASE2_POS1_ROLLBACK_ANCHOR
```

The workflow did not pass the required pre-seed backup gate.

## 14. Exact baseline and outbox deltas

Not executed.

Expected first-run plan from prompt023 remains:

```text
baseline data/marker rows: 24
matching TblLocalOutbox rows: 22
```

Actual physical deltas: not collected because no mutation occurred.

## 15. Completion-marker-last proof

Source/test proof from prompt023 remains valid.

Physical marker-last proof: not executed because rollback anchor failed before seed.

## 16. Excluded/runtime zero-delta proof

No excluded/runtime table mutation occurred because no Phase 2 seed was executed.

Physical zero-delta counts were not collected; the process stopped before mutation.

## 17. Same-version replay zero-delta proof

Not executed.

The first physical run was blocked, so replay was not attempted.

## 18. Transaction/rollback proof

Source/test proof from prompt023 remains valid:

```text
focused InstallationV0 tests: 37 passed, 0 failed, 0 skipped
```

Physical transaction proof: not executed.

No restore was attempted because there was no mutation and no non-transactional corruption.

## 19. WPF operator handoff

No WPF handoff was performed for Phase 2 physical testing.

Current operator state:

```text
WPF remains ready for later Visual Studio/operator launch after rollback-anchor credential handoff is corrected.
Active visible label should remain prompt023.
Phase 2 physical seed is still not started.
```

## 20. First missing requirement

No runtime missing table/default was observed because the physical seed did not run and WPF was not launched for Phase 2 testing.

First concrete missing requirement before physical trial:

```text
approved pgpass credential handoff must supply a password to PostgreSQL clients for the required 127.0.0.1 / hung / obm_pos_dev_v0_pg connection without exposing the secret.
```

## 21. No reference DB mutation/no secret leakage/no source push

Confirmed:

- no mutation to `obm_pos_dev_v0_pg`;
- no mutation to reference/protected/production databases;
- no `pgpass` content read or printed;
- no password, token, connection string, or protected credential printed;
- no WPF/API/POS source commit;
- no OBM source push;
- no restore attempted.

Only this coordination report is committed.

## 22. Coordination commit SHA

This report is intended to be committed as:

```text
report/report024.md
```

The exact coordination commit SHA is reported by Codex after commit and push because a Git commit cannot contain its own final hash without changing that hash.

