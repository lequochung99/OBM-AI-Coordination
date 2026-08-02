# Prompt 083 — Add Weird Tip default-disabled setup to the future-install seed without cross-POS overwrite

## Operator decision

The operator has approved the following product default for future clean installations:

```text
Weird Tip / Round Down Tips
-> default = disabled
-> UI item 1 (Round Down Tips / Enable) = unchecked
-> UI item 2 (Do Not Round Down Tips / Disable) = checked
```

The setting must be part of the canonical future-install seed/materialization flow.

The operator also asks whether the seed should publish a synchronization event through `TblLocalOutbox`.

This prompt must make that decision from the real multi-POS sync contract. Do not blindly emit a normal update event from every new POS installation.

## Prompt082 findings to preserve

Read `report/report082.md` and all local-only prompt082 evidence before editing.

Prompt082 reports:

```text
- two-option invariant corrected;
- one tenant-scoped canonical persistence row proven;
- save + outbox boundary proven;
- WPF receiver/apply proven;
- future seed corrected;
- build/tests passed;
- current DB was not automatically mutated.
```

The public report does not state the exact future-seed storage and outbox semantics. This prompt must audit and lock those details.

## Critical multi-POS safety problem

`TblSetupWeird` is tenant-scoped, not POS-scoped.

A normal unconditional seed update from every new POS is unsafe:

```text
POS1 installed
-> seed false
-> owner later changes setting to true
-> true syncs tenant-wide

POS2 installed later
-> seed false
-> normal U outbox emitted
-> false may overwrite the owner-owned true setting
```

That behavior is prohibited.

A reinstall, replacement POS, or second POS installation must never reset an existing tenant-wide setting to the product default.

## Canonical seed requirements

For a genuinely new local tenant database with no canonical Weird Tip row:

```text
create exactly one tenant-scoped TblSetupWeird row
round-down enabled = false
item 1 unchecked
item 2 checked
```

For an existing canonical row:

```text
preserve the operator-owned value
never overwrite true with false
never rotate/recreate the row
```

Seed rerun:

```text
zero duplicate rows
zero value changes
zero duplicate outbox events
```

Same-tenant duplicate rows:

```text
return explicit conflict
no automatic merge/delete
```

Do not seed two rows for the two UI choices.

## Mandatory decision: seed/outbox authority model

Audit the real local seed, API apply, bootstrap/snapshot, and multi-POS contracts and select exactly one model.

### Model A — LOCAL_SEED_NO_OUTBOX

Use when a normal seed event could overwrite an existing tenant-wide setting or when API/bootstrap is authoritative.

```text
future local baseline creates one false row
seed creates no TblLocalOutbox row
operator edits create normal U outbox rows
API/bootstrap/pull may later replace the local default with the current canonical tenant value
```

### Model B — SEED_CREATE_IF_ABSENT_OUTBOX

Allowed only when the complete sender/API/receiver contract supports explicit create-if-absent semantics.

Required behavior:

```text
seed creates local false row + one seed-origin outbox event atomically
remote apply creates the canonical setting only when it does not already exist
remote existing value is never overwritten
replay is idempotent
second POS/reinstall receives acknowledged no-op when remote row exists
```

A normal unconditional `U` upsert that overwrites an existing remote row does not qualify.

### Model C — PLATFORM_CANONICAL_DEFAULT

Use when Platform/API creates the canonical tenant default and WPF should only materialize it from bootstrap/snapshot.

```text
Platform/API owns initial false default
WPF local baseline does not independently publish a default
local row is populated from canonical bootstrap/snapshot
operator edits use normal outbox
```

### Blocked

If authority/order cannot be proven, stop with:

```text
BLOCKED_WEIRD_TIP_SEED_SYNC_AUTHORITY_AMBIGUOUS
```

## Required audit

Trace the complete contracts:

```text
Phase 2/future-install seed
-> TblSetupWeird local materialization
-> seed marker/version
-> optional TblLocalOutbox event
-> API/relay apply
-> canonical remote state
-> second POS bootstrap/snapshot
-> WPF receiver/apply
-> no-echo handling
```

Return privately:

1. Exact current seed service/manifest and next safe version.
2. Exact canonical row key and tenant key.
3. Exact persisted boolean and default.
4. Whether the current prompt082 seed implementation creates a row or uses no-row-as-false.
5. Whether it currently emits outbox.
6. Exact current outbox operation code and apply semantics.
7. Whether API apply is insert-only, create-if-absent, update, or upsert.
8. Whether a later POS seed can overwrite a remote existing row.
9. Bootstrap/snapshot ordering relative to local baseline seed.
10. Recommended authority model with source evidence.

Do not expose private identifiers, credentials, raw payloads, or database names in the public report.

## Implementation requirements

### Common requirements

Regardless of selected authority model:

```text
- default false;
- exactly one logical tenant row;
- item 2 checked on first clean load;
- rerun idempotent;
- existing operator value preserved;
- no duplicate row;
- no checkout dependency;
- no SourceClientId entity column;
- no outbox echo on inbound apply;
- seed versioned and prior versions preserved.
```

### If Model A is selected

Implement/verify:

```text
seed inserts one false row only when absent
seed writes no outbox
normal operator Save writes exactly one U outbox
inbound/bootstrap current tenant value overrides the local product default when canonical remote data exists
```

Document explicitly that baseline seed is local materialization, not a business edit.

### If Model B is selected

Implement/verify:

```text
row + seed-origin outbox in one transaction
explicit SeedIfAbsent or insert-if-absent operation
stable idempotency key based on tenant + entity + seed contract version
remote existing value causes no-op, never overwrite
second POS/reinstall cannot reset the setting
seed rerun creates zero extra outbox
receiver apply is idempotent and no-echo
```

Do not reuse normal `U` semantics unless the receiver proves it will not overwrite existing state.

### If Model C is selected

Implement/verify:

```text
canonical false row created by Platform/API tenant provisioning
WPF future install consumes bootstrap/snapshot value
local baseline does not invent and publish a tenant-wide default
operator edits use normal U outbox
```

Keep the WPF UI valid while canonical data is pending: no-row displays item 2 checked in memory and does not mutate merely on open.

## Seed versioning

Use the next safe version in the existing Phase 2/future-install convention. Do not guess the version name before auditing the current latest seed marker.

Requirements:

```text
- preserve all previous seed versions and evidence;
- write a new explicit marker only after the canonical Weird Tip seed/materialization commits;
- marker rerun is idempotent;
- do not rewrite the current operator DB automatically;
- no seed outbox unless the selected authority model explicitly requires it.
```

## Transaction boundary

For local seed materialization:

```text
resolve tenant
-> begin transaction
-> verify canonical row count
-> insert false row only when absent
-> optional create-if-absent outbox according to selected model
-> write seed marker
-> one SaveChangesAsync when possible
-> commit
```

If any required component fails, rollback all seed changes and marker changes.

## Required disposable integration tests

Use a disposable PostgreSQL harness. Do not write to the operator's current DB automatically.

Test at least:

```text
1. clean tenant/no row -> one false row
2. UI projection -> item 1 false, item 2 true
3. seed rerun -> same row/value, no duplicate marker
4. existing local true -> preserved
5. same-tenant duplicate rows -> explicit conflict
6. second POS/reinstall scenario -> existing tenant-wide true is never overwritten by false
7. operator change false->true -> one normal U outbox
8. no-op operator save -> zero outbox
9. inbound apply -> idempotent, no echo
10. selected authority-model outbox behavior
11. outbox/marker failure -> complete rollback
```

If Model B is selected, additionally prove with real integration tests:

```text
remote absent -> create false once
remote true exists -> seed event no-op and true preserved
seed replay -> no duplicate remote/local row
```

If Model A is selected, additionally prove:

```text
seed outbox count = 0
operator edit outbox count = 1
canonical inbound true replaces local initial false without local echo
```

## Current DB policy

Do not automatically mutate the current operator DB.

The current DB may be inspected read-only to determine whether a row exists and what seed marker is present, but do not change the setting or insert a seed marker.

Physical testing of the setting remains operator-driven through the WPF UI.

## Build/tests

Run the WPF build and focused Weird Tip/seed/outbox/bootstrap/receiver tests.

Do not broaden into unrelated migration suites.

## Private handoff requirements

Return directly to the operator:

1. Selected authority model: A, B, or C.
2. Why it is safe for multi-POS/reinstall.
3. Exact current and new seed version/marker.
4. Exact default and UI projection.
5. Whether seed creates `TblLocalOutbox`, with a direct yes/no answer.
6. If yes, exact create-if-absent behavior and no-overwrite proof.
7. If no, exact point where canonical sync later enters.
8. Transaction and idempotency proof.
9. Disposable integration results.
10. Exact files changed locally.
11. Physical retest steps.

## Public report

Create and push only an ultra-minimal:

```text
report/report083.md
```

It may include only:

```text
verdict
future seed default-disabled yes/no
single-row/idempotent yes/no
selected authority model A/B/C
seed emits outbox yes/no
multi-POS overwrite prevented yes/no
build/test counts
current DB automatically mutated no
checkout tested no
one aggregate evidence SHA-256
```

Do not include source paths, schema details, table/column names beyond the generic status, database names, payloads, internal values, or credentials.

## Valid verdicts

```text
OBM_POS_WEIRD_TIP_DEFAULT_SEED_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_WEIRD_TIP_SEED_SYNC_AUTHORITY_AMBIGUOUS
```

```text
BLOCKED_WEIRD_TIP_SEED_MULTI_POS_OVERWRITE_RISK
```

```text
BLOCKED_WEIRD_TIP_SEED_BUILD_OR_TEST
```
