# Prompt 031 — Integrate Phase 2 employee upgrade with POS runtime profile/state history

## Current verified state

Read completely:

```text
report/report028.md
report/report030.md
prompt/prompt030.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Current facts:

```text
v001 physical baseline seed: PASS
v001 marker exists
v002 all-employee source implementation: READY_FOR_USER_TEST
v002 physical employee seed: NOT EXECUTED
active source label before this task: prompt030
```

Report030 confirms that `TblEmployee` transformation/transaction/outbox source is implemented, but the target DB employee seed has not yet run.

The operator observed these existing tables:

```text
TblPosRuntimeProfile
TblPosRuntimeStateHistory
TblPosTerminalConfig
```

The InstallationV0 UI has previously shown an incorrect `Not Started` state after restart even though Phase 2 v001 was complete. Prompt030 added marker hydration, but it did not prove that the canonical POS runtime profile/history state machine is updated.

## Objective

Determine and implement the correct relationship among:

```text
Phase2TrialCompletionMarker
TblPosRuntimeProfile
TblPosRuntimeStateHistory
TblPosTerminalConfig
Phase 1 ApiAuthorized checkpoint
InstallationV0 UI state
```

Then integrate the v002 employee upgrade with the canonical runtime state transition so that database state and UI state agree after restart.

Do not invent a parallel state machine if the WPF already has one. Reuse the existing runtime profile/state enum, transition service, and history writer where possible.

## Step 1 — Focused state-machine audit

Inspect source/models/migrations/callers for:

```text
TblPosRuntimeProfile
TblPosRuntimeStateHistory
TblPosTerminalConfig
RuntimeState
InstalledHealthy
Activated
DatabaseProvisioned
BaselineInstalled
installation/bootstrap/runtime profile services
```

Report exactly:

```text
Table/model
Purpose
Current row count in obm_pos_dev_v0_pg
Primary/stable key
Tenant/POS/installation scope
Current state/value
Current writer methods
Current reader/UI methods
History append behavior
Whether TblPosTerminalConfig participates in installation status
```

Use only sanitized values. Do not print credentials, connection strings, terminal secrets, or private configuration.

Classify authority:

```text
current runtime state source of truth
append-only transition history
seed-version completion proof
terminal configuration only
machine-side Phase 1 proof
```

Expected conceptual separation, to verify against source rather than assume:

```text
TblPosRuntimeProfile       -> current runtime state
TblPosRuntimeStateHistory  -> transition audit history
Phase2TrialCompletionMarker -> immutable seed/upgrade completion version
TblPosTerminalConfig       -> terminal/device configuration, not overall installation state
```

If source contradicts this, follow source and document the exact contract.

## Step 2 — TblEmployee status

Confirm prompt030 implementation and current physical target state:

```text
reference employee count/type distribution
current target TblEmployee count
v002 marker presence
current employee outbox count
```

Do not print employee names or private values.

Do not claim employee seeding complete unless physical target rows and v002 marker exist.

## Step 3 — Canonical runtime transition decision

Identify the exact existing runtime-state enum/value that should represent the state after:

```text
Phase 1 authorized
v001 local baseline complete
v002 employee upgrade complete
local DB verified
WPF ready to continue normal application startup
```

Do not create a new state name merely for prompt031 if an existing canonical state such as `Activated` or `InstalledHealthy` already represents it.

If no existing state is correct, stop with:

```text
BLOCKED_POS_RUNTIME_STATE_CONTRACT_UNRESOLVED
```

and report the exact missing decision.

## Step 4 — Atomic v002 integration

Update the real PostgreSQL v002 employee executor so that one target transaction performs, in dependency order:

```text
BEGIN
  revalidate Phase 1 protected identity
  verify target obm_pos_dev_v0_pg Development
  verify PreV002 backup anchor
  acquire advisory transaction lock
  verify v001 marker/runtime profile compatibility
  read reference employees read-only
  insert/adopt TblEmployee rows
  insert matching sanitized TblLocalOutbox rows
  verify employee/UI classification invariants
  update or verify TblPosRuntimeProfile current canonical state
  append one TblPosRuntimeStateHistory transition only when state actually changes
  write v002 marker last
  read back employee/profile/history/marker invariants
COMMIT
```

All employee rows, outbox rows, runtime profile update, state history row, and v002 marker must share the same target PostgreSQL transaction.

On failure:

```text
ROLLBACK employees
ROLLBACK employee outbox
ROLLBACK runtime profile update
ROLLBACK runtime history append
ROLLBACK v002 marker
preserve v001 marker/data
preserve Phase 1 checkpoint
```

## Runtime profile/history idempotency

### State already correct

```text
TblPosRuntimeProfile already equals canonical post-v002 state
-> verify
-> do not append duplicate history solely because action was replayed
```

### State transition required

```text
old canonical state -> new canonical state
-> update profile
-> append exactly one history row with deterministic transition identity/reason/version
```

### Same-version replay

Must produce:

```text
TblEmployee delta = 0
TblLocalOutbox delta = 0
TblPosRuntimeProfile delta = 0
TblPosRuntimeStateHistory delta = 0
v002 marker delta = 0
```

### Marker/profile mismatch

Classify and fail closed with a safe result code. Do not silently display `Complete` when marker and runtime profile contradict each other.

Examples:

```text
marker present + profile older/stale
profile complete + marker absent
history latest transition conflicts with current profile
wrong TenantGuid/PosGuid/runtime profile identity
```

Use exact result codes defined by this implementation and report them.

## Startup hydration and UI

After Phase 1 resume, WPF must query the canonical local status sources and display a truthful state.

UI decision must use:

```text
Phase 1 checkpoint/API identity
v001/v002 completion markers
TblPosRuntimeProfile current state
latest relevant TblPosRuntimeStateHistory entry for diagnostics
```

Expected states:

```text
v001 marker present, v002 absent, runtime profile compatible
-> Phase 2 v001 Complete; Employee v002 Upgrade Available

v002 marker present and runtime profile canonical
-> Phase 2 v002 Complete

marker/profile mismatch
-> Phase 2 Blocked: Local state mismatch

no marker and pre-install runtime state
-> Not Started
```

`TblPosRuntimeStateHistory` is evidence/audit; do not derive current state solely from an arbitrary historical row when `TblPosRuntimeProfile` is the canonical current-state table.

Do not use `TblPosTerminalConfig` as the app installation state unless the focused source audit proves that contract.

## Physical test boundary

The operator has approved the explicit WPF action on:

```text
obm_pos_dev_v0_pg
```

Before physical mutation verify the existing post-v001/pre-v002 backup anchor from report030.

The physical v002 action should:

```text
seed all approved reference employees
write employee outbox
transition/verify runtime profile
append history if changed
write v002 marker last
commit
```

Then restart WPF and prove the status hydrates as `v002 Complete`, not `Not Started`.

Open normal management and checkout UI for operator verification:

```text
management UI sees all employees
checkout sees only Staff
non-Staff excluded from checkout
```

## Exclusions

Do not seed or modify merely for this task:

```text
services/categories/products
employee-service mappings
customers/gift cards
invoice/output/payment data
turn/queue/payroll runtime history
terminal credentials
TblPosTerminalConfig secret/private fields
```

Do not modify historical runtime-state rows. History is append-only.

## WPF label

Because source changes are expected, set:

```text
Build label: prompt031
Window title: OBM InstallationV0 Phase 1/2 - prompt031
```

## Build/tests

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Add focused tests for:

```text
runtime state source-of-truth classification
profile/history/marker transaction ownership
profile update before marker-last
single history append on real transition
no history duplicate on replay
marker/profile mismatch blocking
startup hydration after restart
TblPosTerminalConfig not misused as app state
employee seed/outbox still atomic
rollback restores profile/history/employee/outbox/marker
prompt031 label
```

## Report

Create and push:

```text
report/report031.md
```

Required sections:

1. Verdict.
2. Current TblEmployee implementation versus physical state.
3. Runtime table/model/caller audit.
4. Current row counts and sanitized current-state proof.
5. Authoritative source-of-truth decision.
6. Exact canonical post-v002 runtime state selected.
7. TblPosTerminalConfig classification.
8. Transaction integration details.
9. Runtime profile update and history append behavior.
10. Marker-last/readback proof.
11. Mismatch/fail-closed result codes.
12. Physical v002 employee seed counts/outbox deltas, if executed.
13. Runtime profile/history/marker physical deltas, if executed.
14. Restart/UI hydration proof.
15. Same-version zero-delta proof.
16. Management/checkout filtering proof.
17. Exact source files changed.
18. Build/test commands and counts.
19. Prompt031 label proof.
20. No secret/reference mutation/source push proof.
21. Coordination commit SHA.

## Valid verdicts

Physical v002 employee/state transition passed:

```text
PHASE2_V002_EMPLOYEES_AND_RUNTIME_STATE_POS1_DB_PASS_READY_FOR_WPF_TEST
```

Implementation ready, physical action pending:

```text
PHASE2_V002_EMPLOYEES_AND_RUNTIME_STATE_READY_FOR_USER_TEST
```

State contract unresolved:

```text
BLOCKED_POS_RUNTIME_STATE_CONTRACT_UNRESOLVED
```

Implementation blocked:

```text
BLOCKED_PHASE2_V002_RUNTIME_STATE_INTEGRATION
```
