# Prompt125 Report

## Verdict

BLOCKED_ACTIVE_V0_DB_NAME_FALLBACK_REMOVAL

## Required Status

Report124 artifact SHA verified: yes

Report124 artifact SHA-256:

```text
57ef55a0ac26b70dab8c22b7ecc8c464609648b01c0f3459c587d2fe32b0de28
```

Canonical property name: `LocalPosDatabaseName`

PlatformApp input added: yes

Validation owner/method:

```text
OBM.PlatformAppV0.Contracts.LocalPosDatabaseNameContract.TryNormalize
```

Create-pairing-code request carries field: yes

Stored pairing authorization carries field: yes

Issued authorization immutable proof: yes

Redeem response carries field: yes

WPF checkpoint persists field: yes

Restart reads field without redeem: yes

Resolved Phase2 `TargetDatabaseName`:

```text
obm_pos_dev_v1_pg
```

Active v0 constant fallback count: 6

Database mutation count: 0

WpfJwt/token/header changes count: 0

Operator MainWindow screenshot ready: false

Manual POS1 test ready: false

## Build And Test Totals

PlatformApp tests: 12 passed, 0 failed, 0 skipped

API focused tests: 27 passed, 0 failed, 0 skipped

WPF focused tests: 57 passed, 0 failed, 0 skipped

Focused total: 96 passed, 0 failed, 0 skipped

PlatformApp build: 0 errors, 0 warnings

API build: 0 errors, 6 warnings

WPF InstallationV0 build: 0 errors, 0 warnings

WPF test project build: 0 errors, 197 warnings

## Notes

The canonical handoff was added and verified by focused tests through PlatformApp input, API pairing authorization persistence, redeem response, WPF protected checkpoint persistence, restart resume, and Phase2 request mapping.

The PASS gate is not met because the active InstallationV0 Phase2 path still contains active `obm_pos_dev_v0_pg` fallback references. Physical handoff PASS and MainWindow readiness are therefore not claimed.

No PostgreSQL database was created, dropped, migrated, seeded, or otherwise mutated during prompt125.

No WpfJwt, token, header, signing, validation, lifetime, or authorization-policy behavior was intentionally changed.

## Private Artifact

Private artifact path:

```text
E:\Project2026\RecoveryReports\PlatformPairingLocalDbNameHandoffV001
```

Aggregate SHA-256:

```text
372338b64a6ca458b6cfb3ca36d9786a70ddcff0a4f8e024b65b558add6dedcf
```

Coordination commit SHA:

```text
PENDING_COORDINATION_COMMIT_SHA
```
