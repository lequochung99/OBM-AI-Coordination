# Prompt132 — Restore WPF startup by making database-settings.json schemaVersion backward-compatible, then resume V1 installation safely

## Starting checkpoint

Read completely:

```text
prompt/prompt131.md
report/report131.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL_V004.md
```

Accepted current evidence:

```text
- Legacy SpacePos.Provisioning.Schema active dependency has been removed.
- Provisioning role and runtime role are separated.
- obm_pos_dev_v1_pg already exists but physical migration/ApplicationReady proof is incomplete.
- Build and focused tests passed in prompt131.
- Current WPF startup now fails before InstallationV0 can continue because the protected local configuration reader throws:
  System.InvalidOperationException: Missing schemaVersion in ...\Config\database-settings.json
```

## Exact scope

Fix only the local DB configuration compatibility/startup boundary, then resume the existing V1 installation flow.

Do not:

```text
- drop, recreate, truncate, reset, rename, or copy obm_pos_dev_v1_pg;
- edit database-settings.json manually to force PASS;
- expose or log PostgreSQL passwords, complete connection strings, tokens, or private identifiers;
- create a second configuration store, second reader, second startup coordinator, or second installer;
- change Pairing Code/WpfJwt/API header behavior;
- modify Category Weight, Booking Weight, TblTenantPosDevice, sync routing, CompanionApp, or BookingConsole;
- add a new schema/readiness framework.
```

## Root problem to prove

The current reader requires a nonempty `schemaVersion` property in `database-settings.json` and throws before the app can route to InstallationV0 or MainWindow.

Prove:

```text
CONFIG_READER_OWNER=<class/method>
CONFIG_WRITER_OWNER=<class/method>
CURRENT_FILE_EXISTS=yes/no
CURRENT_SCHEMA_VERSION_PRESENT=yes/no
CURRENT_LAST_KNOWN_VALID_EXISTS=yes/no
CURRENT_FILE_CREATED_BY_WHICH_WRITER=<exact source path/method>
WHY_WRITER_AND_READER_CONTRACT_DRIFTED=<exact evidence>
```

Do not print secret values.

## Required correction

Reuse the one existing protected local DB settings contract.

Implement the smallest backward-compatible rule:

```text
1. A legacy/current file that contains all required connection fields but lacks schemaVersion is not immediately fatal.
2. Parse and validate all required fields using existing validators.
3. Classify it as the supported legacy config version.
4. Rewrite/upgrade it atomically through the existing writer to the current canonical schemaVersion.
5. Preserve all valid non-secret fields and protected-password references.
6. Read back the upgraded file through the normal reader.
7. Continue startup without process termination.
```

Requirements:

```text
- no empty catch;
- no acceptance of malformed/missing required connection fields;
- no blind defaulting when the file is corrupt;
- no plaintext password migration;
- no manual operator file edit;
- use temp file + atomic replace or the existing atomic writer contract;
- preserve last-known-valid fallback semantics;
- unknown future schemaVersion must fail closed with a recoverable UI/error, not silently downgrade;
- invalid JSON must remain recoverable and must not open MainWindow blindly.
```

If the existing schema model already defines a legacy version identifier, reuse it. Do not invent multiple version systems.

## Startup behavior after correction

The exact operator-equivalent launch must:

```text
- show visible label prompt132;
- not crash or exit on missing schemaVersion;
- upgrade/read the config safely;
- resolve the existing V1 local DB settings;
- preserve API-offline local-first behavior;
- route to InstallationV0 resume if ApplicationReady is not yet proven;
- route to MainWindow only when ApplicationReady is valid.
```

## Resume V1 installation safely

After config compatibility is fixed, continue the existing production path against the already-existing `obm_pos_dev_v1_pg`.

Required behavior:

```text
- never drop/recreate the existing V1 DB;
- verify runtime owner/grants;
- run the existing in-process EF/Npgsql migration path idempotently;
- pending migrations = 0;
- run baseline/DatabaseReady transaction only if not already committed;
- run ApplicationReady finalization only if not already committed;
- no installation TblLocalOutbox rows;
- open MainWindow with API 127.0.0.1:7161 offline;
- keep MainWindow responsive for at least 60 seconds;
- close and restart twice;
- both restarts open MainWindow directly with no InstallationV0 flash.
```

If a new exact blocker appears, stop at that boundary and report it; do not reset the DB.

## Focused tests

Add/run focused tests for:

```text
legacy config without schemaVersion upgrades successfully;
upgraded config reads normally after restart;
required-field-missing legacy config fails recoverably;
invalid JSON fails recoverably;
unknown future schemaVersion fails closed;
last-known-valid fallback remains valid;
password is never serialized in plaintext;
atomic rewrite leaves no partial primary file;
ApplicationReady startup remains API-independent;
no destructive DB action is reachable from config migration.
```

Run WPF/InstallationV0 build and focused tests.

## PASS gate

PASS requires all of the following:

```text
CONFIG_SCHEMA_VERSION_COMPATIBILITY_FIXED=true
CURRENT_CONFIG_UPGRADED_THROUGH_PRODUCTION_WRITER=true
WPF_PROCESS_CRASH_AFTER_FIX=false
DB_V1_DROPPED_OR_RECREATED=false
PENDING_MIGRATIONS=0
RUNTIME_PROFILE_STATE=ApplicationReady-equivalent
MAINWINDOW_OPENED_DIRECTLY=true
MAINWINDOW_60_SECOND_PROOF=true
RESTART_1_MAINWINDOW_DIRECT=true
RESTART_2_MAINWINDOW_DIRECT=true
INSTALLATIONV0_FLASH_AFTER_APPLICATIONREADY=0
API_OFFLINE_LOCAL_FIRST=true
```

PASS verdict:

```text
OBM_WPF_CONFIG_SCHEMA_COMPATIBILITY_FIXED_V1_APPLICATIONREADY_MAINWINDOW_OFFLINE_PHYSICALLY_PROVEN
```

Narrow blockers only:

```text
BLOCKED_WPF_CONFIG_READER_WRITER_CONTRACT
BLOCKED_WPF_CONFIG_ATOMIC_UPGRADE
BLOCKED_WPF_V1_MIGRATION_RESUME
BLOCKED_WPF_APPLICATIONREADY_PHYSICAL_PROOF
BLOCKED_WPF_MAINWINDOW_PHYSICAL_PROOF
```

## Required artifact/report

Preserve previous artifacts. Create:

```text
E:\Project2026\RecoveryReports\WpfConfigSchemaCompatibilityV001
```

Include at minimum:

```text
PRIVATE_HANDOFF.md
CONFIG_READER_WRITER_CALLCHAIN.md
LEGACY_CONFIG_CLASSIFICATION.md
ATOMIC_UPGRADE_PROOF.md
BEFORE_AFTER_SANITIZED_CONFIG_SHAPE.md
FOCUSED_TEST_OUTPUT.txt
V1_RESUME_PROOF.md
MAINWINDOW_60_SECOND_PROOF.md
RESTART_PROOF.md
FINAL_STATE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Write and push:

```text
report/report132.md
```

Do not report PASS while WPF cannot start or while MainWindow has not been physically proven.
