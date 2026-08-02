# Prompt126 Addendum — Existing implementation first; complete only proven gaps

This addendum is binding and overrides any wording in `prompt/prompt126.md` that could be interpreted as a greenfield rewrite.

## Authoritative operator correction

The OBM-POS installation flow is already substantially implemented. The task is not to rebuild InstallationV0 from zero.

Treat the current source as the primary implementation and preserve all working owners, UI, services, checkpoints, migrations, and tests unless direct evidence proves a specific defect.

The goal is:

```text
audit existing flow
-> identify exact missing or incorrect boundaries
-> modify the smallest number of production files
-> complete physical installation proof
```

## Mandatory reuse

Before adding any source type, prove no existing owner can be corrected.

Reuse when present:

```text
existing InstallationV0 window and controls
existing local PostgreSQL input model
existing protected DB-settings store
existing CleanLocalDatabaseService
existing canonical DbContext and migration chain
existing baseline seed executor/service
existing runtime-profile repository and state model
existing startup assessment/routing service
existing MainWindow launch path
existing Pairing Code redeem and protected token store
```

Do not create parallel replacements for these owners.

## Expected narrow corrections

The likely remaining corrections are limited to:

```text
1. Remove the prompt125 LocalPosDatabaseName ownership from PlatformApp/pairing/redeem/checkpoint active flow.
2. Restore local DB-name ownership to the existing WPF local DB setup/settings contract.
3. Remove active fixed fallback references to obm_pos_dev_v0_pg from normal installation/runtime.
4. Attach the already-existing TblPosRuntimeProfile and TblPosRuntimeStateHistory mappings to the canonical migration chain when physically missing.
5. Connect the existing baseline transaction to the existing runtime-profile writer for the DatabaseReady transition.
6. Connect the existing local finalization boundary to the same writer for the ApplicationReady transition.
7. Simplify existing startup routing so ApplicationReady opens MainWindow before remote API/session evaluation.
8. Add guards/tests preventing normal startup or migrate from deleting/resetting an existing DB.
```

Do not assume every item requires code changes. First prove which are already implemented and which are actual gaps.

## Diff budget and stop rule

Prefer a surgical patch. Report:

```text
EXISTING_PRODUCTION_OWNERS_REUSED=<count and paths>
NEW_PRODUCTION_SERVICE_COUNT=<expected 0>
NEW_PARALLEL_UI_COUNT=0
PRODUCTION_FILES_CHANGED=<count and paths>
```

If implementation begins expanding into a new architecture, stop and return:

```text
BLOCKED_PROMPT126_SCOPE_EXPANSION_REQUIRES_OPERATOR_REVIEW
```

Do not continue a broad refactor without operator approval.

## Preserve working behavior

Do not rewrite or regress:

```text
existing Pairing Code issue/redeem behavior
existing WpfJwt validation/persistence
existing ProductRoot resolution
existing PostgreSQL provisioning credential boundary
existing local baseline content
existing MainWindow business behavior
existing outbox/provider/sync architecture
```

Pairing moves to post-local-install ownership by changing orchestration/gating, not by rebuilding the pairing implementation.

## Physical acceptance remains mandatory

Tests/builds are not enough. PASS still requires the actual existing WPF application, with the minimal corrections applied, to prove:

```text
fresh local DB name entered through existing WPF local setup
WPF creates the absent DB
existing migrations apply from zero
existing baseline service commits
runtime profile transitions reach ApplicationReady
MainWindow opens with API offline and no Pairing Code requirement
restart opens MainWindow directly
post-install Pairing Code redeem still works through the existing flow
subsequent API/token failure does not re-enter InstallationV0
```

The report must distinguish:

```text
existing code reused unchanged
existing code corrected
new code added only because no existing owner existed
```

## Status locks

Until physical MainWindow proof passes:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=false
MANUAL_POS1_TEST_READY=false
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
```
