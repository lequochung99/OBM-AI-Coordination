# Prompt 017 — Rethink Phase 2 as a simple data-driven template seed

## Operator direction

Do not continue the narrow prompt016 exercise of forcing exactly six parameter rows and exactly three printer rows.

Prompt016 is superseded and must not be executed.

The operator clarified that the old seed approach may be unnecessarily complex because it:

- splits seed work into many separate modules/methods;
- treats each table as a separate special-case session/setup workflow;
- hard-codes arbitrary row counts;
- makes the baseline difficult to reason about and maintain.

The new preferred direction is:

```text
Identify the minimal tables/rows required for a clean POS startup
-> determine table dependency/FK order
-> use one versioned canonical template dataset
-> transform identity-scoped columns to Phase 1 Tenant/POS identity
-> insert everything in one transaction
-> verify and write one completion marker
```

Do not assume that changing only `TenantGuid` is sufficient. Audit every selected column and classify whether it must be preserved, replaced, regenerated, cleared, or derived.

## Status

Read completely:

```text
report/report014.md
report/report015.md
prompt/prompt015.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
E:\Project2026\RecoveryReports\InstallationV0\Phase2SeedAuditV003\
```

Prompt015 stopped safely with:

```text
BLOCKED_PHASE2_V001_MANIFEST_KEYS_UNRESOLVED
```

That blocker should now be treated as evidence that the count-driven seed design is the wrong abstraction, not merely a missing list of six keys and three printer types.

## Task type

Prompt017 is a read-only architecture reset and manifest-design audit.

Do not:

- implement source;
- modify WPF/API code;
- create or alter a database;
- run migrations;
- seed any target;
- change Phase 1 artifacts;
- restart runtime;
- update WPF build label.

WPF label remains:

```text
prompt011
```

## Primary question

Design the simplest safe Phase 2 baseline mechanism that can replace the old fragmented seed modules.

Evaluate this preferred architecture:

```text
Versioned canonical template rows
+ generic identity/value transformer
+ explicit dependency order
+ one transaction executor
+ deterministic verification/marker
```

The design must remain auditable and safe, but avoid creating one custom seed method for every table unless a table genuinely requires custom logic.

## Evidence sources

Use, in priority order:

1. `Phase2SeedAuditV003` live schema evidence.
2. Current WPF startup/read paths.
3. Current EF entity mappings and PostgreSQL constraints.
4. Existing canonical installation/baseline documents.
5. Reference DB safe patterns from report014.
6. Legacy `SeedDbProvider` only as historical evidence.

The legacy seed code is not authoritative. Mark methods obsolete for Phase 2 when a new generic plan makes them unnecessary.

If detailed live reference-row inspection is necessary and the approved local pgpass path still exists, it may be used only with role `hung`, `default_transaction_read_only=on`, `BEGIN TRANSACTION READ ONLY`, and `ROLLBACK`. If it does not exist, do not block automatically; first determine whether V003 and source evidence are sufficient.

## Step 1 — Determine the real minimal table set

Do not start from arbitrary counts.

For every candidate table from report014, determine whether normal WPF startup or the immediate post-install UI actually requires it:

```text
TblTenant
TblPosLocal
TblSetupWeird
TblSetupServicesMethod
TblSetupLoginMethod
TblSetupPaymentMethod
TblEmployeePermission
TblParameterSetting
TblSetupPrinter
TblTurnSetting
TblSetting
TblLocalOutbox
Phase 2 marker
```

For each table report:

```text
startup/read caller paths
behavior when table is empty
hard startup requirement or optional later setup
safe default fallback, if any
whether user can configure later
whether the table must be in baseline v001
```

Do not seed a table merely because the reference DB contains rows.

Do not preserve prior expectations such as `6 parameters` or `3 printers` unless current runtime evidence proves those exact rows are required.

## Step 2 — Build the dependency order

Use live FK/constraint evidence and logical identity dependencies.

Produce:

```text
physical FK graph
logical dependency graph
final insert order
```

The operator expects the solution may be mostly:

```text
insert prerequisite identity rows first
then insert independent configuration/lookup rows
replace TenantGuid with Phase 1 TenantGuid
replace PosGuid where required
```

Confirm this with evidence.

If candidate tables have no physical FK constraints, state that clearly and use a simple explicit logical order rather than inventing complex dependency handling.

## Step 3 — Column transformation matrix

For every selected table and every relevant column, classify into exactly one rule:

```text
PRESERVE_TEMPLATE_VALUE
REPLACE_WITH_PHASE1_TENANT_GUID
REPLACE_WITH_PHASE1_POS_GUID
REPLACE_WITH_PHASE1_POS_NAME_OR_SLOT
DETERMINISTIC_NEW_PRIMARY_KEY
REMAP_FOREIGN_KEY_FROM_TEMPLATE_MAP
GENERATE_CURRENT_UTC_TIMESTAMP
SET_CANONICAL_DEFAULT
CLEAR_TO_NULL_OR_EMPTY
DEFER_MACHINE_BINDING
EXCLUDE_ROW
```

At minimum inspect:

- primary GUID/key;
- TenantGuid;
- PosGuid/POS identity fields;
- FK GUIDs;
- timestamps;
- enabled/active flags;
- machine/printer bindings;
- secret/private/gateway/terminal fields;
- audit/source-client fields;
- row-version/concurrency fields.

Important correction to the simple-copy idea:

```text
Do not copy old primary GUIDs blindly when they are tenant-scoped or could conflict.
Do not change only TenantGuid while leaving old PosGuid/FK/payload references.
```

Prefer deterministic GUID generation from:

```text
manifest version + target TenantGuid + target PosGuid when applicable + table + stable row key
```

This allows repeatable retries without preserving reference-database identities.

## Step 4 — Canonical template source

Recommend one versioned template format, for example:

```text
InstallationV0/Phase2/Templates/phase2-baseline-template-v001/
```

Evaluate whether the source should be:

- typed C# records;
- JSON manifest loaded into typed records;
- generated SQL values;
- or another simple versioned representation.

Preferred properties:

```text
human reviewable
machine readable
diffable
no secrets/no salon-private values
stable row keys
explicit table order
explicit transform rules
expected row counts derived from content, not hard-coded separately
```

The template should contain only canonical safe values. It must not contain raw exports from `enailsalon_phasee1_pos1_pg`.

## Step 5 — New generic seed engine design

Design a minimal new Phase 2 owner rather than calling the legacy fragmented seed path.

Suggested conceptual components:

```text
Phase2BaselineTemplateV001
Phase2TablePlan
Phase2RowTransform
Phase2IdentityMap
Phase2AtomicSeedExecutor
Phase2SeedVerifier
```

The executor should:

```text
revalidate Phase 1 identity
validate target DB eligibility/schema
load one template version
begin one Npgsql/EF transaction
acquire tenant/POS-scoped advisory lock
apply table plans in order
transform identity/key columns
insert rows
optionally write approved outbox rows
write completion marker last
read back invariants
commit
```

Avoid:

- dozens of `SeedXxxAsync` methods;
- table-level `AnyAsync()` shortcuts;
- hidden transactions;
- random GUIDs on every retry;
- session state distributed through unrelated tables;
- copying machine-specific values;
- reusing legacy demo orchestration.

Custom handlers are allowed only for proven exceptions, such as complex payloads or singleton conflict rules.

## Step 6 — Existing seed module disposition

Inventory the old seed modules only enough to classify each as:

```text
REUSE_LOW_LEVEL_HELPER
REPLACE_WITH_GENERIC_TEMPLATE_ENGINE
DEVELOPMENT_DEMO_ONLY
OBSOLETE_FOR_INSTALLATION_V0
UNKNOWN
```

Produce a short replacement map:

```text
old method/module
current responsibility
new owner
can be left in place but never called
must be isolated/retired
```

Do not delete old code in prompt017.

## Step 7 — Outbox reconsideration

The earlier plan assumed one `TblLocalOutbox` row per baseline row. Re-evaluate this instead of carrying it forward automatically.

For each selected baseline table determine:

```text
local-only configuration
must sync to API/other POS
API already owns the authoritative value
safe to emit outbox
must not emit outbox
```

Compare these two strategies:

### Strategy A — No baseline outbox

Use when baseline rows are local installation defaults and later sync/bootstrap will establish authoritative shared data.

### Strategy B — Selective deterministic outbox

Use only for rows that must be propagated from the newly installed POS and for which an established sync contract exists.

Do not recommend `one outbox per row` merely for symmetry. Outbox volume and payload rules must reflect actual synchronization ownership.

Prompt017 must recommend one strategy per table and identify any operator decision still required.

## Step 8 — Marker and idempotency

Keep the proposed dedicated marker concept unless evidence gives a simpler safe alternative:

```text
dbo.TblInstallationV0Phase2SeedVersion
phase2-baseline-seed-v001
```

Marker must be written last.

Idempotency should be based on:

```text
manifest version
manifest content hash
Phase 1 Tenant/POS identity
deterministic stable row keys
```

Do not require duplicate hard-coded count constants. Expected counts should be computed from the template itself.

State handling:

```text
empty eligible target -> apply template
same marker/hash/identity -> verify and return success
partial rows without marker -> block/recovery required
conflicting stable key -> block
newer marker -> block older installer
transaction failure -> rollback everything
```

## Step 9 — Proposed implementation scope

Produce a concrete implementation plan for the next prompt, including:

```text
new folders/classes
minimal existing files to modify
migration/marker addition
UI action/status changes
focused tests
physical disposable E2E plan
```

The next implementation prompt must not return to the old `exact six parameter rows / exact three printer rows` gate. Counts come from the approved template content.

## Mermaid diagrams

Include:

1. Current fragmented legacy seed flow.
2. Proposed simple template-transform transaction flow.
3. Table dependency/order diagram.

## Local artifact

Create a new versioned local-only design folder:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2SeedArchitectureV001\
```

Suggested artifacts:

```text
minimal-table-matrix.tsv
column-transform-matrix.tsv
dependency-order.tsv
legacy-replacement-map.tsv
outbox-ownership-matrix.tsv
template-v001-proposal.json
ARCHITECTURE.md
SHA256SUMS.txt
```

Do not overwrite V003 or any earlier artifact.
Do not commit local-only evidence to the public repository.

## Safety

Keep unchanged:

```text
Phase 1 ProductRoot
ApiAuthorized checkpoint
DPAPI-protected WpfJwt
LocalInstallationGuid
Tenant/POS/InstallationAttempt identity
```

No database/source/runtime mutation.
No secrets/private business rows in the public report.
No WPF label change.

## Report 017 — 100% detail

Create and push:

```text
report/report017.md
```

Required sections:

1. Verdict.
2. Why the count-driven/fragmented seed design was rejected or retained.
3. Minimal required table set with startup evidence.
4. Explicit deferred/excluded tables.
5. Physical FK and logical dependency order.
6. Per-table column transformation matrix.
7. TenantGuid/PosGuid/PK/FK replacement rules.
8. Canonical template format and folder proposal.
9. Proposed generic seed-engine components.
10. Legacy module replacement/disposition map.
11. Outbox ownership recommendation per table.
12. Marker/idempotency/version design.
13. One-transaction flow.
14. Expected counts derived from template proposal.
15. Remaining operator decisions.
16. Exact implementation scope for prompt018.
17. Mermaid diagrams.
18. No mutation/source/runtime/secret proof.
19. Local artifact paths and SHA-256.
20. Coordination commit SHA in final response.

## Valid verdicts

Preferred architecture is implementation-ready:

```text
PHASE2_SIMPLE_TEMPLATE_SEED_ARCHITECTURE_READY_FOR_IMPLEMENTATION
```

Operator decisions remain:

```text
PHASE2_SIMPLE_TEMPLATE_SEED_REQUIRES_OPERATOR_DECISIONS
```

Evidence insufficient:

```text
BLOCKED_PHASE2_SIMPLE_TEMPLATE_SEED_ARCHITECTURE
```
