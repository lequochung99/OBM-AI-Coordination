# Prompt 061 — Canonical V002 employee operational PIN policy and Phase 2 V003 usable seed

## Operator decision

The physical OBM-POS runtime now opens successfully and the startup/station modal defects are closed through prompt060.

The operator now authorizes the next bounded task:

```text
Make future Phase 2 employee seeds immediately usable by assigning valid numeric
operational PINs instead of UNCONFIGURE placeholders.
```

This is a seed-policy task. It is not a new authentication system.

## Exact approved PIN policy

```text
EmployeeType = Staff
-> LoginNumber/PIN is exactly 4 numeric digits
-> example: 1111

Every other real/login-capable EmployeeType
-> LoginNumber/PIN is exactly 5 numeric digits
-> leading zeros are allowed
-> examples: 00001, 00010, 00011
```

The operator's example `000010` has 6 digits and is treated as a typo for `00010` because the explicit policy is 5 digits.

`TblEmployee.LoginNumber` is an employee operational PIN only. It is not:

```text
an application password
PostgreSQL authentication
API authentication
device identity
installation proof
startup credential
ASP.NET Identity
```

## Current physical evidence

The current physical seed shows a few usable values such as:

```text
Staff example: 1111
Non-Staff example: 00001
```

Most other employees currently show values shaped like:

```text
UNCONFIGURE~<suffix>
```

The purpose of this task is to prevent those placeholders in future clean Phase 2 V003 seeds.

Do not print any real/generated PIN values in the public coordination report.

## Mandatory documentation gate

Before editing source, tests, project files, seed manifests, or current documentation, read completely:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
report/report055.md
report/report056.md
report/report057.md
report/report059.md
report/report060.md
```

Before the first source edit, record:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=<current version>
CanonicalDocSha256=<actual current hash>
```

The current canonical V001 documented Non-Staff as 6 digits. The operator has now explicitly changed the policy to 5 digits. Therefore this task must update documentation first and create canonical V002 before implementation edits.

If documentation cannot be safely versioned, stop before source edits with:

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```

## Phase A — create canonical V002 before code changes

Preserve the current canonical V001 exactly under the next valid history location, for example:

```text
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V001\INSTALLATION_RUNTIME_CANONICAL_V001.md
```

Do not overwrite any existing history file. If that exact path already exists, verify its hash and use the next safe versioned path.

Then update the current canonical authority:

```text
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
```

Required header/state:

```text
Version: V002
Status: Current Canonical Authority
Supersedes: V001
```

V002 must state:

```text
Staff operational PIN = exactly 4 numeric digits
Non-Staff real/login-capable operational PIN = exactly 5 numeric digits
Leading zeros are allowed for 5-digit PINs
PIN uniqueness scope = TenantGuid
PINs are operational UI/audit identifiers, not application passwords
```

Update `AGENTS.md` so future employee PIN/seed tasks must follow V002 and must not reintroduce 6-digit Non-Staff PIN language.

Preserve the current `CURRENT_TASK.md` and `CURRENT_RESULT.md` under the next versioned history folder before replacing them.

Only after canonical V002 and the read gate are complete may implementation edits begin.

Required implementation-phase gate evidence:

```text
DOCS_UPDATED_BEFORE_CODE=PASS
CanonicalDocVersion=V002
CanonicalDocSha256=<V002 hash>
```

## Scope of implementation

Audit the exact Phase 2 seed implementation and the exact current EmployeeType enum/model before modifying behavior.

Inspect at least:

```text
Phase2ReferenceDrivenTrialService or current equivalent
employee seed manifest/builder
reference employee projection/mapping
TblEmployee model and schema
EmployeeType enum/constants
Phase2 trial marker/version handling
existing UNCONFIGURE placeholder generator
existing duplicate/LoginNumber validation
current Phase 2 v001/v002 tests
```

Do not guess EmployeeType numeric values. Report the exact Staff value and all current seeded non-Staff types.

## Phase 2 V003 seed versioning

Create a new explicit seed/trial version. Preserve v001 and v002 evidence and behavior for audit/rollback.

Use an exact version name derived from the existing naming convention, such as:

```text
phase2-reference-driven-trial-v003-usable-employee-pins
```

Audit and use the actual project convention rather than silently inventing a parallel marker system.

Requirements:

```text
- v001/v002 files, markers, manifests, reports, and history are preserved;
- V003 is clearly identifiable as the current future clean-install seed;
- seed rerun is idempotent;
- an already completed v001/v002 physical DB is not automatically rewritten;
- no startup-time PIN rotation;
- no hidden automatic upgrade of the current obm_pos_dev_v0_pg database.
```

The current physical database is read-only for this prompt.

## PIN assignment algorithm

### 1. Eligibility

Apply the policy to every real/login-capable employee included in the Phase 2 V003 seed.

```text
EmployeeType == Staff
-> 4 digits

EmployeeType != Staff
-> 5 digits
```

If the seed contains virtual/non-login/system employee rows, do not invent behavior. Audit their exact model and classify them explicitly. Generate no operational PIN for a row that source policy proves cannot log in. Report the exception without private data.

### 2. Preserve valid source values

A source/reference LoginNumber may be preserved only when all are true:

```text
- numeric only;
- exact required length for the employee type;
- appears exactly once among eligible employees for the same tenant;
- does not collide with a valid existing target LoginNumber;
- is not an UNCONFIGURE/placeholder/sentinel value.
```

This allows examples such as a unique Staff `1111` or unique Non-Staff `00001` to remain unchanged.

Do not copy duplicate source PINs.

### 3. Generate missing/invalid values once

For every eligible employee whose source PIN is missing, placeholder, wrong length, nonnumeric, or duplicated:

```text
Staff:
- generate a numeric value in 1000..9999
- format exactly D4

Non-Staff:
- generate a numeric value in 0..99999
- format exactly D5
- leading zeros are required when needed
```

Use `System.Security.Cryptography.RandomNumberGenerator` or an equivalent framework cryptographic RNG. Do not use time-seeded `Random`.

Generated values must be persisted once during the V003 seed transaction.

Do not regenerate or rotate them on seed rerun, app restart, API reconnect, or installation resume.

### 4. Tenant-scoped uniqueness

All non-empty operational PINs must be unique within the same `TenantGuid`, across Staff and Non-Staff together.

Required rules:

```text
- uniqueness is per TenantGuid, not global;
- collision retries are bounded;
- fail the seed transaction if uniqueness cannot be established;
- do not partially insert employees;
- do not silently rewrite valid existing target PINs;
- if an existing target database already contains duplicate valid PINs, return an explicit review/error result rather than randomly rotating them.
```

Do not add a new table.

Audit whether a DB unique index/constraint already exists. Do not add a migration in this task unless the existing canonical schema already requires one and source evidence proves it is missing. Prefer seed validation for this bounded task and report the schema finding.

### 5. Placeholder elimination

Future V003 clean seed must produce zero eligible employee values matching any placeholder pattern such as:

```text
UNCONFIGURE
UNCONFIGURED
UNCONFIGURE~*
UNCONFIGURED~*
```

Do not rename the placeholder and keep using it.

Non-login rows, if any, must use the model's explicit non-login representation, not a fake PIN-shaped string.

## Transaction and idempotency

The V003 employee seed must run transactionally.

Expected behavior:

```text
first V003 execution
-> preserve valid unique source values
-> generate missing valid unique values
-> insert/update intended seed rows
-> write V003 completion marker
-> commit

V003 rerun
-> detect completed/current rows
-> preserve all assigned PINs
-> no rotation
-> no duplicate outbox/event rows
-> no marker duplication
```

If execution stops before commit, retry must not create a new set of persisted values beside partial prior rows.

## Current physical DB policy

Do not mutate:

```text
obm_pos_dev_v0_pg
```

Do not replace the currently visible placeholders in this prompt.

This task prepares correct behavior for the next clean seed.

Any optional one-time repair/import for an existing V002 database must be a separate operator-authorized prompt after V003 source/build/test proof.

## Security and reporting

Raw operational PIN values are local salon data. They must not be printed in:

```text
report/report061.md
logs
public coordination artifacts
test output
Graphify/evidence files
exception messages
```

Safe report evidence includes only:

```text
employee counts
Staff/non-Staff counts
preserved/generated counts
length-validation counts
duplicate count = 0
placeholder count = 0
idempotency booleans
```

The actual application UI may display the values according to existing employee-management behavior; do not change that UI in this task.

## Tests

Add focused unit/integration tests for at least:

```text
Staff valid unique source 1111 -> preserved
Non-Staff valid unique source 00001 -> preserved
Staff placeholder -> generated exact 4 numeric digits
Non-Staff placeholder -> generated exact 5 numeric digits
Non-Staff generated value preserves leading-zero formatting through D5
wrong-length source value -> not preserved
duplicate source PIN -> not copied as duplicate
all final eligible PINs unique per TenantGuid
same PIN may exist in a different TenantGuid when policy permits
no final eligible PIN matches UNCONFIGURE placeholder patterns
V003 rerun preserves exact assigned PINs
V003 rerun does not duplicate markers/outbox/events
existing target duplicate valid PINs -> explicit failure/review, no rotation
transaction failure -> no partial employee/PIN commit
Staff and Non-Staff rules use audited EmployeeType values
current physical database is not accessed/mutated
canonical V002 terminology guard
```

Use a disposable test database only when an existing safe harness is already available. Never point tests at `obm_pos_dev_v0_pg` or the reference production-like database.

## Seed result evidence

Create the next available versioned evidence folder:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2UsablePinSeedV003\
```

If it exists, use the next versioned folder; never overwrite.

Expected artifacts:

```text
README.md
SHA256SUMS.txt
employee-type-policy.md
seed-version-inventory.csv
pin-assignment-algorithm.md
safe-test-counts.json
placeholder-scan.txt
idempotency-proof.md
```

Do not include raw PIN values, employee names, private GUIDs, credentials, or business rows.

## Documentation completion

Canonical remains V002 after implementation.

Update current docs to record:

```text
- Phase 2 V003 usable PIN seed implemented;
- Staff exact 4 digits;
- Non-Staff exact 5 digits;
- tenant-scoped uniqueness;
- valid unique source values preserved;
- missing/invalid values generated once;
- current V002 physical DB was not rewritten;
- next approved task is a clean disposable V003 seed retest, not production/current DB repair.
```

Update the naming/terminology guard so active canonical/current docs fail on:

```text
Non-Staff PIN = 6 digits
UNCONFIGURE placeholders as approved seed output
employee password
manager password
```

Do not erase historical evidence under versioned history/report folders.

## Build label

If InstallationV0 build-label source changes, set:

```text
Build label: prompt061
Window title: OBM InstallationV0 Phase 1/2 - prompt061
```

A build-label change is optional if no InstallationV0 UI/source file needs modification. Report the decision.

## Required builds/tests

Run sequentially:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Phase2|FullyQualifiedName~Seed|FullyQualifiedName~Employee|FullyQualifiedName~LoginNumber|FullyQualifiedName~Pin|FullyQualifiedName~Documentation|FullyQualifiedName~Naming" -v minimal
```

## Prohibited actions

Do not:

```text
mutate obm_pos_dev_v0_pg
mutate the reference database
run the V003 seed physically against the current operator DB
change current employee LoginNumber values
create employee passwords
change API contracts/tokens
change PostgreSQL roles/passwords/GRANTs
redeem Pairing Code
launch WPF automatically
drop ASP.NET Identity tables
commit/push OBM source
print raw PIN values or employee private data
```

## Report 061

Create and push:

```text
report/report061.md
```

Required sections:

1. Verdict.
2. Original V001 gate evidence.
3. Canonical V001 preservation path/hash.
4. Canonical V002 created-before-code proof and final hash.
5. Exact EmployeeType audit and Staff classification.
6. Current seed implementation/placeholder root cause.
7. V003 seed version and marker convention.
8. PIN preservation and generation algorithm.
9. Tenant-scoped uniqueness behavior.
10. Placeholder elimination proof.
11. Transaction/idempotency proof.
12. Existing-target duplicate behavior.
13. Current physical DB non-mutation proof.
14. Raw PIN secrecy/reporting proof.
15. Exact source/docs/tests files changed.
16. Build/test commands and counts.
17. Evidence folder and hashes.
18. Current canonical/current-task/current-result hashes.
19. Build-label decision.
20. Exact next clean disposable V003 seed retest steps.
21. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_PHASE2_V003_USABLE_EMPLOYEE_PIN_SEED_READY_FOR_CLEAN_RETEST
```

```text
BLOCKED_CANONICAL_DOCUMENTATION_GATE
```

```text
BLOCKED_EMPLOYEE_TYPE_POLICY_AMBIGUOUS
```

```text
BLOCKED_PHASE2_V003_PIN_SEED_BUILD_OR_TEST
```
