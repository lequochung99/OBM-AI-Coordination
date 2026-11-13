# TASK-SETUP-COST-FEE-MERCHANT-SEED-V001

## Classification

- Type: `INVESTIGATE_THEN_IMPLEMENT`
- Risk: `L2_LOCAL_SEED_WITH_POSSIBLE_SYNC_CONTRACT`
- Implementation difficulty: `MEDIUM`
- Recommended executor now: `Cursor`
- DeepSeek: `NOT_NEEDED_BY_DEFAULT`
- Codex: `OPTIONAL_LATER_REVIEW_IF_SYNC_CONTRACT_OR_COMPLEXITY_EXPANDS`
- Manual UI owner: `OPERATOR`
- Expected UI work: `NONE`

## Goal

Resolve the real contract for the setup table the operator currently calls `TblSetupCostAndFeeMerchant`, determine whether the canonical code/schema name is instead `TblSetupCostAndFreeMerchant`, inspect the real non-secret configuration values from the legacy/source PostgreSQL database `enailsalon_phasee1_pos1_pg`, and add the approved values to the canonical initial WPF seed path so a fresh or missing-key installation receives the correct defaults.

The investigation must be persisted as reusable DB context so future Cursor/Codex tasks do not rediscover this table from scratch.

## Hard safety boundaries

1. `enailsalon_phasee1_pos1_pg` is READ-ONLY source evidence for this task. Do not update, migrate, truncate, drop, or otherwise mutate it.
2. Do not reset or recreate the active WPF development database.
3. Do not modify ApiServer, PlatformApp, BookingConsole, payment runtime, or unrelated WPF modules unless a concrete compile-time contract proves a minimal change is required. If that occurs, report the dependency before expanding scope.
4. Do not copy secrets or merchant credentials into source control. Exclude passwords, tokens, private keys, processor secrets, account secrets, connection strings, device secrets, or any sensitive merchant identifiers not required as safe default configuration.
5. Do not invent a new outbox/sync path. Reuse only the existing canonical initial-seed/outbox mechanism if the table is already a syncable contract.
6. Preserve operator-modified existing rows. Seed by stable key/missing-row semantics; do not blindly overwrite an existing configured row.
7. No UI investigation or visual verification is required for this task.

## Required read order

Before touching product source:

1. `AGENTS.md`
2. `CURRENT.md`
3. `constitution/CURRENT.md`
4. `state/CURRENT_TASK.json`
5. this task file
6. `context/database/TblSetupCostAndFeeMerchant/CURRENT.md`
7. only then query Graphify and inspect targeted product source

## Phase A — Graphify-first targeted investigation

Product root:

`E:\Project2026\4POS\NailSalonNet8`

Do not load the full `graphify-out/graph.json` into AI context.

First check whether the graph is current enough for the source commit. If source has materially changed since graph generation, run the normal incremental Graphify update; otherwise reuse the existing graph.

Run exact-symbol queries first:

```powershell
graphify explain "TblSetupCostAndFeeMerchant"
graphify explain "TblSetupCostAndFreeMerchant"
```

If one spelling has no node, record that fact. If both exist, determine which is entity/table/runtime contract and whether the other is historical or merely a naming mismatch.

Then query only the exact discovered symbols/methods needed to locate:

- EF entity and `DbSet`;
- mapper/DTO if present;
- initial seed owner;
- seed manifest/data source;
- stable key / primary key behavior;
- any existing `TblLocalOutbox` staging path;
- any API-sync contract references;
- tests that already cover the table or adjacent setup tables.

Use `graphify explain`, exact `graphify query`, and `graphify path` as navigation aids. If Graphify reports `ambiguous`, resolve by exact file/symbol in source rather than guessing.

### Persist selected Graphify findings

Save only sanitized, useful query results—not the 23+ MB raw graph—to the coordination repo under:

```text
graphify/wpf/current/queries/SetupCostFeeMerchant/
```

At minimum create:

```text
SYMBOL_SUMMARY.md
SEED_PATH.md
SYNC_PATH.md
```

These files should contain concise Graphify-derived navigation facts, source paths/symbol names, source commit SHA, and any ambiguity/conflict. Do not paste source code bodies or secrets.

## Phase B — Physical database investigation

Use PostgreSQL metadata and read-only SQL against `enailsalon_phasee1_pos1_pg` to establish FACTS:

1. exact physical table name;
2. row count;
3. complete column names and PostgreSQL types;
4. primary key / unique key / foreign keys;
5. which columns are business defaults versus identity/audit/secret fields;
6. the real values required for canonical initial seed;
7. whether TenantGuid/PosGuid or other installation-specific identity must be substituted at runtime rather than copied literally.

Also inspect the active WPF target database read-only before implementation to establish whether the table is empty, missing keys, or contains existing operator data. Resolve the active DB from current runtime/configuration; do not hardcode an old lane name.

### Sensitive-value gate

Before committing any value obtained from the source DB, classify every nontrivial column:

- `SAFE_SEED_CONFIGURATION`
- `RUNTIME_IDENTITY_REPLACE`
- `AUDIT_GENERATED`
- `SECRET_DO_NOT_COPY`
- `UNKNOWN_BLOCK`

If any required seed value is `UNKNOWN_BLOCK` or appears to be a secret/credential, stop and report the exact column instead of committing it.

## Phase C — Implement canonical initial seed

Locate and reuse the existing WPF initial seed architecture. Do not create a second seeder.

Required behavior:

```text
missing stable key -> insert approved canonical default
existing stable key -> preserve existing row
partial set -> insert only missing approved keys
complete set -> no-op
```

Seed values must come from the verified safe configuration in `enailsalon_phasee1_pos1_pg`, transformed only where the current installation lane requires runtime Tenant/POS identity or generated audit values.

### Sync/outbox rule

Determine from Graphify + source whether this table is part of the existing setup-data synchronization contract.

- If YES: stage it using the existing canonical initial-seed -> `TblLocalOutbox` path and existing entity/table naming, expected event count, transaction grouping, and mapper/DTO contract. Do not create a new uploader.
- If NO: seed local-only and explicitly record `NO_EXISTING_SYNC_CONTRACT` in the report/context.
- If evidence conflicts: do not guess; mark `BLOCKED_SYNC_CONTRACT_CONFLICT` and stop before inventing behavior.

Initial seed and outbox changes, if applicable, must follow the existing transaction/idempotency pattern used by adjacent approved setup tables.

## Phase D — Focused validation

At minimum prove:

1. main WPF project builds with zero new errors;
2. focused seed tests pass;
3. empty/missing stable key receives the canonical row(s);
4. rerun is idempotent and adds no duplicates;
5. existing configured row is preserved;
6. source DB `enailsalon_phasee1_pos1_pg` has zero mutations from this task;
7. no secret value was committed;
8. if syncable, exact expected `TblLocalOutbox` row/group is produced through the existing path and no unrelated outbox rows are added;
9. if local-only, zero outbox rows are created for this table;
10. no unrelated database reset/drop/recreate occurred.

Do not spend quota on UI verification.

## Phase E — Mandatory reusable DB knowledge artifact

Before closing the task, create:

```text
context/database/TblSetupCostAndFeeMerchant/V001/DB_CONTEXT.md
```

and update:

```text
context/database/TblSetupCostAndFeeMerchant/CURRENT.md
```

to point to V001.

`DB_CONTEXT.md` must include:

- canonical entity name;
- canonical physical table name;
- historical/alternate spelling and resolution;
- source commit SHA used for code investigation;
- Graphify snapshot/source commit used;
- source DB name (`enailsalon_phasee1_pos1_pg`) and read-only evidence timestamp;
- safe row count and stable keys;
- column/type summary;
- PK/unique/FK summary;
- column classification (`SAFE_SEED_CONFIGURATION`, `RUNTIME_IDENTITY_REPLACE`, etc.);
- sanitized canonical seed values only;
- canonical seed owner/file/method;
- idempotency behavior;
- mapper/DTO ownership if any;
- outbox/sync behavior and exact existing owner if any;
- tests proving behavior;
- known risks/unknowns;
- exact invalidation conditions that would require reinvestigation.

Also update `context/investigations/INDEX.md` with a one-line reference if a new reusable investigation artifact is created.

Future agents must be able to answer "how is this table seeded and synchronized?" by reading this DB context plus targeted source verification, without scanning the whole WPF repository.

## Required task report

Create a concise implementation report in the normal report area available to the current workflow. If the local numbered prompt/report sequence is ahead of the remote coordination branch, do not guess or overwrite a number; use the current local sequence determined by the operator/repository.

The report must include:

- verdict;
- exact canonical table/entity spelling;
- Graphify queries reused and files inspected;
- source DB row count and sanitized values used;
- target pre-state and post-state;
- seed code files changed;
- idempotency proof;
- outbox/sync verdict;
- build/test result;
- source DB mutation count = 0;
- secret-copy audit result;
- reusable context paths written;
- unresolved blockers, if any.

## PASS criteria

PASS only when the exact table contract is resolved, approved non-secret legacy values are represented in the canonical initial seed, idempotency is proven, sync behavior follows the existing architecture (or is proven local-only), build/tests pass, the source DB remains untouched, and reusable DB context has been committed.

Suggested verdict:

`WPF_SETUP_COST_FEE_MERCHANT_CANONICAL_INITIAL_SEED_PASS`

If blocked, use an exact `BLOCKED_*` verdict describing the unresolved contract or sensitive-data issue.
