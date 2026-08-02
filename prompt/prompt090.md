# Prompt 090 — Diagnose and fix Price Rule Save in the direct Windows WPF runtime

## Operator correction — authoritative runtime

OBM-POS WPF runs directly on Windows and connects directly to the local PostgreSQL service.

```text
No Docker container is involved in the WPF runtime.
Docker being stopped is irrelevant to reproducing the physical Price Rule Save failure.
```

The operator rebuilt and physically retested the current WPF source. `Price / Amount Rule Settings` loads correctly, `Add Row` works, but clicking `Save` still produces the same failure.

This prompt supersedes the Docker/disposable-endpoint assumptions from prompt089.

Do not wait for Docker. Do not use Docker as a blocker. Use the direct Windows WPF runtime and the existing local PostgreSQL configuration path.

## Current proven state

From reports 086–088:

```text
- WPF active runtime is PostgreSQL/Npgsql-only.
- SQL Server active provider paths were removed.
- Price Rule Load succeeds.
- Target Price Rule state is legitimately empty.
- Add Row creates an in-memory row.
- Physical Save still fails.
- Root cause, Save boundary, outbox, and receiver apply remain unproven.
```

The empty list is no longer the issue. The only current blocker is the real Save path.

## Scope

1. Instrument the actual Windows WPF Price Rule Save path safely.
2. Obtain the exact inner exception from one operator-triggered Save.
3. Fix the proven Save defect.
4. Preserve PostgreSQL-only runtime.
5. Complete atomic rule + outbox save.
6. Complete WPF receiver/apply and no-echo behavior when the existing sync contract permits it.
7. Do not test checkout/payment.
8. Do not automatically mutate the current DB; the operator's manual Save is the only authorized current-DB mutation.
9. Do not commit or push OBM source.

## Mandatory documentation/evidence gate

Read completely before editing:

```text
<WPF_ROOT>/AGENTS.md
<WPF_ROOT>/docs/refactoryInstallation/INSTALLATION_RUNTIME_CANONICAL.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_TASK.md
<WPF_ROOT>/docs/refactoryInstallation/CURRENT_RESULT.md
report/report086.md
report/report087.md
report/report088.md
```

Read all local Price Rule evidence from prompts 085–089.

Record locally:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
RUNTIME_MODE=DIRECT_WINDOWS_WPF_LOCAL_POSTGRESQL
DOCKER_REQUIRED=NO
EVIDENCE_ESCALATION=100_PERCENT_DIRECT_PHYSICAL_EXCEPTION
CanonicalDocVersion=<actual>
CanonicalDocSha256=<actual>
```

## Phase A — freeze and expose the real Save path

Before changing Save semantics, capture the complete current C# method bodies, repository-relative paths, and line ranges for:

1. Price Rule `Save` click handler.
2. DataGrid focused-cell/row commit helper.
3. New/dirty/deleted row detection.
4. Validation methods.
5. Tenant and policy/scope resolution.
6. Production Price Rule save service.
7. Insert/update/delete entity mapping.
8. Transaction begin/commit/rollback.
9. Outbox creation.
10. `SaveChangesAsync` calls.
11. Reload-after-save.
12. Exception/result-code mapping.

Do not provide isolated snippets. Include complete control-flow blocks in the private handoff.

## Phase B — add sanitized direct-runtime diagnostics

The current generic failure is insufficient. Add a safe diagnostic result contract to the Price Rule Save path.

Required safe fields:

```text
StageId
ResultCode
ProviderName
TenantResolved
PolicyScopeResolved
NewRows
DirtyRows
DeletedRows
ValidatedRows
InvalidRows
RuleEntityStates
OutboxRowsStaged
SaveChangesCallNumber
TransactionStarted
Committed
ReloadSucceeded
ExceptionType
InnerExceptionType
PostgreSqlState
SafeTable
SafeColumn
SafeConstraint
```

Do not display or write:

```text
rule values
raw rule IDs
raw tenant/policy GUIDs
connection string
password
username
raw payload
business-private notes
```

The diagnostic may be shown in the status line and written to a versioned local evidence file.

Use the real exception chain. Do not replace it with a generic message.

## Phase C — operator-assisted physical capture

After adding diagnostics and building successfully, return a private handoff with exact steps for the operator:

```text
1. Stop all old WPF processes.
2. Rebuild WPF.
3. Launch directly from Visual Studio/Windows.
4. Open Price / Amount Rule Settings.
5. Click Add Row.
6. Enter one valid minimal rule using the current UI validation contract.
7. Keep focus in the last edited cell.
8. Click Save exactly once.
9. Capture the full sanitized status/result code and local diagnostic artifact.
```

Expected intermediate verdict:

```text
PRICE_RULE_DIRECT_WINDOWS_DIAGNOSTIC_CAPTURE_READY
```

Do not claim the defect fixed until the operator-triggered exception is captured and explained.

## Phase D — root-cause classification

Classify the proven failure narrowly:

```text
GRID_EDIT_NOT_COMMITTED
NEW_ROW_NOT_MARKED_OR_KEY_NOT_GENERATED
VALIDATION_RESULT_NOT_PROPAGATED
MANDATORY_POLICY_FK_MISSING
TENANT_OR_POLICY_SCOPE_MISMATCH
DATE_TIME_KIND_MAPPING_MISMATCH
COLUMN_MAPPING_MISMATCH
NULLABILITY_OR_LENGTH_CONSTRAINT
DECIMAL_PRECISION_OR_SCALE_MISMATCH
ENTITY_TRACKING_CONFLICT
OUTBOX_DTO_OR_SERIALIZATION_FAILURE
OUTBOX_INFRASTRUCTURE_UNAVAILABLE
TRANSACTION_BOUNDARY_WRONG
SAVECHANGES_FAILURE_OTHER
RELOAD_MISREPORTED_AS_SAVE_FAILURE
```

Do not speculate. Support the classification with the exact exception, stack trace, entity states, schema mapping, and failing stage.

## Phase E — direct schema and mapping proof

Use read-only catalog queries against the current local PostgreSQL DB when credentials are already available through the WPF runtime or an operator-run read-only command. Do not request or print credentials.

For every editable field, prove:

| UI field | Entity property | DB column | C# type | PostgreSQL type | Nullable | Precision/length | Constraint | Outbox field |
|---|---|---|---|---|---:|---|---|---|
| Rule Name | | | | | | | | |
| Min Amount | | | | | | | | |
| Max Amount | | | | | | | | |
| Factor1 | | | | | | | | |
| Factor2 | | | | | | | | |
| Turn Credit | | | | | | | | |
| Sort Order | | | | | | | | |
| Active | | | | | | | | |
| Notes | | | | | | | | |
| Created/Updated timestamps | | | | | | | | |
| Tenant/policy keys | | | | | | | | |

Pay special attention to:

```text
- DateTime Kind vs timestamp without time zone;
- mandatory policy foreign key while the parent is No Draft;
- GUID/key generation for new rows;
- decimal precision/scale;
- null/open-ended Max Amount;
- default values omitted by payload serialization;
- required fields not represented in the UI.
```

## Phase F — canonical Save boundary

After proving the defect, implement one owned save method:

```text
commit focused DataGrid cell/row
-> classify new/dirty/deleted rows
-> if no changes: PRICE_RULE_SAVE_NO_CHANGES
-> validate the complete proposed final rule set
-> resolve current tenant and proven rule scope
-> begin one Npgsql transaction
-> reload current tenant rule set
-> apply inserts/updates/deletes
-> stage canonical outbox event(s) in the same DbContext
-> one SaveChangesAsync when possible
-> commit once
-> reload through canonical loader
```

Rules and outbox must commit or roll back together.

If outbox infrastructure is unavailable:

```text
explicit save failure
zero committed rule changes
```

If reload fails after commit:

```text
Committed=True
PRICE_RULE_SAVED_RELOAD_FAILED
```

Do not silently create or activate a Turn Policy.

If the schema requires a mandatory policy FK that conflicts with model A, return:

```text
BLOCKED_PRICE_RULE_MANDATORY_POLICY_SCOPE
```

with the exact schema/code proof and a proposed domain correction. Do not create a hidden Draft Policy.

## Phase G — sync receiver/apply

Trace the existing sync path:

```text
Price Rule local mutation
-> TblLocalOutbox
-> DTO/payload
-> relay/API contract
-> WPF inbound apply
-> local Price Rule table
```

Implement or prove:

```text
insert/update/delete
stable IDs
wrong-tenant rejection
idempotent replay
no duplicate rules
no local outbox echo
runtime rule refresh
```

Do not create a second sync framework.

If the DTO/relay contract is genuinely missing and cannot be safely added, return:

```text
BLOCKED_PRICE_RULE_SYNC_CONTRACT_MISSING
```

## Phase H — tests without Docker

Docker is not required.

Use the following proof layers:

1. Focused unit/contract tests.
2. EF/Npgsql schema-mapping tests using the approved existing local harness when available.
3. Operator-triggered direct WPF Save against the current local PostgreSQL DB for the final physical proof.

Do not automatically write test data to the operator's current DB.

If no disposable PostgreSQL harness is available, that does not block capturing and fixing the physical root cause. It only limits automated integration coverage; state that honestly.

Required focused tests:

```text
new row remains marked new through DataGrid commit
valid new row validation passes
invalid row blocks Save
focused-cell edit reaches row model
new key generated correctly
DateTime mapping correct for PostgreSQL
No Draft does not create a policy
outbox unavailable rolls back rule changes
no-op Save writes nothing
reload failure is distinguished from commit failure
PostgreSQL-only guard remains green
```

## Physical acceptance after fix

```text
1. Add one valid rule.
2. Save once.
3. Expected structured result:
   NewRows=1
   ValidatedRows=1
   OutboxRowsStaged=<proven expected count>
   Committed=True
   ReloadSucceeded=True
4. Close/reopen and verify the rule persists.
5. Save again without changes and confirm NO_CHANGES.
6. Verify expected outbox event safely.
7. Do not activate Turn Policy.
8. Do not test checkout/payment.
```

## Safety boundaries

- No Docker dependency.
- No automatic current-DB mutation.
- No SQL Server code/package/config reintroduction.
- No hidden policy creation/activation.
- No checkout/payment test.
- No OBM source commit/push.
- No secrets/private row values in GitHub artifacts.

## Reporting protocol

First response/private handoff must provide:

```text
PRICE_RULE_DIRECT_WINDOWS_DIAGNOSTIC_CAPTURE_READY
```

with the exact operator steps and local diagnostic artifact path.

After the operator returns the sanitized physical exception, continue the same task, fix the proven cause, and create/push only an ultra-minimal:

```text
report/report090.md
```

Valid final verdicts:

```text
OBM_POS_PRICE_RULE_DIRECT_WINDOWS_SAVE_READY_FOR_PHYSICAL_RETEST
BLOCKED_PRICE_RULE_MANDATORY_POLICY_SCOPE
BLOCKED_PRICE_RULE_SYNC_CONTRACT_MISSING
BLOCKED_PRICE_RULE_SAVE_ROOT_CAUSE_UNPROVEN
BLOCKED_PRICE_RULE_BUILD_OR_TEST
```
