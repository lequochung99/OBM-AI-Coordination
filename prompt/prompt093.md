# Prompt 093 — Implement Price Rule CRUD outbox, receiver apply, and policy dependency ordering

## Authoritative evidence from prompt092

Read `report/report092.md` and the complete local artifact:

```text
E:\Project2026\RecoveryReports\PriceRuleSaveSyncCodeHandoffV001
```

The prompt092 private handoff proves:

```text
Root cause of OutboxRowsStaged=0 = PRICE_RULE_OUTBOX_CALL_MISSING
Price Rule outbox method called = no
No exception was swallowed
Current Price Rule service has no explicit transaction
Current Save stages TblTurnAmountRule only
SaveChangesAsync commits rule rows without TblLocalOutbox
Price Rule DTO/mapper/receiver registration are not proven
WPF inbound dispatch has no TblTurnAmountRule case
```

The local Price Rule Save itself now physically passes after the timestamp correction:

```text
NewRows=1
ValidatedRows=1
OutboxRowsStaged=0
Committed=True
ReloadSucceeded=True
```

This task must close only the missing synchronization and transaction boundary. Do not revisit Load/ListView or the resolved timestamp defect.

## Operator escalation — mandatory code-level handoff

The operator explicitly requires detailed C# evidence because several prior high-level reports looked similar while physical behavior remained incorrect.

Use:

```text
EVIDENCE_ESCALATION=100_PERCENT_DIRECT_CODE_AND_RUNTIME_PROOF
```

The private handoff must contain complete actual C# method bodies before and after, repository-relative paths, line ranges, schema/DTO mappings, transaction ownership, exact tests, and unified diffs.

A verdict and test counts alone are unacceptable.

## Critical hidden dependency discovered in the current code

The current Price Rule save calls:

```csharp
turnPolicySetupService.GetOrCreateDraftPolicyAsync(tenantGuid)
```

and every `TblTurnAmountRule` is written with:

```text
TurnPolicyGuid = policy.TurnPolicyGuid
```

Therefore a Price Rule outbox event may reference a Draft Policy that does not yet exist on another POS.

Do not add rule outbox events until the policy dependency is proven and ordered correctly.

Before implementation, audit and choose exactly one policy model based on canonical documentation and active source:

### Model P1 — EXISTING_DRAFT_REQUIRED

```text
Price Rule editor may open/load with No Draft
Price Rule Save requires an existing operator-owned Draft Policy
No hidden policy creation from the child editor
If no Draft exists, Save returns a clear non-mutating result
```

### Model P2 — ATOMIC_DRAFT_CREATION_WITH_SYNC

Allowed only if canonical docs prove that clicking Save in the Price Rule child is an authorized way to create the first Draft Policy.

Required boundary:

```text
create Draft Policy
-> stage TblTurnPolicy outbox I event
-> stage Price Rule outbox I/U/D events
-> same transaction group
-> policy event ordered before rule events
-> one atomic local transaction
```

Do not preserve the current hidden `GetOrCreateDraftPolicyAsync` behavior without selecting and proving P1 or P2.

If the policy contract cannot be resolved, return:

```text
BLOCKED_PRICE_RULE_POLICY_DEPENDENCY_CONTRACT
```

## Architecture boundaries

```text
WPF runtime = direct Windows
Database provider = PostgreSQL/Npgsql only
Docker required = no
Initialization model = A, no automatic Price Rule seed
Checkout/payment = out of scope
TurnEngine calculation = out of scope
Turn Policy activation = out of scope
```

Do not reintroduce SQL Server providers, packages, config, fallback contexts, or tests.

Do not automatically mutate the operator's current DB. Later operator-triggered WPF saves are the only authorized current-DB mutations.

Do not commit or push OBM source.

## Mandatory documentation gate

Read completely before editing:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report086.md
report/report087.md
report/report088.md
report/report091.md
report/report092.md
prompt/prompt092.md
```

Read every file in the local prompt092 evidence artifact, including:

```text
PRIVATE_HANDOFF.md
BEFORE_CODE.md
AFTER_CODE.md
SCHEMA_DTO_MAPPING.md
TRANSACTION_BOUNDARY.md
UNIFIED_DIFF.patch
TEST_OUTPUT.txt
```

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
RUNTIME_MODE=DIRECT_WINDOWS_WPF_LOCAL_POSTGRESQL
DOCKER_REQUIRED=NO
EVIDENCE_ESCALATION=100_PERCENT_DIRECT_CODE_AND_RUNTIME_PROOF
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
SelectedPolicyModel=P1_OR_P2
```

## Required new local evidence artifact

Create a new versioned local folder, never overwrite prompt092 evidence, for example:

```text
PriceRuleCrudSyncImplementationV001
```

Required files:

```text
PRIVATE_HANDOFF.md
POLICY_DEPENDENCY.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SCHEMA_DTO_MAPPING.md
TRANSACTION_BOUNDARY.md
SENDER_RECEIVER_CALL_CHAIN.md
TEST_OUTPUT.txt
SHA256SUMS.txt
```

Do not push these detailed files to GitHub.

## Phase A — prove policy dependency before rule sync

Return privately:

1. Exact `TblTurnPolicy` PK, tenant key, status/version fields, and Price Rule FK relationship.
2. Whether PostgreSQL enforces the Price Rule to Policy FK.
3. Current `TblTurnPolicy` outbox sender support.
4. Current API/relay support for `TblTurnPolicy`.
5. Current WPF inbound dispatch/apply support for `TblTurnPolicy`.
6. Whether `GetOrCreateDraftPolicyAsync` writes through the same DbContext/transaction as Price Rule Save.
7. Whether it creates an outbox event.
8. Whether another POS can receive a Price Rule before its parent policy exists.
9. Selected P1 or P2 model and exact justification from canonical docs/source.

If P2 is selected, policy creation and rule creation must share one explicit transaction and one ordered outbox transaction group.

If P1 is selected, replace hidden get-or-create with a non-mutating Draft lookup and a clear result such as:

```text
PRICE_RULE_DRAFT_POLICY_REQUIRED
```

Do not silently create a policy.

## Phase B — choose and prove the Price Rule payload contract

The prompt092 artifact found no proven Price Rule DTO or mapper.

Audit working sync entities and choose one proven model:

### Payload model E — direct entity payload

Allowed only if the existing generic outbox/relay convention safely serializes `TblTurnAmountRule` and receiver apply can deserialize it without exposing navigation objects or provider-only state.

### Payload model D — typed DTO

Create the smallest typed DTO and bidirectional mapper when direct entity payload is not already a proven convention.

Return the selected payload model and complete C# code.

The payload contract must include every required field:

```text
TurnAmountRuleGuid
TenantGuid
TurnPolicyGuid
RuleName
MinAmount
MaxAmount
Factor1
Factor2
TurnCredit
SortOrder
IsActive
Notes
CreatedAt
CreatedBy
UpdatedAt
UpdatedBy
```

Prove serialization of default/clear values:

```text
IsActive=false
SortOrder=0 when contract permits
MaxAmount=null
TurnCredit=null
Notes=null or empty clear operation
```

Do not allow default-value omission to preserve stale receiver values.

All PostgreSQL local timestamp fields must use the proven local-timestamp normalization contract.

## Phase C — true change-set detection

The current diagnostic save counts every existing row as dirty. Replace this with a real final change set.

Required categories:

```text
Inserted rules
Truly updated rules
Deleted rules
Unchanged rules
```

Use original snapshots or compare the proposed final set against reloaded DB entities.

Required behavior:

```text
No changes
-> PRICE_RULE_SAVE_NO_CHANGES
-> rule writes = 0
-> outbox writes = 0

One insert
-> I event = 1

One true update
-> U event = 1

One delete
-> D event = 1
```

Do not emit U events for unchanged rules merely because they were present in the grid.

For deletes, construct the canonical delete payload/key before marking/removing the entity.

## Phase D — canonical atomic sender boundary

Implement one clearly owned Price Rule save method.

Required order:

```text
Commit focused DataGrid binding/cell/row
-> calculate inserted/updated/deleted/unchanged
-> no changes returns without DB/outbox writes
-> validate entire final rule set
-> resolve current tenant and selected policy model
-> verify outbox infrastructure is available before mutation
-> create one DbContext
-> begin explicit PostgreSQL transaction
-> reload current tenant/policy rules
-> revalidate keys/scope/concurrency
-> stage inserts/updates/deletes
-> build canonical payload for every changed rule
-> stage one TblLocalOutbox CRUD event per changed rule
-> when P2 applies, stage parent policy event before rule events
-> verify expected outbox count equals expected changed entity event count
-> one SaveChangesAsync when possible
-> commit once
-> reload
```

Canonical Price Rule event model is:

```text
one CRUD event per changed rule
I = insert
U = true update
D = delete
EntityType = nameof(TblTurnAmountRule)
EntityGuid = TurnAmountRuleGuid
```

Use one transaction GUID and deterministic sequence order for all events in one operator Save.

When P2 applies:

```text
sequence 1 = parent policy I/U event
sequence 2..N = Price Rule CRUD events
```

Required fail-closed behavior:

```text
changed rule count > 0 AND staged Price Rule outbox count == 0
-> PRICE_RULE_OUTBOX_NOT_STAGED
-> no SaveChangesAsync
-> rollback

outbox factory/repository unavailable
-> explicit failure
-> rollback

payload serialization fails
-> rollback

SaveChanges fails
-> rollback rule + outbox
```

A successful changed-rule Save must never return:

```text
Committed=True
OutboxRowsStaged=0
```

Prefer exactly one `SaveChangesAsync` for rule and outbox additions inside the explicit transaction.

A reload failure after commit must return:

```text
Committed=True
ReloadSucceeded=False
PRICE_RULE_SAVED_RELOAD_FAILED
```

and must not be reported as an uncommitted Save failure.

## Phase E — API/relay and subscription proof

Audit whether the generic API/relay accepts arbitrary `EntityType` values or uses an allowlist/subscription registry.

Return actual code for:

```text
outbox publish
API ingest/store
subscription/entity registration
delivery creation
source-client exclusion/echo suppression
```

If `TblTurnAmountRule` requires registration, add only the smallest additive registration.

If `TblTurnPolicy` must be delivered before rules under P2, prove ordered delivery by transaction GUID and sequence.

Do not modify unrelated API entities or delivery behavior.

If relay support cannot safely carry Price Rules, return:

```text
BLOCKED_PRICE_RULE_API_RELAY_CONTRACT
```

## Phase F — WPF receiver/apply

Add a typed inbound dispatch case in the existing `PullDbApiBot.ApplyDeliveryAsync` path:

```text
case nameof(TblTurnAmountRule)
```

Do not create a second sync channel.

Receiver requirements:

```text
validate TenantGuid and stable rule key
validate current tenant
validate parent policy dependency
I/U idempotent upsert by TenantGuid + TurnAmountRuleGuid
D idempotent delete/deactivate according to active contract
preserve decimal precision
normalize timestamp kinds for PostgreSQL local timestamp columns
apply false/0/null clear values correctly
no duplicate on replay
no local TblLocalOutbox echo
refresh/invalidate current tenant Price Rule runtime state
```

Do not overwrite `CreatedAt/CreatedBy` on an existing row unless the existing sync convention requires it.

For an incoming rule whose parent policy is missing:

```text
Do not silently insert an invalid orphan
Do not discard and acknowledge success
Return a retryable/deferred delivery result or enforce parent-policy-first ordering
```

Return the complete receiver method body and dispatch registration with repository-relative paths and line ranges.

## Phase G — runtime refresh

The prompt092 artifact did not identify a Price Rule cache refresh hook.

Audit whether Price Rule lookup uses:

```text
fresh DbContext query
singleton cache
static collection
TurnEngine service cache
window-local collection
```

After local Save and inbound apply, refresh or invalidate only the proven runtime owner.

Do not invent a cache service when runtime already queries the DB per calculation.

Return the exact decision and code.

## Phase H — tests and direct PostgreSQL proof

Docker is not used or required.

Run build and focused tests. Use the approved direct/local PostgreSQL harness when available; otherwise create an operator-run package using interactive `psql -W` without exposing credentials.

Prove at least:

```text
insert rule -> one rule + one I outbox commit
update rule -> one rule + one U outbox commit
delete rule -> rule removed/inactive + one D outbox commit
unchanged Save -> zero writes/outbox
multiple rule changes -> one transaction group with ordered sequences
outbox unavailable -> rule rollback
payload serialization failure -> rule rollback
SaveChanges failure -> rule/outbox rollback
wrong tenant rejected
receiver I replay -> one row
receiver U replay -> one updated row
receiver D replay -> final absent/inactive state
receiver no echo -> zero local outbox rows
false/0/null fields clear receiver values
parent policy dependency behavior under selected P1/P2
focused DataGrid cell committed before change detection
reload failure distinguished from commit failure
PostgreSQL-only provider guard remains PASS
```

Do not automatically write to the operator's current DB.

## Phase I — mandatory private C# handoff

Return directly to the operator:

1. Selected policy model P1 or P2 and complete proof.
2. Selected payload model E or D and complete proof.
3. Exact previous zero-outbox C# branch.
4. Complete BEFORE and AFTER Save button method.
5. Complete BEFORE and AFTER Price Rule save service.
6. Complete outbox staging helper.
7. Complete DTO/entity mapper or approved direct payload builder.
8. Complete API/relay registration code if changed.
9. Complete inbound dispatch and receiver apply method.
10. Complete runtime refresh code or proof that no cache exists.
11. Exact explicit transaction boundary diagram.
12. Exact `SaveChangesAsync` count.
13. Exact event count/operation/sequence behavior.
14. Unified diff for every changed method.
15. Test commands and full sanitized outputs.
16. Physical retest instructions.
17. Explicit list of unrelated paths unchanged.
18. Local artifact path and SHA-256 manifest.

Do not return only a summary or counts.

## Physical retest contract

After source/test proof:

```text
1. Stop all old WPF processes.
2. Rebuild and run direct Windows WPF.
3. Open the existing Price Rule.
4. Change exactly one non-sensitive field.
5. Save once.
6. Confirm:
   DirtyRows=1
   ValidatedRows=1
   OutboxRowsStaged=1
   Committed=True
   ReloadSucceeded=True
7. Verify one matching U event in TblLocalOutbox safely.
8. Save again without changes.
9. Confirm NO_CHANGES and zero new outbox.
10. Test one insert and one delete only after update passes.
11. Do not activate Turn Policy.
12. Do not test checkout/payment.
```

## Safety boundaries

- PostgreSQL/Npgsql only.
- Direct Windows WPF; Docker not required.
- No automatic mutation of the operator's current DB.
- No checkout/payment test.
- No automatic Price Rule seed.
- No unrelated TurnEngine calculation changes.
- No OBM source commit/push.
- No credentials, connection strings, raw GUIDs, raw payloads, or private rule values in the public report.

## Public report

Create and push only:

```text
report/report093.md
```

It may contain only:

```text
verdict
policy dependency model proven yes/no
Price Rule outbox sender proven yes/no
atomic rule/outbox proven yes/no
API/relay support proven yes/no
receiver apply proven yes/no
no-echo/idempotency proven yes/no
private C# handoff created yes/no
real PostgreSQL proof yes/no
build/test counts
current DB automatically mutated no
checkout tested no
one aggregate evidence SHA-256
```

## Valid verdicts

```text
OBM_POS_PRICE_RULE_CRUD_SYNC_READY_FOR_PHYSICAL_RETEST
```

```text
BLOCKED_PRICE_RULE_POLICY_DEPENDENCY_CONTRACT
```

```text
BLOCKED_PRICE_RULE_API_RELAY_CONTRACT
```

```text
BLOCKED_PRICE_RULE_RECEIVER_APPLY_CONTRACT
```

```text
BLOCKED_PRICE_RULE_REAL_POSTGRESQL_PROOF
```

```text
BLOCKED_PRICE_RULE_BUILD_OR_TEST
```
