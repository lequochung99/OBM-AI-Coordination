# Prompt 084 — Final audit of Weird Tip seed: exactly one tenant row, default disabled, no seed outbox

## Operator physical state

The operator manually removed one duplicate `TblSetupWeird` row from the current local database. The Weird Tip UI now behaves correctly.

This manual repair fixes the current database only. It does not by itself prove that future clean installations, seed reruns, POS2 installation, or reinstall will always materialize exactly one canonical row.

The operator requests a final seed-flow audit to guarantee:

```text
one tenant
-> exactly one canonical TblSetupWeird row
-> Round Down Tips default = disabled
-> UI item 1 unchecked
-> UI item 2 checked
-> seed creates no TblLocalOutbox row
```

## Existing decisions that must remain authoritative

Read `report/report082.md`, `report/report083.md`, and all local-only evidence from prompts 082–083 before editing.

Preserve these contracts:

```text
- TblSetupWeird is tenant-scoped, not POS-scoped.
- SourceClientId is transport/envelope metadata only.
- UI has exactly two mutually exclusive choices projected from one persisted boolean.
- Authority model is A: LOCAL_SEED_NO_OUTBOX.
- Default is disabled.
- Existing operator-owned value must be preserved.
- Seed rerun must be idempotent.
- Same-tenant duplicate rows are a conflict, not an automatic merge/delete.
- Operator UI edits create normal outbox updates.
- Inbound bootstrap/pull applies without local outbox echo.
```

Do not add `SourceClientId` to `TblSetupWeird`.

## Scope

This task is a final audit and hardening of the future-install seed flow only.

1. Prove there is exactly one active seed/materialization path for `TblSetupWeird`.
2. Prove clean install inserts at most one tenant row.
3. Prove the inserted value is disabled.
4. Prove rerun, second POS, and reinstall do not create duplicates or reset an existing value.
5. Prove seed creates zero `TblLocalOutbox` rows.
6. Prove marker and setting row are atomic.
7. Correct code/tests only if direct evidence shows a defect.
8. Do not mutate the operator's current database automatically.
9. Do not test checkout/payment.

## Mandatory documentation gate

Read before source/test/doc edits:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report082.md
report/report083.md
```

Read the complete local evidence artifacts from prompts 082 and 083.

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
EVIDENCE_MODE=FINAL_SEED_FLOW_AUDIT
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase A — inventory every seed path

Search the complete WPF installation/Phase2 seed source for all references to:

```text
TblSetupWeird
Weird Tip
Round Down Tips
IsRoundTipEnabled
phase2-reference-driven-trial-v004-weird-tip-local-default
```

Return privately a table:

| Seed source/path | Can insert TblSetupWeird | Can update existing row | Can create outbox | Active in canonical flow | Action |
|---|---:|---:|---:|---:|---|

The final active architecture must have exactly one canonical writer for the future-install default.

Classify any additional writer as:

```text
legacy inactive
runtime operator edit
inbound apply
test-only
accidental duplicate seed path
```

If two active seed writers can both insert the row, fix the duplication before declaring PASS.

## Phase B — prove exact seed branch behavior

Return the complete actual C# method body, repository-relative path, and line range for the v004 Weird Tip materialization logic.

The method must follow this logical contract inside one transaction:

```text
resolve tenant
-> query canonical active TblSetupWeird rows for that tenant

count == 0:
  create exactly one row
  Round Down Tips = false
  write v004 marker
  create zero outbox rows
  commit

count == 1:
  preserve existing row and value unchanged
  write marker only if missing and contract requires it
  create zero outbox rows
  commit/no-op as appropriate

count > 1:
  return explicit duplicate conflict
  write no marker
  change no rows
  create zero outbox rows
  rollback
```

Prove the exact predicates used to define the canonical row, including tenant and any active/deleted/status predicate. Do not use `SourceClientId` as a row key or filter.

## Phase C — prove default projection

Prove from active source/model/schema that the seeded value projects to:

```text
Round Down Tips / Enable = unchecked
Do Not Round Down Tips / Disable = checked
```

The persisted seed value must be exactly the disabled boolean representation used by runtime and receiver apply.

Verify there is no inversion such as:

```csharp
runtimeValue = !persistedValue;
```

unless the active schema name is semantically inverted and the mapping is proven. Report the exact mapping privately.

## Phase D — prove zero seed outbox

Audit the v004 seed executor and every helper it calls.

Prove:

```text
TblLocalOutbox rows added by clean v004 seed = 0
TblLocalOutbox rows added by v004 rerun = 0
TblLocalOutbox rows added when adopting existing true = 0
TblLocalOutbox rows added on duplicate conflict = 0
```

There must be no implicit generic repository/outbox helper invoked by the seed.

Operator UI edits remain outside this seed path and must still produce exactly one normal update outbox when the value changes.

## Phase E — marker and transaction atomicity

Prove the v004 marker and the optional new `TblSetupWeird` row are committed atomically.

Required behavior:

```text
row insert succeeds + marker succeeds -> commit both
row insert succeeds + marker fails -> rollback row
marker exists + row missing -> do not silently claim success; return explicit inconsistent-state result or repair plan
```

Return the exact transaction begin/SaveChanges/commit/rollback code and the exact SaveChanges count.

Do not use two independent transactions.

## Phase F — disposable PostgreSQL final proof

Use the approved disposable PostgreSQL harness. Do not write to the operator's current DB.

Required test matrix:

### Clean tenant

```text
before rows = 0
run v004
rows after = 1
stored disabled = true as a semantic assertion
item 1 checked = false
item 2 checked = true
v004 marker count = 1
seed outbox count = 0
```

### Seed rerun

```text
run v004 again
rows remain = 1
marker remains = 1
value unchanged
outbox remains = 0
```

### Existing operator true

```text
precreate exactly one canonical row with Round Down Tips enabled
run v004
rows remain = 1
true remains true
outbox = 0
```

### Second POS/reinstall simulation

```text
same tenant identity, different local POS/station identity
run v004
rows remain = 1
operator value unchanged
outbox = 0
```

### Duplicate same-tenant rows

```text
precreate two canonical rows
run v004
explicit conflict
rows unchanged
marker not falsely advanced
outbox = 0
```

### Marker failure

```text
force marker write failure
new setup row rolls back
outbox = 0
```

### Concurrent seed attempt

When the schema/harness permits, run two seed attempts for the same tenant concurrently or simulate the relevant race.

Final result must still be:

```text
one canonical row maximum
```

If the schema lacks a database uniqueness constraint and concurrency can create duplicates, do not declare final PASS. Return:

```text
BLOCKED_WEIRD_TIP_SEED_CONCURRENCY_GUARD_MISSING
```

and prepare an additive operator-reviewed schema guard or a serializable/advisory-lock solution without applying it to the current DB.

## Phase G — current DB read-only sanity check

The operator manually deleted one duplicate row.

When credentials are available through an approved read-only method, verify only aggregate current state:

```text
current-tenant canonical row count = 1
UI projection resolves to exactly one checked option
```

Do not print raw IDs, payloads, or business values.

If non-interactive credentials are unavailable, provide an operator-run read-only command/script. Do not weaken credential controls and do not block the seed code audit solely on this optional current-DB check.

## Tests

Add/retain focused tests covering:

```text
exactly one active seed writer
clean install creates one row
seed default is disabled
UI projection item1=false item2=true
rerun idempotency
existing true preserved
second POS/reinstall preserves value
duplicate conflict
marker rollback
zero seed outbox in every branch
operator edit outbox remains separate
inbound apply no echo
no SourceClientId row scoping
concurrency/uniqueness protection
```

Run WPF build and focused Phase2/Weird Tip seed tests only. Do not include unrelated migration suites.

## Source-change boundary

Allowed changes:

```text
v004 Phase2 seed manifest/executor/constants
direct Weird Tip seed helper
focused seed tests
disposable integration harness
canonical seed docs/current task/result
```

Do not change:

```text
checkout/payment
Employee Weight
TurnEngine policy
Weird Tip operator-save semantics already proven by prompt082
receiver apply unless a direct seed regression is proven
other Phase2 seed tables
```

No OBM source commit/push.

## Private handoff requirements

Return directly to the operator:

1. Exact count of active seed writers.
2. Complete v004 seed method body and call chain.
3. Exact default field/value and UI projection.
4. Exact one-row predicates.
5. Exact transaction and marker boundary.
6. Exact proof that seed creates zero outbox.
7. Disposable PostgreSQL test matrix and output.
8. Concurrency/uniqueness proof.
9. Current DB aggregate sanity result if available.
10. Build/test counts.
11. Every source file changed locally.
12. Clear statement that the current DB was not automatically mutated.

## Public report

Create and push only an ultra-minimal:

```text
report/report084.md
```

It may contain only:

```text
verdict
single active seed writer yes/no
clean seed single row yes/no
default disabled yes/no
rerun/multi-POS preservation yes/no
seed outbox zero yes/no
marker atomicity yes/no
concurrency guard proven yes/no
build/test counts
current DB automatically mutated no
checkout tested no
one aggregate evidence SHA-256
```

Do not include source paths, schema/table/column names, DB names, raw counts from the operator DB, or internal code in the public report.

## Valid verdicts

```text
OBM_POS_WEIRD_TIP_SEED_SINGLE_ROW_DEFAULT_DISABLED_VERIFIED
```

```text
BLOCKED_WEIRD_TIP_MULTIPLE_ACTIVE_SEED_WRITERS
```

```text
BLOCKED_WEIRD_TIP_SEED_CONCURRENCY_GUARD_MISSING
```

```text
BLOCKED_WEIRD_TIP_SEED_DEFAULT_MAPPING_AMBIGUOUS
```

```text
BLOCKED_WEIRD_TIP_SEED_BUILD_OR_TEST
```
