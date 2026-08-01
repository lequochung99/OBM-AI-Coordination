# Prompt 016 — Resolve exact Phase 2 v001 parameter keys and printer types

## Context

Read completely:

```text
prompt/prompt014.md
report/report014.md
prompt/prompt015.md
report/report015.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Prompt015 correctly stopped before source implementation with:

```text
BLOCKED_PHASE2_V001_MANIFEST_KEYS_UNRESOLVED
```

The only unresolved implementation gate is:

```text
exact 6 safe mandatory TblParameterSetting keys
exact 3 placeholder-safe TblSetupPrinter.PrinterType rows
```

Prompt016 is a narrow read-only manifest-resolution audit. It must not implement Phase 2, modify WPF, change schema, seed a database, or restart runtime.

## Authoritative evidence priority

Resolve the keys/types from these sources in order:

1. Existing canonical baseline/provisioning artifacts that previously produced the accepted development baseline.
2. Live read-only state of the canonical development database:

```text
obm_pos_dev_v0_pg
```

3. Current WPF startup/read paths and enums.
4. Prompt014 V003 reference-DB evidence.
5. Legacy seed code only as supporting evidence, never as sole authority.

The canonical development database is a secondary read-only baseline reference, not a target for mutation.

## Known canonical development baseline expectation

Prior accepted project state records:

```text
TblParameterSetting rows = 6
TblSetupPrinter rows = 3
```

Prompt016 must identify the exact safe stable keys/types behind those accepted counts and determine whether they remain valid for `phase2-baseline-seed-v001`.

Do not assume the row identities. Prove them.

## Source/document search scope

Read-only search under:

```text
E:\Project2026\RecoveryReports
E:\Project2026\CanonicalInstallationDocs
E:\Project2026\4POS\NailSalonNet8\SeedDb
E:\Project2026\4POS\NailSalonNet8\Services
E:\Project2026\4POS\NailSalonNet8\Enums
E:\Project2026\4POS\NailSalonNet8.Tests
```

Search for artifacts and code related to:

```text
obm_pos_dev_v0_pg
System Baseline
ParameterSetting
PrinterDefaults
TblSetupPrinter
Date Hien Tai
FeatureGateNames
PrinterType
phase2-baseline-seed
seed manifest
baseline manifest
```

For every decisive artifact, report:

```text
path
version/date if present
role in prior provisioning
whether it is canonical, historical, or ambiguous
```

## Read-only development DB audit

Database:

```text
obm_pos_dev_v0_pg
```

Use only approved local non-secret credential handling already available on the machine. Do not ask for or print a password, connection string, pgpass content, or secret.

Preferred role:

```text
hung
```

If a fixed local pgpass file exists, Codex may set process-local `PGPASSFILE` without reading the file contents. If no approved credential path exists, continue the source/artifact audit and report the DB portion as blocked; do not fallback to `postgres` or modify privileges.

Every DB batch must use:

```text
PGOPTIONS=-c default_transaction_read_only=on
psql -X -v ON_ERROR_STOP=1
BEGIN TRANSACTION READ ONLY
ROLLBACK
```

Required proof if DB access succeeds:

```text
transaction_read_only = on
current_database = obm_pos_dev_v0_pg
current_user = hung
```

Cấm mọi mutation, including sequence calls with side effects.

## Exact parameter-key resolution

For the six existing `TblParameterSetting` rows in `obm_pos_dev_v0_pg`, collect only safe fields needed to identify the manifest:

```text
PropertyName or canonical stable key
scope discriminator if present
safe boolean/numeric/string default category
row count per stable key
whether current WPF reads the key
whether startup requires it
whether it is global/tenant/POS/machine scoped
whether it is secret/private/environment-specific
```

Do not report private URLs, gateway values, credentials, tokens, terminal identifiers, salon-specific values, or sensitive configuration.

For each proposed key, require all:

- non-secret;
- stable across new tenants;
- startup/safe-baseline relevance proven by code or canonical prior manifest;
- no ambiguous duplicate without an explicit scope key;
- value can be generated canonically rather than copied from salon history.

Produce final table:

```text
Ordinal
Exact PropertyName
Scope key
Canonical default
Evidence paths/read methods
Mandatory reason
Stable-key/idempotency rule
```

If the canonical development DB contains rows that are obsolete, accidental, or not code-backed, do not preserve the count of six merely for compatibility. Report the justified count and require operator approval.

## Exact printer-type resolution

For the three existing `TblSetupPrinter` rows in `obm_pos_dev_v0_pg`, collect only safe identifying fields:

```text
PrinterType
logical purpose
required columns
safe placeholder/default behavior
current WPF lookup/read paths
```

Never report/copy:

```text
printer name
IP
Windows path
share name
driver
machine name
physical binding value
```

Compare against the five source enum values:

```text
CUSTOMER_TERMINAL
MERCHANT_TERMINAL
POS_CUSTOMER
POS_MERCHANT
POS_EMPLOYEE
```

Produce final table:

```text
Ordinal
Exact PrinterType
Logical purpose
Why included/excluded
Placeholder-safe fields
Bind-later fields
Evidence path/read method
```

Determine whether the canonical three are:

- genuinely required baseline types;
- merely historical development rows;
- or incomplete relative to current WPF behavior.

Do not select three arbitrarily.

## Cross-check against current runtime paths

For each selected parameter key and printer type, identify all current read/caller paths:

```text
full source path
class/type
method/property
startup or feature behavior
behavior when row is absent
```

Classify absence behavior:

```text
hard startup failure
safe default fallback
feature disabled
screen-specific later setup
unknown
```

Only hard-startup or approved safe-baseline keys/types should be mandatory in v001.

## Final decision rules

Prompt016 must end with one of:

### Exact six and exact three proven

```text
PHASE2_V001_MANIFEST_KEYS_RESOLVED_READY_FOR_IMPLEMENTATION
```

### Counts should change based on evidence

```text
PHASE2_V001_MANIFEST_COUNT_CHANGE_REQUIRES_OPERATOR_APPROVAL
```

### Evidence remains insufficient

```text
BLOCKED_PHASE2_V001_MANIFEST_KEYS_UNRESOLVED
```

Do not implement source regardless of verdict.

## Phase 1/source/runtime safety

- Keep Phase 1 ProductRoot/checkpoint/DPAPI credential unchanged.
- Do not redeem another Pairing Code.
- Do not modify WPF/API source.
- Do not update WPF label; it remains `prompt011` because no WPF source is changed.
- Do not build/test solutions.
- Do not stop/restart ApiServer, PlatformAppV0, or WPF.
- Do not mutate `enailsalon_phasee1_pos1_pg` or `obm_pos_dev_v0_pg`.

## Local evidence

Create a new versioned local-only folder if detailed evidence is needed:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2ManifestKeysV001\
```

Do not overwrite prior audit folders.
Do not commit local evidence.
Hash every local evidence file and list hashes in the public report.

## Report 016 — 100% detail

Create and push:

```text
report/report016.md
```

Required sections:

1. Verdict.
2. Prompt015 blocker confirmation.
3. Canonical artifact inventory.
4. Development DB read-only proof or exact blocker.
5. Exact existing six parameter rows, sanitized.
6. Current WPF read-path matrix for each parameter.
7. Final parameter-key decision and canonical values.
8. Exact existing three printer types, sanitized.
9. Current WPF read-path matrix for each printer type.
10. Final printer-type decision and placeholder/bind-later fields.
11. Comparison with five source enum values.
12. Whether counts remain exactly 6 and 3.
13. Exact manifest row-count correction for the next implementation prompt.
14. Stable-key/idempotency rules.
15. Remaining operator decision, if any.
16. Exact implementation scope for prompt017.
17. No mutation/source/runtime/secret proof.
18. Local evidence paths and SHA-256.
19. Coordination commit SHA in final response.
