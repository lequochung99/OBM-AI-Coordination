# Prompt 039 — Close post-v002 restart hydration and outbox accounting

## Physical evidence

Read completely:

```text
report/report037.md
report/report038.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The operator physically completed the prompt038 Phase 2 v002 transaction successfully. The success screen showed:

```text
PHASE2_V002_IDENTITY_SPINE_POS1_DB_PASS_READY_FOR_WPF_TEST
Phase 1 identity revalidated
TblTenant identity verified
TblPosLocal identity verified
TblPosRuntimeProfile identity verified
Identity spine verified
RuntimeState Activated
Permission parents reconciled
Employees inserted/adopted
Rows verified: 47
Outbox delta: 41
Runtime profile rows changed: 1
Runtime history rows inserted: 0
Marker context verified
Marker last: True
Transaction committed
Phase 2 v002 Complete
```

After closing and reopening WPF with the same ProductRoot, startup incorrectly displayed:

```text
Phase 2 Local DB Baseline: v001 complete; Employee v002 Runtime-State Upgrade Available
```

The button `Install Local Database Baseline` was enabled again.

This is a post-commit startup/hydration defect until proven otherwise. Do not run the Phase 2 action again automatically and do not ask the operator to click it again before reconciliation.

## Objective

Close two issues:

1. Startup must read durable database state and display `Phase 2 v002 Complete` after restart when all committed invariants exist.
2. Determine exactly what `Outbox delta: 41` meant and whether all required v002 permission/employee outbox events were actually committed.

Do not mutate the target database during prompt039. This is read-only physical reconciliation plus source/build/test correction only.

## Approved target and identity

```text
database = obm_pos_dev_v0_pg
environment = Development
ProductRoot = E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
v002 version = phase2-reference-driven-trial-v002-employees
```

The Phase 1 → Phase 2 identity spine remains authoritative:

```text
Phase 1 checkpoint/bootstrap identity
=
TblTenant
=
TblPosLocal
=
TblPosRuntimeProfile
=
marker context
```

Do not weaken or bypass this invariant.

## First requirement — authoritative read-only post-v002 audit

Connect as role `hung` using:

```text
BEGIN TRANSACTION READ ONLY
SELECT-only queries
ROLLBACK
```

Record sanitized counts and classifications for:

```text
TblTenant
TblPosLocal
TblPosRuntimeProfile
TblPosRuntimeStateHistory
TblEmployeePermission
TblEmployee
TblLocalOutbox
all physical marker/version tables used by InstallationV0/System Baseline
```

Expected successful post-v002 data state:

```text
current Phase 1 TblTenant row exists
current Phase 1 TblPosLocal row exists
one TblPosRuntimeProfile row aligned to Phase 1
RuntimeState = Activated
TblPosRuntimeStateHistory unchanged for identity-only repair
TblEmployeePermission = 7
TblEmployee = 20
v002 completion version physically present exactly once
```

Do not print employee names, PINs, contacts, payroll values, raw payloads, tokens, credentials, or connection strings.

## Marker table/source-of-truth audit

There has been inconsistent terminology across prior trial reports:

```text
Phase2TrialCompletionMarker
TblSystemBaselineVersion
baseline marker table with columns BaselineVersion, AppliedAtUtc, ToolName
```

Determine the exact physical source of truth by auditing:

```text
actual table name(s)
actual schema
actual columns
which table prompt038 executor wrote
which table startup hydration reads
which versions physically exist
whether v002 version exists exactly once
```

Do not create a second marker system.

Canonical rule:

```text
one physical version-marker source of truth
one writer path
one startup reader path
same version normalization/comparison
```

If prompt038 wrote v002 successfully but startup reads a different table or version identifier, fix startup to use the same canonical marker repository/query as the executor.

If no v002 marker physically exists despite the success screen, stop with:

```text
BLOCKED_PHASE2_V002_MARKER_COMMIT_PROOF_MISSING
```

Do not fabricate completion from row counts alone.

## Startup hydration behavior

After Phase 1 resume, startup must query the target database and classify:

```text
v001 absent
→ Not Started / Blocked according to prerequisites

v001 present, v002 absent
→ v001 Complete; Employee v002 Upgrade Available

v002 present and all identity/data/runtime invariants valid
→ Phase 2 v002 Complete

v002 present but identity/data/runtime invariants invalid
→ PHASE2_V002_COMMITTED_STATE_INVARIANT_MISMATCH
```

When v002 is complete:

```text
Install Local Database Baseline button disabled
or changed to Verify Local Database Baseline
```

Do not enable the mutating install action merely because the process restarted.

Startup must show safe proof fields:

```text
Marker version found
Identity spine verified
RuntimeState
Permission count
Employee count
Outbox accounting status
```

## Outbox authoritative accounting

The pre-v002 target had:

```text
TblLocalOutbox total = 21
```

The intended v002 work was:

```text
4 missing TblEmployeePermission insert events
20 TblEmployee insert events
expected new event count = 24
expected post-v002 total = 45
```

The physical success UI displayed:

```text
Outbox delta: 41
```

Determine exactly whether `41` was:

```text
A. total selected rows after run
B. actual transaction delta
C. count for a filtered subset
D. mislabeled UI value
E. evidence that permission outbox events were omitted
```

Audit read-only, using safe counts only:

```text
current TblLocalOutbox total
count by EntityType and Operation
count for TblEmployeePermission v002 events
count for TblEmployee v002 events
count by canonical SourceClientId shape
count by historical POS_N versus canonical POS_D shape
```

Do not print payload contents, raw employee identifiers, or private GUID maps.

### Outbox closure classification

If physical evidence proves:

```text
4 permission events + 20 employee events exist
```

then fix only the UI/accounting label so it clearly distinguishes:

```text
Outbox inserted this run
Outbox total after run
```

If employee events exist but one or more required permission events are missing, do not silently claim v002 fully closed. Return:

```text
PHASE2_V002_PERMISSION_OUTBOX_CLOSURE_REQUIRED
```

Implement only a versioned, deterministic closure plan in source; do not execute it in prompt039. The closure must not rewrite historical payloads or duplicate employee events.

If the legacy save/outbox contract proves permission rows are local-only and do not require outbox, document that source proof explicitly and correct the expected count rather than inventing events.

## Same-version replay safety

Do not physically run replay in prompt039.

Source/startup tests must prove that when v002 marker and invariants exist:

```text
startup reports Complete
mutating action is disabled or verify-only
no permission insert
no employee insert
no outbox insert
no runtime profile update
no history append
no marker insert
```

The production executor's same-version path must still be idempotent if invoked through a controlled test harness.

## Marker durability improvement decision

The current marker schema may expose only:

```text
BaselineVersion
AppliedAtUtc
ToolName
```

Do not alter marker schema in prompt039 unless startup cannot reliably verify v002 without it.

Report whether Production Installation V1 should add durable identity context fields such as:

```text
TenantGuid
PosGuid
LocalInstallationGuid
InstallationAttemptGuid
ManifestHash
BaselineRowCount
OutboxRowCount
```

Treat this as a Production V1 recommendation unless a current correctness defect requires a migration now.

## Existing v001 outbox SourceClientId

Prompt038 found 21 historical v001 outbox rows using `POS_N`, while new v002 rows should use canonical `POS_D`.

Prompt039 must audit counts but must not rewrite historical outbox rows.

Classify whether a separate versioned migration is needed before Production V1.

## Source boundaries

Likely permitted source areas:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2
E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0
```

Do not refactor unrelated WPF/POS modules.

## WPF build label

If source changes are needed, set:

```text
Build label: prompt039
Window title: OBM InstallationV0 Phase 1/2 - prompt039
```

If all behavior is correct and only the operator opened a stale binary, do not change source/label; prove the stale-binary condition exactly.

## Required tests

Add focused tests for:

```text
canonical marker writer and startup reader use the same table/repository
v002 marker found after restart
v002 complete classification requires marker + identity spine + runtime/data invariants
v002 complete disables mutating action
v001-only state still offers v002 upgrade
marker present with invariant mismatch fails closed
outbox inserted-this-run versus total-after-run labels are distinct
permission and employee outbox accounting
same-version complete startup performs zero mutations
prompt039 label when source changes
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Do not run WPF or mutate PostgreSQL automatically.

## Report 039

Create and push:

```text
report/report039.md
```

Required sections:

1. Verdict.
2. Physical prompt038 success evidence.
3. Restart hydration failure evidence.
4. Authoritative post-v002 DB counts.
5. Exact physical marker table/version source of truth.
6. Executor writer versus startup reader mismatch, if any.
7. Startup hydration correction.
8. Identity spine/runtime/data invariant readback.
9. Exact outbox total and v002 grouped counts.
10. Explanation of UI `Outbox delta: 41`.
11. Permission outbox closure decision.
12. Historical POS_N versus canonical POS_D counts and recommendation.
13. Same-version replay/startup zero-mutation proof.
14. Marker-schema Production V1 recommendation.
15. Exact source files changed.
16. Build/test commands and counts.
17. Active label proof.
18. No DB mutation/no secret leakage/no source push.
19. Exact operator restart/retest steps.
20. Coordination commit SHA.

## Valid verdicts

```text
PHASE2_V002_RESTART_HYDRATION_AND_OUTBOX_ACCOUNTING_READY_FOR_USER_RETEST
```

```text
PHASE2_V002_RESTART_HYDRATION_READY_PERMISSION_OUTBOX_CLOSURE_REQUIRED
```

```text
BLOCKED_PHASE2_V002_MARKER_COMMIT_PROOF_MISSING
```

```text
BLOCKED_PHASE2_V002_POST_COMMIT_INVARIANT_MISMATCH
```
