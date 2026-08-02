# prompt143 — Fix fresh-create continuation from migrated empty DB to bootstrap/ApplicationReady

## Context

Current physical operator result after intentionally dropping disposable Development DB `obm_pos_dev_v1_pg`:

```text
Target DB absent
-> WPF creates DB
-> migrations create schema/tables
-> DB remains empty of approved baseline/bootstrap rows
-> Install/Resume Local Database becomes disabled
-> flow does not continue to bootstrap
-> MainWindow must not open
```

The active source already contains prior corrections from prompt134–prompt142, including:

- V005 order: PostgreSQL preflight -> Pairing/checkpoint -> DB create/resume -> migrations -> bootstrap -> ApplicationReady/Activated -> MainWindow.
- `LocalPosStartupService` now calls shared `BootstrapCompletionVerifier` before returning `Ready`.
- `PostgreSqlPhase2ReferenceSeedExecutor` is the approved bootstrap executor.
- bootstrap completion marker and baseline rows must commit before `Activated`.
- installation/bootstrap must create zero `TblLocalOutbox` rows.

The current defect is the fresh-create continuation boundary: after successful DB creation and migrations, the UI/command state does not continue or allow continuation into bootstrap.

## Objective

Repair the existing InstallationV0 flow so a fresh database proceeds deterministically from schema creation to bootstrap and `ApplicationReady`, without requiring a restart, manual DB edits, a second installer, or a new readiness framework.

Canonical outcome:

```text
checkpoint exists
+ local DB inputs valid
+ target DB absent
-> create DB
-> apply migrations
-> write/retain DatabaseReady semantic as appropriate
-> invoke approved bootstrap executor
-> insert approved baseline + completion marker
-> write Activated/ApplicationReady last in same bootstrap transaction
-> shared verifier returns complete
-> Open OBM-POS/MainWindow becomes eligible
```

## Mandatory investigation before editing

Trace the exact C# call chain for one click of `Install/Resume Local Database` from UI to completion:

1. `InstallationV0Window` click/command handler.
2. command predicate in `InstallationV0CommandRules` or current owner.
3. local DB create/resume owner (`CleanLocalDatabaseService` or exact active owner).
4. migration completion callback/result.
5. post-migration readiness reassessment.
6. Phase 2/bootstrap invocation decision.
7. UI state recomputation and button enablement.

Produce a before/after table with:

| Order | File | Class | Method | Input state | Condition | Returned state | Next call |
|---:|---|---|---|---|---|---|---|

Identify exactly which condition disables `Install/Resume` after migrations while bootstrap is incomplete.

## Required correction

Use the existing owners only. Prefer one continuous user action:

```text
Install/Resume click
-> create/resume DB
-> migrate
-> immediately invoke existing approved bootstrap executor
-> reassess via shared verifier
-> refresh UI
```

If the current architecture intentionally separates migration and bootstrap into two steps, then after migration completion the same button must remain/re-become enabled with a clear `Resume bootstrap` state. However, the preferred minimal correction is automatic continuation in the same operation because the operator already initiated Install/Resume and no further input is required.

The correction must handle all idempotent states:

```text
DB absent
-> create + migrate + bootstrap

DB exists with schema only
-> skip create, confirm/resume migrations, bootstrap

DB exists partially bootstrapped
-> resume bootstrap idempotently

DB complete with marker/baseline + Activated
-> do not reseed; route Ready

DB contains business/user data with inconsistent installation state
-> fail closed/recovery; no automatic destructive action
```

## State and transaction rules

- `Activated/ApplicationReady` must never be written before bootstrap rows and completion marker.
- Approved baseline/bootstrap rows + marker + final runtime state must commit atomically through the existing bootstrap executor/transaction owner.
- Migration transaction remains owned by the migration path.
- No long transaction across UI or network calls.
- No manual runtime-profile force patch.
- No duplicate completion marker semantics.
- Startup assessment remains read-only.

## Command enablement requirements

After pairing/checkpoint is valid:

- Before DB creation: `Install/Resume Local Database` enabled when local DB inputs and safety checks are valid.
- During operation: disabled only while the operation is busy.
- After migration but bootstrap incomplete: operation automatically continues, or the button re-enables immediately for bootstrap resume.
- `Open OBM-POS` remains disabled until shared `BootstrapCompletionVerifier` returns complete.
- Clearing the Pairing Code textbox after successful redeem must not disable the operation.
- Existing protected checkpoint must be enough after restart; no new Pairing Code required unless credential/attempt recovery explicitly requires it.

Add a safe non-secret disabled reason/status so the operator can distinguish at least:

```text
Busy
MissingCheckpoint
InvalidLocalDbInputs
UnsafeTarget
MigrationRequired
BootstrapRequired
ApplicationReady
```

Do not expose passwords, tokens, Pairing Codes, raw identity GUIDs, or full connection strings.

## Baseline policy

Use only the approved existing bootstrap owner and its existing manifest/reference rows. Do not invent a new manifest in this prompt.

Expected installation baseline includes the existing approved required owners such as settings/parameters/printer defaults/roles/setup/runtime identity according to current source. It must not seed ordinary business data unless the existing approved bootstrap owner explicitly requires it.

Installation/bootstrap `TblLocalOutbox` row count must remain `0`.

## Prohibited actions

Do not:

- drop/recreate/reset/copy the DB;
- use `EnsureDeleted`, `DROP DATABASE`, `DROP SCHEMA`, or `TRUNCATE`;
- create a second installer, migration runner, bootstrap executor, startup coordinator, checkpoint store, or readiness table;
- bypass shared `BootstrapCompletionVerifier`;
- manually insert `Activated/ApplicationReady`;
- change API, PlatformApp, pairing, sync, SignalR, Category/Booking/Price work;
- hardcode secrets or identity values;
- claim physical PASS from build/tests only.

## Build label

Update the visible InstallationV0 coordination label to exactly:

```text
prompt143
```

This is required so the operator can prove the physical debug binary contains this correction.

## Required tests

Add focused tests proving at minimum:

1. Fresh DB absent -> one Install/Resume action continues through create, migrate, bootstrap, final readiness.
2. Schema-only existing DB -> bootstrap resumes; no create/drop.
3. Migration success + bootstrap incomplete never leaves command permanently disabled.
4. Bootstrap failure leaves non-ready state and allows safe retry.
5. Successful bootstrap writes marker/baseline before `Activated` and then shared verifier returns complete.
6. Existing complete DB does not duplicate baseline/marker.
7. `Open OBM-POS` disabled before completion and enabled only after shared verifier complete.
8. Installation/bootstrap outbox count remains zero.
9. No destructive SQL/API calls in active path.
10. Restart with checkpoint + schema-only DB routes to bootstrap resume without new redeem.

Run focused InstallationV0 and Startup tests plus builds for `InstallationV0.csproj` and `NailSalonNet8.csproj`.

## Physical proof

Physical PASS requires the operator-visible lane on the same disposable Development DB:

1. Run latest WPF debug main lane and visibly confirm label `prompt143`.
2. Use the existing valid protected pairing checkpoint.
3. Target DB is absent or current schema-only DB from the operator observation.
4. Click `Install/Resume Local Database` once.
5. WPF creates/resumes DB, migrations finish, bootstrap proceeds without dead-end.
6. Verify pending migrations = 0.
7. Verify completion marker exists and approved baseline counts are nonzero/expected.
8. Verify exactly one current runtime profile row in `Activated/ApplicationReady` only after bootstrap.
9. Verify installation/bootstrap `TblLocalOutbox` rows = 0.
10. Open MainWindow; verify baseline-backed screens are populated where expected.
11. Keep MainWindow responsive for 60 seconds with API offline.
12. Restart twice; both launches open MainWindow directly without InstallationV0 flashing.

If physical UI cannot be driven by Codex, report source/test verdict as blocked-physical and provide exact operator steps. Do not claim final PASS.

## Deliverable

Create and push:

```text
report/report143.md
```

Report must include:

- verdict;
- exact root-cause classification;
- before/after C# call-chain table;
- exact predicate that caused the post-migration dead-end;
- files/classes/methods changed;
- fresh/schema-only/partial/complete behavior table;
- build/test totals;
- destructive operation counts;
- physical result or blocked-physical reason;
- sanitized evidence only.

Use final PASS verdict only if all physical requirements succeed:

```text
OBM_WPF_V005_FRESH_CREATE_MIGRATE_BOOTSTRAP_APPLICATIONREADY_MAINWINDOW_OFFLINE_PHYSICALLY_PROVEN
```

Otherwise use a precise blocked verdict.