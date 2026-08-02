# Prompt128 — Operator-approved physical fresh WPF installation to obm_pos_dev_v1_pg under canonical V004

## Operator approval

The operator explicitly approves one isolated Development physical installation run with the exact target:

```text
obm_pos_dev_v1_pg
```

This approval is narrow. It does not authorize dropping, truncating, clearing, renaming, or replacing any existing database.

Canonical authority:

```text
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL_V004.md
SHA-256: 54D38EB1DD0B1FE53564D4550138F74D8B3099E1D85D521A2821F20D808450CB
```

Starting report:

```text
report/report126.md
coordination commit: 7938ed6e5894fceb55a98043438076358cc71587
verdict: BLOCKED_WPF_DATABASE_SELF_PROVISIONING
private artifact manifest SHA-256: 2A5B03C3F792CE8373A955C0524A833B3724F5DFD83A4273B34B58F5AE5903E5
```

## Scope

Use the existing implementation completed under prompt126. Do not redesign or rebuild InstallationV0.

Execute only:

```text
1. Inspect the active WPF Phase2 target safety guard.
2. Make the smallest change needed to approve exactly obm_pos_dev_v1_pg for this Development installation lane.
3. Use one isolated, versioned Development ProductRoot.
4. Prove obm_pos_dev_v1_pg does not exist before WPF installation begins.
5. Run the real WPF local installation UI.
6. Let WPF create the DB, migrate from zero, seed the baseline, write runtime-profile states, and open MainWindow.
7. Restart twice with API offline.
8. Perform Pairing Code redeem only after local MainWindow acceptance passes.
```

## Hard safety rules

The safety change must be exact-target and fail-closed.

Allowed:

```text
exact database name: obm_pos_dev_v1_pg
environment: Development
one explicitly identified isolated ProductRoot
WPF CreateCleanDatabaseService creates the DB only when absent
```

Forbidden:

```text
wildcard database-name approval
prefix-only approval such as obm_pos_dev_*
disabling the safety guard globally
fallback to obm_pos_dev_v0_pg
manual CREATE DATABASE outside WPF
DROP DATABASE
DROP SCHEMA
TRUNCATE
EnsureDeleted
copying v0 into v1
resetting API DB
mutating production/customer/reference DB
```

If obm_pos_dev_v1_pg already exists at the start, stop with:

```text
BLOCKED_WPF_V1_TARGET_ALREADY_EXISTS
```

Do not delete it automatically.

## Preserve existing implementation

Reuse the current owners already proven by prompt126:

```text
existing InstallationV0 UI
existing local PostgreSQL input model
existing protected DB-settings store
CleanLocalDatabaseService.CreateCleanDatabaseAsync
canonical WPF DbContext and attached migrations
existing baseline seed service
existing runtime-profile repository
existing startup routing
existing MainWindow launch path
existing Pairing Code redeem/token store
```

New parallel service count must remain zero.

## Phase A — Exact safety-boundary correction

Record before and after:

```text
SAFETY_GUARD_OWNER=<class/method>
APPROVED_TARGET_BEFORE=<safe value/rule>
APPROVED_TARGET_AFTER=obm_pos_dev_v1_pg exact only
WILDCARD_APPROVAL_COUNT=0
GLOBAL_GUARD_BYPASS_COUNT=0
ACTIVE_V0_FALLBACK_COUNT=0
```

Add focused tests proving:

```text
obm_pos_dev_v1_pg accepted in the approved Development lane
obm_pos_dev_v0_pg is not used as fallback
other unapproved names are rejected
protected/production names are rejected
an existing target is not dropped or cleared
normal runtime cannot reach destructive reset helpers
```

## Phase B — Physical local installation

Use visible WPF label:

```text
prompt128
```

Use a new versioned isolated ProductRoot. Record its exact path.

Before launching WPF prove safely:

```text
TARGET_DB=obm_pos_dev_v1_pg
TARGET_DB_EXISTS_BEFORE=false
API_7161_LISTENER=false
PAIRING_CODE_REDEEMED_BEFORE_LOCAL_READY=false
```

Through the real WPF local setup UI provide:

```text
loopback PostgreSQL host
canonical PostgreSQL port
operator-approved local PostgreSQL username
protected local PostgreSQL password
DatabaseName=obm_pos_dev_v1_pg
```

Never print the password or full connection string.

WPF must perform:

```text
connection test
create DB because absent
persist protected local DB settings
apply attached Npgsql migrations from zero
pending migrations = 0
seed minimal baseline
write DatabaseReady atomically with baseline
complete local finalization
write ApplicationReady
open production MainWindow
```

Required physical database proof:

```text
TblPosRuntimeProfile exists
TblPosRuntimeStateHistory exists
current profile row count = 1
current profile state = ApplicationReady or the proven existing semantic equivalent
DatabaseReady transition count = 1
ApplicationReady transition count = 1
installation-created TblLocalOutbox rows = 0
no unrelated business seed
```

## Phase C — Offline MainWindow acceptance

Keep API port 7161 without a listener.

PASS requires:

```text
MainWindow opens
InstallationV0 closes and does not reopen
MainWindow remains responsive for at least 60 seconds
local DB remains usable
no Pairing Code/token is required
```

Close normally and restart twice.

Each restart must prove:

```text
MainWindow opens directly
InstallationV0 does not flash
profile remains ApplicationReady
DB rows remain intact
no migration reset
no duplicate baseline
no duplicate runtime-state transition
```

## Phase D — Post-install pairing

Only after Phase C passes:

```text
start full API/PlatformApp
create one valid Pairing Code through the existing Tenant/POS flow
redeem from WPF
persist WpfJwt through the existing protected token store
```

Then stop API and restart WPF once more.

Required:

```text
MainWindow still opens directly
cloud status may be Offline/Degraded
ApplicationReady is unchanged
local DB configuration is unchanged
```

Do not implement refresh tokens.

## Frozen work

Do not modify:

```text
Category Weight
Booking Weight
Price Weight save semantics
TblTenantPosDevice
API destination routing
sync architecture
CompanionApp/payment-terminal modeling
Firebase cleanup
.env cleanup
```

## PASS gate

PASS verdict only when all physical checks pass:

```text
OBM_WPF_V004_APPROVED_V1_SELF_PROVISIONING_MAINWINDOW_OFFLINE_AND_POST_INSTALL_PAIRING_PASS
```

Otherwise use one narrow blocker:

```text
BLOCKED_WPF_V1_TARGET_ALREADY_EXISTS
BLOCKED_WPF_EXACT_TARGET_SAFETY_GUARD
BLOCKED_WPF_V1_DATABASE_CREATION
BLOCKED_WPF_V1_MIGRATION_FROM_ZERO
BLOCKED_WPF_V1_BASELINE_TRANSACTION
BLOCKED_WPF_V1_APPLICATION_READY
BLOCKED_WPF_V1_MAINWINDOW_PHYSICAL_PROOF
BLOCKED_WPF_V1_POST_INSTALL_PAIRING
```

Never report PASS while InstallationV0 remains visible instead of MainWindow.

## Artifact and report

Create a new versioned private artifact:

```text
E:\Project2026\RecoveryReports\WpfV004ApprovedV1PhysicalInstallationV001
```

At minimum include:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
OPERATOR_APPROVAL.md
SAFETY_GUARD_BEFORE_AFTER.md
TARGET_ABSENCE_PROOF.md
PRODUCT_ROOT_PROOF.md
WPF_SELF_DATABASE_CREATION.md
MIGRATION_FROM_ZERO.md
BASELINE_DATABASE_READY_TRANSACTION.md
APPLICATION_READY_TRANSACTION.md
MAINWINDOW_60_SECOND_PROOF.md
RESTART_1_PROOF.md
RESTART_2_PROOF.md
POST_INSTALL_PAIRING_PROOF.md
FINAL_OFFLINE_RESTART_PROOF.md
FOCUSED_TEST_OUTPUT.txt
BUILD_OUTPUT.txt
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Push:

```text
report/report128.md
```

Do not edit prompt/report files other than the required report in the coordination repository.