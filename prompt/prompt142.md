# prompt142 — Move bootstrap completion validation into the active LocalPosStartupService startup authority

## Authority

Canonical architecture authority:

`E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL_V005.md`

Evidence authority:

- `report/report140.md`
- `report/report141.md`

Prompt141 proved the source-level root cause:

- Primary: `C5_BASELINE_VALIDATION_NOT_EXECUTED`
- Contributing: `C2_NORMAL_STARTUP_BYPASSES_PROMPT140_ASSESSMENT`

The active startup authority is:

```text
App.xaml.cs
-> LocalPosStartupService.AssessAsync
-> RuntimeModeGate
-> StartNormalApplicationAsync
-> ShowMainWindowForActivatedRuntimeAsync
```

The prompt140 marker/baseline guard currently lives in `RuntimeProfileStartupAssessmentService`, which is not called by the active normal startup route.

## Objective

Correct the active startup authority so MainWindow eligibility requires all of the following:

```text
exactly one Activated/ApplicationReady runtime profile row
Tenant identity matches
POS identity matches
canonical schema is present/current
Phase2 completion marker exists and matches Tenant/POS/version/status
approved baseline/bootstrap proof is complete
```

A database with `Activated` but missing marker or required baseline must return a recoverable installation/bootstrap-incomplete decision and must not open MainWindow.

## Mandatory reuse-first rule

Do not create a new startup coordinator, new readiness table, new route, or new verification service.

Reuse existing owners and logic:

- `LocalPosStartupService`
- existing marker/baseline verification logic currently in `RuntimeProfileStartupAssessmentService`
- existing runtime profile repository/state semantics
- existing InstallationV0 recovery/install-resume route

Prefer extracting one shared verifier used by both services only if necessary to prevent duplicated SQL/threshold drift. If extraction is unnecessary, move the authoritative validation into `LocalPosStartupService` and make the other assessor delegate to the same owner.

There must be exactly one semantic definition of `ApplicationReady/Activated` eligibility.

## Phase A — Read-only source confirmation

Before editing, report exact current C# owners and call sites:

1. `App.xaml.cs` constructor startup assessment call.
2. `RetryStartupAssessmentAsync` call.
3. `LocalPosStartupService.AssessAsync` final `Ready` branch.
4. `RuntimeProfileStartupAssessmentService.VerifyBootstrapBaselineAsync`.
5. InstallationV0 route/result mapping for incomplete bootstrap.
6. Any DI registrations for both assessment services.

Produce a short before-change call-chain table with file, class, method, condition, and next call.

## Phase B — Correct the active startup gate

Implement the minimum safe correction.

### Required active predicate

Before `LocalPosStartupService` returns `Ready`, it must verify the same canonical completion proof used by prompt140.

At minimum:

```text
marker table exists
matching completion marker count = 1
required baseline proof meets the approved source contract
runtime profile state = Activated/ApplicationReady
Tenant/POS identity matches
```

Do not merely check that tables exist.

Do not treat runtime state alone as completion proof.

### Baseline contract consistency

Use the exact baseline contract implemented by the approved Phase 2 bootstrap executor and marker hard gate. Do not invent new thresholds or substitute unrelated employee/service/business counts.

Prompt141 showed a legacy verifier using permission/employee thresholds. Audit whether those thresholds actually match the current prompt140 bootstrap owner. If they do not, replace them with the exact approved baseline proof from the current bootstrap executor/marker contract.

The startup verifier and bootstrap executor must agree on the same completion semantics.

### Decision mapping

If marker/baseline is incomplete:

```text
LocalPosStartupDecision = InstallationIncomplete or equivalent recoverable bootstrap-incomplete decision
CanEnterNormalApplication = false
MainWindow = not constructed/not shown
InstallationV0 or recovery path = shown
```

Use a specific safe technical reason, for example:

```text
LOCAL_BOOTSTRAP_COMPLETION_PROOF_MISSING
LOCAL_BOOTSTRAP_BASELINE_INCOMPLETE
```

Do not map this to `UnknownFailure`.

### Existing incomplete DB repair

The current `obm_pos_dev_v1_pg` must remain intact.

After the corrected startup gate blocks MainWindow, the existing Install/Resume flow must be able to run the prompt140 bootstrap executor idempotently and repair the DB.

Do not automatically mutate the DB during normal startup assessment.

## Phase C — Remove semantic divergence

Audit `RuntimeProfileStartupAssessmentService` after the correction.

Choose one:

1. Delegate both services to one shared bootstrap-completion verifier, or
2. Make `RuntimeProfileStartupAssessmentService` delegate to the active authority, or
3. Remove/deprecate only the duplicate readiness predicate if proven safe and unused.

Do not leave two independent definitions of readiness that can drift again.

Document the final owner explicitly.

## Required tests

Add/update focused tests proving:

1. `Activated` + Tenant/POS + missing marker -> not Ready.
2. `Activated` + marker + incomplete baseline -> not Ready.
3. `Activated` + marker + complete approved baseline -> Ready.
4. `DatabaseReady` + complete baseline -> not Ready until final state transition.
5. Duplicate runtime profile rows -> not Ready.
6. Tenant mismatch -> not Ready.
7. POS mismatch -> not Ready.
8. Missing marker table -> recoverable installation incomplete, not UnknownFailure.
9. Both normal startup and Open OBM-POS retry path use the same active verifier.
10. No startup assessment performs DB mutation.
11. Prompt140 bootstrap executor still creates zero installation `TblLocalOutbox` rows.
12. Existing incomplete DB remains eligible for Install/Resume repair.

Build:

- `InstallationV0` build: 0 errors.
- focused Startup/InstallationV0 tests: all pass.

## Physical verification

After source/test verification, run the operator-visible physical lane if possible.

Use the existing DB; do not reset it.

Expected order:

```text
launch latest WPF label prompt142
-> startup detects Activated but marker/baseline incomplete
-> MainWindow does not open
-> InstallationV0/recovery UI opens
-> Install/Resume Local Database Baseline enabled
-> run once
-> prompt140 bootstrap executor repairs baseline + marker + Activated atomically
-> MainWindow opens
```

Then prove:

```text
pending migrations = 0
exactly one runtime profile row
state = Activated/ApplicationReady
completion marker = exactly one matching row
approved baseline proof complete
installation TblLocalOutbox rows = 0
MainWindow responsive 60 seconds with API offline
restart 1 -> MainWindow directly
restart 2 -> MainWindow directly
InstallationV0 does not flash after completion
```

If no live process/operator session is available, do not claim physical PASS.

## Forbidden actions

Do not:

- drop/recreate/reset/copy the DB;
- manually insert/update marker, baseline, or runtime state;
- bypass readiness to open MainWindow;
- create another startup/readiness service;
- modify pairing, PlatformApp, API authorization, sync, SignalR, Companion, payment terminal, category/booking/price work;
- seed employees, services, customers, invoices, booking, output info, or business history;
- expose secrets, tokens, raw GUIDs, passwords, or connection strings.

## Required report

Create and push:

`report/report142.md`

The report must include:

- verdict;
- exact final canonical startup authority;
- before/after C# call chain;
- files/classes/methods changed;
- exact readiness predicate after correction;
- how semantic divergence was removed;
- test/build totals;
- count-only DB evidence if physically inspected;
- destructive-operation counts;
- physical result or exact blocker;
- private artifact version/SHA if created;
- coordination commit SHA.

PASS verdict only if physically proven:

`OBM_WPF_V005_ACTIVE_STARTUP_BOOTSTRAP_GATE_REPAIRED_MAINWINDOW_OFFLINE_RESTARTS_PROVEN`

Otherwise use a precise blocked verdict.