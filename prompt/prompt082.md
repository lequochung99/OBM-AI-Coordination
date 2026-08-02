# Prompt 082 — Canonical Weird Tip two-option invariant, single-row persistence, and WPF receiver apply

## Operator correction — authoritative behavior

The operator physically inspected the `Weird - Tip - Setup` tab and clarified the intended UI contract:

```text
There are exactly two checkbox choices.
Exactly one choice must always be checked.
If item 1 is checked, item 2 must be unchecked.
If item 2 is checked, item 1 must be unchecked.
```

This is equivalent to a radio-button group, but the operator wants the existing two-checkbox presentation retained unless the active UI framework proves that radio buttons are required for accessibility or correctness.

The current physical UI is wrong because both visible rows can appear checked and both labels appear equivalent.

## Prompt081 findings to preserve

Read `report/report081.md` and the complete local-only prompt081 evidence before editing.

Prompt081 established:

```text
- TblSetupWeird is tenant-scoped.
- SourceClientId belongs to the sync/outbox envelope; do not add SourceClientId to TblSetupWeird merely for transport identity.
- The apparent repeated rows involved a missing tenant filter.
- The WPF receiver/apply contract for Weird Tip is missing and blocks end-to-end sync proof.
```

Do not regress these findings.

## Canonical model to prove before implementation

The expected design is:

```text
one canonical TblSetupWeird row per tenant
+ one persisted boolean setting
+ two mutually exclusive UI choices projected from that boolean
```

Expected projection, subject to source/schema proof:

```text
Item 1: Round Down Tips / Enable
Item 2: Do Not Round Down Tips / Disable

persisted true  -> item 1 checked, item 2 unchecked
persisted false -> item 1 unchecked, item 2 checked
```

The two UI choices must not require two physical TblSetupWeird rows.

Before coding, verify the active schema/model/reference behavior. If active source proves that the two choices represent different semantics than Enable/Disable, preserve the proven meanings and report them privately. Do not guess labels or create a second business row.

If source/reference evidence contradicts the one-row/one-boolean model, stop with:

```text
BLOCKED_WEIRD_TIP_TWO_OPTION_CONTRACT_AMBIGUOUS
```

## Scope

1. Prove and implement the exactly-one-of-two UI invariant.
2. Fix current-tenant loading so other-tenant rows cannot appear as duplicate options.
3. Preserve one canonical tenant setup row.
4. Implement the missing WPF receiver/apply contract through the existing sync pipeline.
5. Ensure save + outbox are atomic.
6. Correct future clean-install seed/idempotency for this setup.
7. Do not test checkout/payment.

## Documentation gate

Read before edits:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report079.md
report/report081.md
```

Also read the complete local evidence artifacts produced by prompts 079 and 081.

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase A — prove the current data/UI model

Trace the complete current path:

```text
open Weird Tip tab
-> resolve current tenant
-> query TblSetupWeird
-> project UI rows/options
-> checkbox binding/events
-> save/update
-> TblLocalOutbox
-> sender DTO/payload
-> API/relay contract
-> WPF receiver/apply
-> runtime SetupWeird state refresh
```

Return privately:

1. Exact current table key and tenant key.
2. Current row count for the selected tenant and aggregate counts for other tenants, using read-only queries only.
3. Whether there is exactly one canonical row per tenant.
4. Exact persisted property used for round-down enablement.
5. Exact active meanings of any additional fields such as round type or precision.
6. Why the UI currently displays two equivalent checked rows.
7. Whether those rows are:
   - two tenant records accidentally loaded;
   - two static UI options incorrectly bound to the same value;
   - duplicate same-tenant data;
   - another proven cause.

Use read-only transactions. Do not print raw tenant IDs, row IDs, or business values in the public report.

## Phase B — exactly-one-of-two UI invariant

Build one small view model/state owner for the two choices. Do not bind a checkbox row directly to every database row.

Required invariant:

```text
Item1Checked XOR Item2Checked == true
```

Required behavior:

```text
load true:
  item 1 = checked
  item 2 = unchecked

load false or no row/default-disabled state:
  item 1 = unchecked
  item 2 = checked

operator checks item 1:
  item 1 = checked
  item 2 = unchecked

operator checks item 2:
  item 1 = unchecked
  item 2 = checked

operator clicks the currently checked item attempting to uncheck it:
  invariant remains valid; the UI must not allow both unchecked
```

Use `IsThreeState=false` or equivalent. Null/indeterminate is not a valid production state.

The labels must be distinct and operator-readable. Use the active bilingual terminology style. If the proven semantics are Enable/Disable, use labels equivalent to:

```text
Round Down Tips (Enable)
Do Not Round Down Tips (Disable)
```

Do not leave both rows labeled `Rounds Down Tips`.

The UI must display exactly two choices even though persistence uses one canonical row.

## Phase C — current-tenant query and duplicate handling

All normal load/save operations must resolve the current local tenant first and query only that tenant.

Expected normal state:

```text
current-tenant canonical row count = 0 or 1
```

If zero:

```text
UI uses disabled/default state in memory
item 2 checked
no row is created merely by opening the tab
```

If one:

```text
project the stored boolean into the two-option UI
```

If more than one same-tenant canonical row:

```text
show structured setup-data conflict
block Save
prepare a separate operator-reviewed repair plan
no automatic delete/merge in this prompt
```

Other-tenant rows must never appear in this tenant's UI.

## Phase D — save and outbox boundary

Audit whether this screen saves immediately on checkbox click or through an explicit Save action. Preserve the established operator interaction, but perform one logical save per final selection.

Do not execute two independent saves such as:

```text
check item 1
save item 1
uncheck item 2
save item 2
```

The canonical persistence operation is one boolean update.

Required boundary:

```text
final mutually exclusive selection
-> resolve current tenant
-> begin one local DB transaction
-> load/create exactly one canonical TblSetupWeird row
-> set the persisted round-down boolean
-> preserve/update only other proven setup fields
-> add exactly one canonical TblSetupWeird update to TblLocalOutbox
-> one SaveChangesAsync when possible
-> commit
-> refresh UI and runtime state
```

If outbox infrastructure is unavailable, rollback the setup change. Do not silently commit with zero outbox rows.

No-op selection must create zero DB writes and zero outbox rows.

`SourceClientId` remains envelope-only unless new direct schema evidence contradicts prompt081. Do not add a transport-only column to TblSetupWeird.

## Phase E — missing WPF receiver/apply contract

Implement the missing receiver/apply path using the existing generic inbound delivery framework.

Required behavior:

```text
incoming TblSetupWeird update
-> validate tenant/envelope identity
-> deserialize existing DTO/payload
-> locate the current-tenant canonical TblSetupWeird row
-> idempotent insert/update as required
-> apply the persisted round-down boolean and other proven fields
-> save locally without producing a new TblLocalOutbox echo
-> refresh runtime SetupWeird state immediately
```

Do not create a second sync channel.

The receiver must be idempotent:

```text
same delivery repeated
-> same final row state
-> no duplicate row
-> no local outbox echo
```

Preserve source identity in the envelope for audit/echo suppression.

Verify the runtime assignment is not inverted. The active runtime value must correspond directly to the persisted round-down-enabled boolean unless source evidence proves a differently named semantic. Do not reintroduce a `!` inversion.

## Phase F — future clean-install seed

The setting must begin in a valid exactly-one-selected state.

The product-safe default is expected to be disabled because the schema/default and operator notice describe the feature as optional. Prove this from current schema/source before implementation.

If proven, future clean installation must result in:

```text
one canonical tenant row
round-down enabled = false
UI item 1 unchecked
UI item 2 checked
```

Seed requirements:

```text
- tenant-scoped;
- exactly one row;
- idempotent rerun;
- no duplicate row;
- no outbox seed row;
- do not overwrite an operator-owned existing setting;
- preserve prior seed versions for audit.
```

If the canonical architecture prefers no setup row until explicit Save, preserve no-row storage but still show item 2 checked in memory. State the proven decision privately. Do not invent two seed rows.

## Tests

Add focused tests for at least:

```text
one canonical DB row projects to exactly two UI choices
true -> first checked, second unchecked
false -> first unchecked, second checked
no row -> disabled option checked, no mutation on open
checking first unchecks second
checking second unchecks first
clicking checked option cannot leave both unchecked
both checked state cannot persist
both unchecked state cannot persist
other-tenant rows excluded
duplicate same-tenant rows produce conflict, not silent merge
one selection change -> one entity update + one outbox update
no-op selection -> zero DB/outbox writes
outbox unavailable -> rollback setting
SourceClientId envelope-only contract
receiver idempotent apply
receiver creates no echo outbox
runtime state refreshed after local save and inbound apply
no inverted boolean mapping
future seed/no-row default is idempotent and never creates two rows
```

Use a real disposable PostgreSQL integration test for save/outbox/apply atomicity and receiver idempotency when an approved harness is available. Do not write to the operator's current DB automatically.

Run the WPF build and focused Weird Tip/setup/outbox/inbound-apply tests. Do not include unrelated migration suites.

## Physical retest

After PASS:

1. Rebuild and launch WPF manually.
2. Open Weird Tip Setup.
3. Confirm exactly two distinctly labeled choices.
4. Confirm exactly one is checked.
5. Select item 1; confirm item 2 immediately unchecks.
6. Select item 2; confirm item 1 immediately unchecks.
7. Attempt to uncheck the currently selected item; confirm one option remains selected.
8. Save according to the established UI action.
9. Close/reopen; confirm the same selection persists.
10. Confirm exactly one setup outbox update for the changed selection.
11. Apply/replay one inbound test delivery in the disposable harness and verify no duplicate/echo.
12. Do not test checkout/payment.

## Safety boundaries

- No automatic mutation of the operator's current DB.
- No automatic WPF launch/clicking.
- No checkout/payment test.
- No SourceClientId column added without direct contradictory evidence.
- No deletion/merge of same-tenant duplicate rows without separate operator approval.
- No OBM source commit/push; source changes remain local.
- No private identifiers, raw payloads, credentials, or internal code in the public report.

## Private handoff

Return directly to the operator:

1. Exact proven meaning of item 1 and item 2.
2. Exact root cause of both checked/equivalent rows.
3. Canonical one-row persistence model.
4. Tenant filter and duplicate-state proof.
5. Before/after checkbox state code.
6. Save + outbox transaction code/boundary.
7. WPF receiver/apply code and no-echo proof.
8. Future seed decision.
9. Build/test/integration results.
10. Physical retest steps.

## Public report

Create and push only an ultra-minimal:

```text
report/report082.md
```

It may contain only:

```text
verdict
two-option invariant corrected yes/no
single-row tenant persistence proven yes/no
save/outbox boundary proven yes/no
WPF receiver apply proven yes/no
future seed corrected yes/no
build/test counts
current DB automatically mutated no
checkout tested no
one aggregate evidence SHA-256
```

## Valid verdicts

```text
OBM_POS_WEIRD_TIP_TWO_OPTION_SYNC_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_WEIRD_TIP_TWO_OPTION_CONTRACT_AMBIGUOUS
```

```text
BLOCKED_WEIRD_TIP_DUPLICATE_TENANT_DATA_REPAIR_REQUIRED
```

```text
BLOCKED_WEIRD_TIP_RECEIVER_APPLY_CONTRACT
```

```text
BLOCKED_WEIRD_TIP_BUILD_OR_TEST
```
