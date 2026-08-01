# Prompt 062 — Resolve the unrelated Payroll migration artifact blocker and certify Phase 2 V003 PIN seed tests

## Operator decision

Prompt061 implemented the Phase 2 V003 usable employee operational PIN seed and created canonical V002, but the required broad test filter ended with one unrelated failure:

```text
E:\Project2026\1ApiServer\ApiServer01\SpacePos.Provisioning.Schema\Migrations\011_payroll_three_table_data_foundation.sql
```

The two required builds passed. The V003 source work is present. This prompt must close the test/repository artifact gap without hiding it and without changing the approved PIN policy.

Do not merely narrow the test filter and declare success. First prove why the Payroll migration artifact is missing or why the test path is obsolete.

## Mandatory documentation gate

Before editing source, tests, project files, migration artifacts, or current documentation, read completely:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report060.md
report/report061.md
```

Record before the first edit:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V002
CanonicalDocSha256=916AE7CE71ADA14D768DE1D11E7E5F3604416E8176AA3C59479353481AA2DFB2
```

If the current files differ, read the changed contents and verify this bounded task remains authorized. Otherwise stop before edits with:

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```

## Canonical PIN policy must remain unchanged

```text
Staff real/login-capable employee
-> exactly 4 numeric digits

Non-Staff real/login-capable employee
-> exactly 5 numeric digits
-> leading zeros allowed

Uniqueness
-> TenantGuid scope
-> Staff and Non-Staff share the same uniqueness set
```

Do not revert to 6-digit Non-Staff PINs.

Do not create an employee password framework.

## Scope boundary

This prompt is primarily a repository/test-integrity closure.

Allowed:

```text
- inspect the failing test and its call stack;
- inspect source/project/git history for the missing migration artifact;
- restore an exact proven artifact from canonical source history when available;
- correct an obsolete path/reference when source proves the migration was renamed/moved;
- update the unrelated test to the current canonical migration inventory;
- add precise V003 PIN tests when current coverage is incomplete;
- update CURRENT_TASK/CURRENT_RESULT and versioned history;
- build and test.
```

Not allowed:

```text
- invent SQL migration contents;
- create a dummy/empty file to satisfy the test;
- silently skip or ignore the failing test;
- weaken assertions unrelated to an established repository change;
- mutate PostgreSQL;
- run V003 against obm_pos_dev_v0_pg;
- change employee PINs in the current operator DB;
- launch WPF;
- commit/push OBM source.
```

## Phase 1 — identify the exact failing test

Rerun the single failure with detailed output and record safely:

```text
FullyQualifiedName
source file/method/line
assertion
call stack
why the prompt061 filter includes the test
whether the test belongs to Payroll migration integrity, general schema integrity, employee/payroll, or Phase 2 seed
```

Do not infer from the filename alone.

Find every active source/test/project reference to:

```text
011_payroll_three_table_data_foundation.sql
SpacePos.Provisioning.Schema\Migrations
payroll_three_table_data_foundation
011_payroll
```

Record:

```text
reference file/line
expected path construction
whether the file is Content/None/EmbeddedResource
CopyToOutput behavior
relative versus absolute path assumptions
```

## Phase 2 — inspect canonical migration provenance

Inspect the current local Git worktrees and history without modifying them first:

```text
E:\Project2026\1ApiServer\ApiServer01
E:\Project2026\4POS\NailSalonNet8
```

Use safe provenance checks:

```text
git status --short
git ls-files
git log --all -- <path>
git show <proven-commit>:<path>
git grep
filesystem search for exact/nearby migration names
project file inspection
migration manifest/index inspection
backup/evidence folder search by exact filename and checksum
```

Do not copy from an arbitrary stale backup.

The report must classify the root cause as exactly one of:

```text
MIGRATION_RENAMED_OR_MOVED_TEST_PATH_OBSOLETE
TRACKED_MIGRATION_MISSING_FROM_WORKTREE
GENERATED_ARTIFACT_NOT_MATERIALIZED
MIGRATION_SUPERSEDED_TEST_EXPECTATION_OBSOLETE
REQUIRED_MIGRATION_SOURCE_ARTIFACT_GENUINELY_MISSING
OTHER_PROVEN_CAUSE
```

## Phase 3 — correction rules by classification

### A. Migration renamed or moved

If source/git history proves the canonical migration now exists under a different path/name:

```text
- update the test or migration inventory to the canonical current path;
- preserve semantic assertions;
- do not duplicate the migration under the old name;
- document old -> new path and provenance.
```

### B. Tracked file missing from worktree

If Git proves the exact file is tracked in a reachable canonical commit and its absence is accidental:

```text
- restore the exact bytes from the proven commit/path;
- record commit SHA and file SHA-256;
- do not edit the SQL contents;
- do not apply the migration physically.
```

### C. Generated artifact not materialized

If an approved generator owns the file:

```text
- run only the existing documented deterministic generator;
- record generator command/version/input/output hash;
- do not author the SQL manually;
- ensure normal checkout/build/test materializes or no longer incorrectly expects generated source.
```

### D. Migration superseded / test expectation obsolete

If schema history proves migration 011 is no longer a current required artifact:

```text
- update the test to validate the current canonical migration manifest/history;
- retain an audit reference to the superseded migration when needed;
- do not weaken the expected payroll schema contract.
```

### E. Genuinely missing required artifact

If no trustworthy source/provenance can reconstruct the required SQL:

```text
- do not invent or approximate it;
- stop with BLOCKED_REQUIRED_PAYROLL_MIGRATION_ARTIFACT_MISSING;
- report the exact expected contract and provenance searches attempted.
```

## Phase 4 — independently certify V003 PIN logic

Regardless of the Payroll classification, run a precise V003 test subset that names the actual V003 PIN policy/seed test classes or methods. Do not rely only on a broad substring filter.

Required proof areas:

```text
- Staff exact 4-digit generation/preservation;
- Non-Staff exact 5-digit generation/preservation;
- leading-zero formatting;
- placeholder rejection;
- invalid/wrong-length source rejection;
- duplicate source handling;
- TenantGuid-scoped uniqueness;
- same value allowed across different tenants when policy permits;
- cryptographic RNG code path;
- no raw PIN logging;
- V003 marker version;
- idempotent rerun/no rotation;
- no duplicate marker/outbox behavior;
- current operator DB not referenced.
```

Report the exact test names and counts without raw PIN values.

If these V003 tests fail for a real implementation defect, fix only the proven defect and report it. Do not mix unrelated redesign into this prompt.

## Phase 5 — rerun the original acceptance commands

Run sequentially:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Phase2|FullyQualifiedName~Seed|FullyQualifiedName~Employee|FullyQualifiedName~LoginNumber|FullyQualifiedName~Pin|FullyQualifiedName~Documentation|FullyQualifiedName~Naming" -v minimal
```

The final accepted result requires:

```text
0 failed
```

Do not declare final PASS solely from the narrower V003 subset.

## Phase 6 — source and migration integrity checks

After correction, verify:

```text
- the migration artifact/reference is reproducible from a clean checkout or canonical generator;
- no absolute developer-specific path is required by the test;
- no duplicate conflicting migration file was introduced;
- no SQL was applied to any DB;
- Phase 2 V003 code still does not reference the Payroll migration artifact;
- the Payroll test remains logically independent from V003 seed behavior.
```

Add or update a focused test only when needed to prevent the exact path/provenance regression.

## Documentation and versioning

Canonical architecture and PIN policy remain V002.

Before updating current docs, preserve the current `CURRENT_TASK.md` and `CURRENT_RESULT.md` under the next available versioned history folder.

After full build/test PASS:

```text
CURRENT_RESULT.md
-> records Payroll artifact/test blocker closure and V003 full test PASS

CURRENT_TASK.md
-> authorizes only a clean disposable Phase 2 V003 seed retest
-> does not authorize current DB repair
-> does not authorize ASP.NET Identity deletion
```

Do not create canonical V003 unless the PIN policy or architecture changes; this task should not change either.

## Evidence folder

Create the next available folder:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2V003TestClosureV001
```

Never overwrite an existing folder.

Expected safe artifacts:

```text
README.md
SHA256SUMS.txt
failing-test-before.txt
migration-provenance.md
migration-reference-inventory.csv
v003-focused-test-results.txt
original-filter-test-results.txt
build-results.txt
```

Do not include SQL contents, raw PINs, credentials, private GUIDs, or business data.

## Build label

Keep `prompt061` if no InstallationV0 UI/build-label source changes are needed.

If prompt062 changes InstallationV0 source or UI, set:

```text
Build label: prompt062
Window title: OBM InstallationV0 Phase 1/2 - prompt062
```

Report the decision.

## Prohibited actions

Do not:

```text
mutate obm_pos_dev_v0_pg
mutate any reference/production-like DB
apply Payroll migrations
run V003 seed physically
change current employee LoginNumber values
print raw PINs
change API tokens/contracts
change DB roles/passwords/GRANTs
launch WPF
commit/push OBM source
create dummy migration SQL
hide the failing test by exclusion
```

## Report 062

Create and push:

```text
report/report062.md
```

Required sections:

1. Verdict.
2. DOCS_READ_BEFORE_CODE_GATE proof.
3. Exact failing test and why the broad filter selected it.
4. Migration reference/path inventory.
5. Git/provenance search evidence.
6. Root-cause classification.
7. Exact correction or blocker.
8. Proof no SQL content was invented/applied.
9. Exact V003 focused tests and counts.
10. Original prompt061 filter rerun and counts.
11. Build results.
12. Phase 2 V003 implementation unchanged or exact bounded fix.
13. Current physical DB non-mutation proof.
14. Evidence folder and hashes.
15. Canonical V002/current docs/history hashes.
16. Build-label decision.
17. Exact next clean disposable V003 retest steps.
18. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_PHASE2_V003_PIN_SEED_BUILD_TEST_PASS_READY_FOR_DISPOSABLE_RETEST
```

```text
BLOCKED_REQUIRED_PAYROLL_MIGRATION_ARTIFACT_MISSING
```

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```

```text
BLOCKED_PHASE2_V003_PIN_SEED_REAL_TEST_FAILURE
```

```text
BLOCKED_PHASE2_V003_TEST_CLOSURE_BUILD_OR_TEST
```
