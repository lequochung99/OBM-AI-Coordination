# Prompt 081 — Audit and correct Weird Tip Setup sync identity, SourceClientId ownership, and duplicate-row behavior

## Operator physical evidence

The operator has physically confirmed:

```text
Employee Weight Save succeeds.
The corresponding recent edits are visible in TblLocalOutbox.
```

Therefore the Employee Weight local-save-to-outbox path is physically functioning and the operator is moving to the next discovered setup/sync issue.

In Advanced Settings, the `Wreid - Tip - Setup` / Weird Tip Setup tab displays the `Rounds Down Tips` configuration. The operator observed:

```text
TblSetupWeird does not contain SourceClientId.
```

The current UI also visibly shows two apparently identical `Rounds Down Tips` rows. Audit whether these are legitimate distinct records or duplicate logical setup rows.

## Critical rule — do not add SourceClientId blindly

First determine where source identity is canonically owned for this entity:

```text
A. TblSetupWeird business row
B. TblLocalOutbox envelope only
C. DTO/payload only
D. both entity row and envelope
E. another proven source
```

`SourceClientId` must not be added to the table merely because other synced tables contain it.

If the outbox envelope already carries source identity and the receiver uses it for echo suppression/idempotency, the entity table may intentionally omit the column.

If active source/receiver/conflict-resolution logic requires `TblSetupWeird.SourceClientId`, then prove the schema gap and prepare the smallest additive correction.

## Canonical source identity shape

For a WPF POS-originated mutation, the canonical source identity shape is expected to be derived from runtime station identity, for example:

```text
POS:{PosGuid:D}
```

Do not hard-code POS1, any GUID, tenant, or device value.

Prove the actual project contract before implementing.

## Scope

Audit and, only when proven necessary, correct:

1. Weird Tip Setup local load/save behavior.
2. `TblSetupWeird` schema/model/DTO mapping.
3. Local outbox creation and envelope metadata.
4. Sender payload and receiver/apply mapping.
5. Echo suppression/idempotency/conflict behavior across POS stations.
6. The apparent duplicate `Rounds Down Tips` rows in the UI.
7. Future-install seed/materialization behavior for this setup.

Do not test checkout/payment.
Do not alter Employee Weight code.
Do not create a second sync framework.
Do not mutate the current DB automatically.

## Mandatory documentation/evidence gate

Read completely before source edits:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report078.md
report/report079.md
```

Also read local-only evidence from the Employee Weight save/outbox boundary only to understand the established outbox contract. Do not modify that flow.

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
EVIDENCE_ESCALATION=DIRECT_SCHEMA_AND_SYNC_PROOF
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase A — exact Weird Tip UI and persistence call chain

Locate the real tab/control/window and map:

```text
Open Wreid/Weird Tip Setup
-> resolve current tenant/POS
-> query TblSetupWeird
-> project UI rows
-> checkbox edit
-> Save/update command
-> local transaction
-> TblLocalOutbox enqueue
-> UI reload
```

Return privately the complete relevant C# method bodies with repository-relative paths and line ranges for:

```text
UI load
UI checkbox edit/save
service/repository update
transaction boundary
outbox creation
DTO mapping
receiver/apply handler
```

Do not provide only high-level prose.

## Phase B — schema and identity ownership matrix

Inspect active EF model and both current target/reference schemas read-only.

For `TblSetupWeird`, provide privately:

| Concept | Entity property | DB column/type | Nullable/default | Required | Owner |
|---|---|---|---|---:|---|
| Primary key | | | | | |
| Tenant identity | | | | | |
| POS/station scope if any | | | | | |
| Round-down flag | | | | | |
| SourceClientId | | | | | |
| UpdatedAt/version | | | | | |
| Active/deleted status | | | | | |

Also map the outbox envelope:

| Outbox concept | Column/property | Populated from | Receiver use |
|---|---|---|---|
| TenantGuid | | | |
| Entity name/type | | | |
| Entity/business key | | | |
| Operation | | | |
| TransactionGuid | | | |
| Sequence | | | |
| OccurredAt | | | |
| SourceClientId | | | |
| Payload | | | |
| Idempotency/delivery key | | | |

Answer explicitly:

```text
Does TblLocalOutbox already contain SourceClientId for a TblSetupWeird update?
Does the payload contain SourceClientId?
Does the receiver require SourceClientId on the entity row?
Does apply use envelope SourceClientId to avoid sending the same change back?
```

## Phase C — determine entity scope

Prove whether Weird Tip Setup is:

```text
tenant-wide store setting
POS-station-specific setting
device-specific setting
global/default setting
```

The UI wording suggests a shop-level tip-rounding rule, but source/schema evidence is authoritative.

Required consequences:

### Tenant-wide

```text
one logical active row per TenantGuid
all POS stations consume the same setting
source POS is transport/audit metadata, not business scope
```

### POS-specific

```text
one row per TenantGuid + PosGuid/PosStationId
source identity must not be confused with business station scope
```

Do not use SourceClientId as a substitute for a missing business-scope key.

## Phase D — investigate the two visible rows

The UI currently shows two visually identical `Rounds Down Tips` rows.

Determine whether this is caused by:

```text
TWO_DISTINCT_CANONICAL_OPTIONS
DUPLICATE_DB_ROWS
DUPLICATE_SEED
DUPLICATE_UI_BINDING
TENANT_FILTER_MISSING
ACTIVE_AND_INACTIVE_ROWS_BOTH_SHOWN
LEGACY_AND_CURRENT_ROWS_MERGED
OTHER_PROVEN_CAUSE
```

Use read-only counts and key/equality booleans. Do not expose raw GUIDs or business identifiers in the public report.

If the rows are duplicates:

```text
- correct the future-install seed/idempotency contract;
- correct UI query/binding so one logical setting appears once;
- prepare, but do not automatically apply, a versioned operator-reviewed repair for the current DB;
- never delete a row without proving which row is canonical and whether references/outbox history exist.
```

If both rows are legitimate, correct the labels so the operator can distinguish them.

## Phase E — SourceClientId decision

Choose exactly one evidence-based result:

```text
SOURCECLIENTID_ENVELOPE_ONLY_CORRECT
SOURCECLIENTID_ENTITY_COLUMN_REQUIRED
SOURCECLIENTID_DTO_ONLY_REQUIRED
SOURCECLIENTID_CURRENT_CONTRACT_INCONSISTENT
```

### If envelope-only is correct

Do not add a DB column.

Prove:

```text
local save creates outbox with SourceClientId
receiver uses envelope metadata
entity row remains tenant/POS scoped through its true business keys
no echo loop
```

Add comments/docs/tests to prevent future AI from assuming every synced entity needs a SourceClientId column.

### If an entity column is required

Prepare an additive, versioned schema/source correction:

```text
nullable-first or proven non-null strategy
backfill derived from canonical runtime/outbox evidence
EF mapping
DTO mapping
save/apply mapping
index/constraint only when proven
future-install schema/seed update
```

Do not apply the migration to the current DB automatically.

Do not invent SourceClientId for historical rows when provenance is ambiguous. Return an operator-review blocker for those rows.

## Phase F — save + outbox atomicity

The canonical Weird Tip Save must be:

```text
commit UI edit
-> validate
-> resolve tenant/business scope
-> begin one local transaction
-> update exactly one canonical TblSetupWeird row
-> create exactly one canonical outbox update
-> one SaveChangesAsync when possible
-> commit
-> reload
```

If outbox infrastructure is unavailable:

```text
save fails
transaction rolls back
```

No silent `return 0` or local-only commit is allowed.

No-op Save:

```text
zero entity writes
zero outbox writes
```

## Phase G — sync receiver and echo-loop proof

Trace the complete path:

```text
TblSetupWeird local edit
-> TblLocalOutbox
-> DTO/payload
-> API/SignalR delivery
-> WPF receiver/apply
-> target TblSetupWeird update
```

Prove:

```text
idempotent apply
correct row key
correct tenant/POS scope
SourceClientId handling
no apply-to-outbox echo loop
no duplicate update when the same delivery is retried
```

If receiver/apply support is missing, return a blocker instead of claiming sync complete.

## Phase H — future-install seed/materialization

Audit how `TblSetupWeird` is created during clean installation.

Required seed policy:

```text
minimal
idempotent
one logical row per proven scope
no runtime/outbox rows seeded
no duplicate Rounds Down Tips rows
```

Determine the product default for round-down tips only from existing canonical source/reference evidence.

If no universal default is proven:

```text
seed disabled/neutral only when schema requires a row
OR seed no row and let UI create it
```

Choose the behavior that active source actually supports. Do not copy a salon-specific value into every tenant.

## Tests

Add focused tests for at least:

```text
correct tenant/POS scope
one logical UI row
load current value
edit/save/reload
no-op save
entity + outbox atomic transaction
outbox unavailable rolls back
SourceClientId location contract
sender payload/envelope metadata
receiver/apply idempotency
echo suppression
retry does not duplicate row/outbox
future-install seed rerun does not duplicate TblSetupWeird
```

Use a disposable real PostgreSQL schema for save/outbox/apply proof when an approved harness exists.

Do not write to the operator's current DB automatically.

## Physical retest package

Provide the operator exact steps to verify:

```text
1. Rebuild and open Weird Tip Setup.
2. Confirm expected logical row count.
3. Toggle one approved setting.
4. Save once.
5. Reload and verify persistence.
6. Verify one matching TblLocalOutbox update using safe aggregate/equality output.
7. Save again without changes and verify no new outbox.
8. Verify a second POS/apply harness receives the value when safe infrastructure exists.
```

Do not test checkout/payment yet.

## Safety boundaries

```text
No automatic mutation of current target/reference DB
No automatic WPF launch/click
No checkout/payment test
No Employee Weight source changes
No raw credentials/GUIDs/payload/business rows in public report
No OBM source commit/push
```

## Evidence and public reporting

Create a versioned local-only evidence artifact, for example:

```text
WeirdTipSetupSyncIdentityV001
```

The private handoff must contain exact schema/source/mapping/call-chain proof and any required C# blocks.

Create and push only an ultra-minimal:

```text
report/report081.md
```

Public report may contain only:

```text
verdict
entity scope proven yes/no
SourceClientId ownership decision category
apparent duplicate-row cause category
save/outbox/apply boundary proven yes/no
future-install seed corrected yes/no
build/test counts
current DB automatically mutated no
checkout tested no
one aggregate SHA-256
```

Do not include internal paths, table/column details, source symbols, database names, counts from operator DB, GUIDs, payloads, or credentials.

## Valid verdicts

```text
OBM_POS_WEIRD_TIP_SETUP_SYNC_IDENTITY_READY_FOR_PHYSICAL_RETEST
```

```text
OBM_POS_WEIRD_TIP_SETUP_SCHEMA_REPAIR_READY_FOR_OPERATOR_REVIEW
```

```text
BLOCKED_WEIRD_TIP_SOURCECLIENTID_CONTRACT_AMBIGUOUS
```

```text
BLOCKED_WEIRD_TIP_DUPLICATE_ROW_REPAIR_AMBIGUOUS
```

```text
BLOCKED_WEIRD_TIP_SYNC_APPLY_CONTRACT_MISSING
```

```text
BLOCKED_WEIRD_TIP_BUILD_OR_TEST
```
