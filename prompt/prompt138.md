# prompt138 — Refresh InstallationV0 command enablement after successful Pairing Code redeem

## Context

The operator physically completed the V005 pairing-first flow in WPF. The screenshot shows:

- Pairing Code redeem returned HTTP 200.
- Protected API call succeeded with `WPF_BOOTSTRAP_IDENTITY_VERIFIED`.
- WpfJwt was received.
- Required scope was verified.
- Tenant identity, POS1 identity, PosGuid, InstallationAttemptId, AttemptVersion, LocalInstallationGuid, and credential expiration were all verified.
- The UI text says Phase 1 resume is verified and Pairing Code is not required.

However, after redeem succeeds, the following buttons remain disabled:

- `Install/Resume Local Database`
- `Open OBM-POS`

This indicates stale UI/readiness state or command enablement not being recomputed after the protected pairing checkpoint is persisted.

## Goal

Make the smallest source correction so that successful pairing/checkpoint persistence immediately refreshes InstallationV0 readiness and enables the correct next action under V005.

Do not redesign the installation flow.

## Canonical behavior

V005 order:

```text
PostgreSQL preflight
-> Pairing Code redeem
-> protected installation checkpoint persisted
-> create/resume local DB
-> migrations
-> DatabaseReady
-> Tenant/POS bootstrap
-> ApplicationReady/Activated
-> MainWindow
```

After successful redeem/checkpoint persistence:

```text
Install/Resume Local Database = enabled
Open OBM-POS = disabled until ApplicationReady/Activated
Redeem Pairing Code = no longer required for the same valid attempt
```

After ApplicationReady/Activated:

```text
Install/Resume Local Database = disabled or non-primary according to existing UI behavior
Open OBM-POS = enabled
```

## Required investigation

Trace the exact active owners for:

1. Pairing redeem completion handler.
2. Protected checkpoint persistence.
3. PostgreSQL preflight state.
4. `Install/Resume Local Database` IsEnabled/CanExecute predicate.
5. `Open OBM-POS` IsEnabled/CanExecute predicate.
6. Any cached readiness model, view-model property, command, or code-behind state.
7. Startup/resume assessment that already computes the same state on process restart.

Determine whether the bug is one of:

- missing reassessment after successful redeem;
- stale cached checkpoint state;
- missing `PropertyChanged` / `CanExecuteChanged` notification;
- UI predicate still tied to pre-V005 ordering;
- button predicate incorrectly requiring DB/ApplicationReady before allowing DB installation;
- successful redeem path not invoking the existing resume/readiness owner.

## Implementation rules

Prefer reusing the existing startup/resume assessment method after checkpoint persistence.

The minimal expected pattern is conceptually:

```text
redeem succeeds
-> persist protected checkpoint
-> reload/reassess installation state through existing owner
-> update UI status model
-> raise property/command notifications
-> Install/Resume Local Database becomes enabled
```

Do not duplicate readiness logic in the click handler.

## Critical button rules

### Install/Resume Local Database

Enable only when all required local fields and V005 prerequisites are valid, including:

- PostgreSQL host/port valid;
- provisioning and runtime credentials present in the current UI attempt;
- database name passes safety validation;
- protected pairing checkpoint exists and is valid for the current installation attempt;
- DB is absent, empty, or safely resumable;
- installation is not already ApplicationReady/Activated;
- no install operation is already running.

The button must not require Pairing Code text to remain in the textbox after redeem.

### Open OBM-POS

Must remain disabled until the local runtime state is ApplicationReady/Activated.

Do not enable `Open OBM-POS` merely because pairing succeeded.

## Existing DB safety

The target `obm_pos_dev_v1_pg` currently exists and was observed empty. Preserve it.

Forbidden:

```text
DROP DATABASE
DROP SCHEMA
TRUNCATE
EnsureDeleted
recreate/reset
manual CREATE TABLE
manual ApplicationReady insertion
```

After the button fix, the operator will physically test resume on this existing empty DB.

## Security

Do not print or commit:

- Pairing Code
- JWT/token
- Google identity
- TenantGuid
- PosGuid
- InstallationAttemptId
- LocalInstallationGuid
- PostgreSQL passwords
- full connection strings

The prior screenshot exposed raw GUID-like values in the UI diagnostics. Do not expand this scope into a diagnostics-redaction redesign unless the active source change is trivial and directly adjacent. At minimum, ensure report138 does not reproduce those values.

## Tests

Add focused tests proving:

1. Successful redeem + persisted checkpoint + valid PostgreSQL inputs => Install/Resume enabled.
2. Pairing textbox cleared after success does not disable Install/Resume.
3. Failed redeem => Install/Resume remains disabled.
4. Missing/invalid checkpoint => Install/Resume disabled.
5. Install operation running => Install/Resume disabled.
6. ApplicationReady/Activated => Open OBM-POS enabled.
7. Pairing success without ApplicationReady => Open OBM-POS disabled.
8. Restart with valid protected checkpoint restores Install/Resume eligibility without another redeem.
9. No destructive DB operation is introduced.

Run:

- InstallationV0 build
- focused InstallationV0 tests

## Physical test

If the operator session is available, verify:

1. Launch WPF with label `prompt138`.
2. Existing valid protected pairing checkpoint is loaded, or redeem a fresh Pairing Code.
3. `Install/Resume Local Database` becomes enabled immediately after successful redeem/reassessment.
4. `Open OBM-POS` remains disabled before ApplicationReady.
5. Click Install/Resume once.
6. Resume the existing empty V1 DB without drop/recreate.
7. Continue migration/bootstrap toward ApplicationReady and MainWindow if no new blocker occurs.

Do not claim full V005 physical PASS unless MainWindow 60-second proof and two restart proofs complete.

## Scope freeze

Do not change:

- PlatformApp admin authorization
- API Google exchange wiring
- Pairing endpoint/token contract
- installation plan identity contract
- migration architecture
- baseline contents
- sync/SignalR
- Category/Booking/Price weights
- Companion/payment/Firebase
- refresh-token architecture

## Report

Write and push:

```text
report/report138.md
```

Report must include:

- root-cause classification;
- exact active state/readiness owner reused;
- files changed;
- before/after command predicates;
- build/test totals;
- destructive operation counts;
- physical result, if performed;
- no secrets or raw identity values.

Source-ready verdict:

```text
WPF_V005_POST_REDEEM_INSTALL_RESUME_ENABLEMENT_FIXED
```

Full physical verdict only if the entire remaining V005 acceptance succeeds:

```text
OBM_WPF_V005_PAIRING_FIRST_APPLICATIONREADY_MAINWINDOW_OFFLINE_PHYSICALLY_PROVEN
```
