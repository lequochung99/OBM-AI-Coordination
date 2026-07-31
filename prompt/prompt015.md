# Prompt 015 — Implement Phase 2 atomic baseline seed v001

## Authoritative inputs

Read completely before changing source:

```text
prompt/prompt012.md
report/report012.md
prompt/prompt013.md
report/report013.md
prompt/prompt014.md
report/report014.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Read the local-only evidence produced by prompt014:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2SeedAuditV003\
```

Verify the hashes against `report/report014.md` before using those artifacts. Do not copy local evidence into the public coordination repository.

Prompt014 verdict:

```text
PHASE2_BASELINE_SEED_AUDIT_READY_FOR_IMPLEMENTATION_PROMPT
```

Phase 1 physical closure remains authoritative:

```text
PHASE1_WPF_API_AUTHORIZATION_AND_MACHINE_PERSISTENCE_PASS_DATABASE_NOT_STARTED
```

## Objective

Implement InstallationV0 Phase 2 baseline database installation so that:

1. Phase 1 protected credential/checkpoint is revalidated first;
2. target PostgreSQL schema is an explicit prerequisite;
3. the approved baseline rows, matching `TblLocalOutbox` rows, and Phase 2 completion marker are written in **one PostgreSQL transaction**;
4. any error rolls back the entire seed;
5. retry and same-version replay are deterministic and idempotent;
6. no employee, catalog, customer, transaction, booking, output, payment, queue, payroll, or historical business data is seeded;
7. WPF visibly identifies the active binary as `prompt015`.

This prompt implements the code and focused tests. It must not mutate the reference database `enailsalon_phasee1_pos1_pg`.

## Operator decisions locked for v001

Use these decisions without asking again:

### Marker

Approve the proposed dedicated marker table:

```text
dbo.TblInstallationV0Phase2SeedVersion
```

Manifest/version:

```text
phase2-baseline-seed-v001
```

Do not reuse the absent `TblSystemBaselineVersion` contract for Phase 2 completion.

### Roles

Seed exactly these three default permission/role definitions:

```text
Owner
Admin
Sub_Manager
```

`Sub_Manager` is the physical database representation of the operator-facing `SubAdmin` role.

Do not seed employee rows, PINs, `Manager`, `Staff`, `AI_Assistant`, or `VirtualAnyTechnician` in v001.

### Printer defaults

Include placeholder-safe printer defaults in v001.

Expected baseline count:

```text
3 placeholder printer rows
```

Do not copy physical printer name, IP, Windows path, driver, share name, terminal identifier, or machine-specific reference data from the reference database.

### Turn settings

Defer `TblTurnSetting` entirely from v001.

### TblSetting

Defer `TblSetting` from v001 because the live reference table is empty and no concrete startup requirement was proven.

### Outbox writer

Create a dedicated InstallationV0 Phase 2 deterministic outbox writer/adapter.

Do not directly reuse the legacy demo-seed outbox method as the Phase 2 transaction owner. Reuse only low-level serialization/helper code after proving deterministic payload and same-transaction behavior.

## Approved v001 row groups

The final v001 baseline is limited to:

```text
1 TblTenant row from Phase 1 identity
1 TblPosLocal row from Phase 1 identity
1 TblSetupWeird row
1 TblSetupServicesMethod row
3 TblSetupLoginMethod rows
6 TblSetupPaymentMethod rows
3 TblEmployeePermission rows: Owner, Admin, Sub_Manager
6 approved mandatory TblParameterSetting rows
3 placeholder-safe TblSetupPrinter rows
1 Phase 2 completion marker row
Matching TblLocalOutbox rows for every actual inserted/updated baseline row
```

Expected non-marker baseline rows when all are newly inserted:

```text
25
```

Expected outbox rows when all 25 rows are newly inserted:

```text
25
```

No outbox event for the Phase 2 marker itself unless an existing canonical contract explicitly requires it; default is no marker outbox.

### Exact parameter keys and printer types gate

Before implementation, identify the exact six safe mandatory parameter keys and exact three placeholder printer types from:

```text
Phase2SeedAuditV003\safe-candidate-patterns.tsv
Phase2SeedAuditV003\source-live-comparison.md
current startup/read paths in WPF
existing canonical System Baseline artifacts
```

Do not select six arbitrary rows from the 110-row reference table.

The six parameter keys must be:

- non-secret;
- required by startup or safe baseline behavior;
- stable across tenants;
- not terminal/payment gateway credentials;
- not salon historical/custom values;
- not duplicated ambiguous keys without an explicit scope discriminator.

Printer rows must use safe printer-type definitions only and blank/neutral bind-later fields.

If exact six parameter keys or three printer types cannot be proven without guessing, stop before DB/source implementation with:

```text
BLOCKED_PHASE2_V001_MANIFEST_KEYS_UNRESOLVED
```

Report the safe key/type names if non-sensitive; never report secret/private values.

## Phase 1 freeze and prerequisite

Do not modify, delete, rotate, move, or recreate:

```text
E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
InstallationV0\Checkpoints\api-authorized.json
InstallationV0\Secrets\bootstrap-credential.dpapi
LocalInstallationGuid
Tenant/POS/InstallationAttempt identity
```

Before enabling Phase 2 execution, WPF must:

```text
read ApiAuthorized checkpoint
DPAPI-unprotect WpfJwt in memory
GET protected /api/platform-v0/wpf/bootstrap/hello
verify exact controller marker
GET /api/platform-v0/wpf/bootstrap/me
verify Tenant/POS/Attempt/LocalInstallation identity
```

Do not request or redeem another Pairing Code.

If Phase 1 credential is expired or invalid, fail closed with an explicit result. Do not erase Phase 1 artifacts and do not silently begin Phase 2.

## Source boundaries

Primary permitted source boundaries:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0
E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0
E:\Project2026\4POS\NailSalonNet8\MyData
E:\Project2026\4POS\NailSalonNet8\SeedDb
E:\Project2026\4POS\NailSalonNet8\Services\Bootstrap
```

Schema/migration test support may touch the canonical WPF migration location only after identifying it exactly.

ApiServer/PlatformAppV0 source may be changed only if a narrowly required Phase 1 revalidation contract correction is proven. Phase 2 seed logic belongs in WPF/local installation code, not PlatformAppV0.

Do not refactor unrelated POS business/runtime modules.

## WPF build label

Because prompt015 changes WPF, update the single canonical label constant to:

```csharp
public const string CoordinationPromptLabel = "prompt015";
```

Visible requirements:

```text
Window title: OBM InstallationV0 Phase 1/2 - prompt015
Build label: prompt015
```

Focused tests must prove `prompt011` is absent from active title/header/build-info source.

## Schema prerequisite boundary

The seed transaction must not silently create the full POS schema.

Implement an explicit Phase 2 preflight that verifies:

- target database is the approved target, never the reference DB;
- expected POS schema/migrations are present;
- marker table schema is present;
- target database is eligible for v001;
- protected database names are rejected;
- reference database name `enailsalon_phasee1_pos1_pg` is always rejected as a target.

Introduce the marker table only through the canonical WPF schema/migration mechanism:

```text
dbo.TblInstallationV0Phase2SeedVersion
```

Minimum marker fields should support:

```text
ManifestVersion
SeedBatchGuid
TenantGuid
PosGuid
LocalInstallationGuid
InstallationAttemptGuid
AppliedAtUtc
ManifestHash
BaselineRowCount
OutboxRowCount
Status/Completion proof
```

Use exact PostgreSQL-compatible types and constraints. Add a unique key that prevents duplicate completion for the same manifest/tenant/POS identity.

Schema migration is a prerequisite outside the baseline seed transaction. The **baseline row seed itself** remains one transaction.

## Target database safety

Never mutate:

```text
enailsalon_phasee1_pos1_pg
obm_pos_v1_local_pos1_pg
any production/reference/protected DB
```

Do not hard-code a target database connection string or password.

Use the canonical target database configuration/installation input path. Report only sanitized host/database/role classification; no credential or connection string.

A target is eligible only when:

- schema is at the expected migration level;
- no newer Phase 2 marker exists;
- no conflicting tenant/POS identity exists;
- no partial approved baseline state exists without a valid marker;
- operational/transaction tables are empty as required by a new installation.

## Canonical manifest implementation

Create a versioned manifest in source, preserving future versions:

```text
InstallationV0/Phase2/Manifests/phase2-baseline-seed-v001/
```

Include machine-readable manifest plus typed C# representation. Do not overwrite future or prior versions.

Manifest must define for every row group:

```text
table/entity
stable key
expected count
identity source
value source
normalized value hash rule
logical insert order
outbox rule
verification rule
```

All GUIDs that need deterministic repeatability must be derived deterministically from manifest version + TenantGuid/POS identity + stable row key, or must be persisted once and reused. Do not generate new random row identities on idempotent replay.

## One PostgreSQL transaction — hard requirement

Use one connection/DbContext transaction for all baseline mutations:

```text
BEGIN
  acquire pg_advisory_xact_lock
  validate no conflicting/newer/partial marker state
  upsert/insert TblTenant
  upsert/insert TblPosLocal
  upsert lookup rows
  upsert singleton settings
  upsert six approved parameter rows
  upsert three safe printer rows
  insert matching deterministic TblLocalOutbox rows for actual changes
  insert completion marker last
  read back stable keys/counts/invariants
COMMIT
```

All operations must participate in the same underlying Npgsql transaction. Do not open a second hidden connection for outbox or verification.

Any failure must cause:

```text
ROLLBACK all baseline rows
ROLLBACK all outbox rows
ROLLBACK marker
Phase 1 checkpoint unchanged
Phase 2 remains NotStarted or FailedRetryable
```

No partial commit, no catch-and-continue, and no marker before successful readback.

## Advisory lock

Use transaction-scoped advisory lock derived deterministically from:

```text
InstallationV0 + Phase2 + TenantGuid + PosGuid + manifest version
```

Second instance behavior must be explicit and testable:

- serialize safely; or
- fail retryable with a clear result code.

Do not use a global lock that blocks unrelated tenants/POS installations.

## Stable-key conflict and idempotency rules

### Same version already complete

- verify marker and every manifest row;
- verify no missing/changed expected row;
- return idempotent success;
- create no duplicate baseline or outbox rows.

### Partial rows without marker

Fail closed:

```text
PHASE2_PARTIAL_BASELINE_RECOVERY_REQUIRED
```

Do not blind-merge or delete existing rows.

### Conflicting stable-key row

Fail closed with table and safe stable key only:

```text
PHASE2_BASELINE_CONFLICT
```

Do not print private values.

### Newer marker exists

Fail closed:

```text
PHASE2_NEWER_MANIFEST_PRESENT
```

### Transaction rollback and retry

A retry after a confirmed rollback must be safe with the same Phase 1 identity and manifest version.

## TblLocalOutbox policy

Create one deterministic outbox row only for each baseline row actually inserted or materially updated.

No outbox for a no-op replay.

All outbox rows must use:

```text
same SeedBatchGuid / transaction identity
TenantGuid from Phase 1
SourceClientId from Phase 1 POS identity
canonical entity/table name
canonical I/U operation
sanitized deterministic payload
stable idempotency identity
```

Do not include:

- password/PIN;
- Pairing Code/WpfJwt;
- terminal/payment credentials;
- customer/employee private data;
- reference DB row dumps;
- physical printer binding values.

The outbox writer must use the same DbContext/transaction as seed rows.

## UI/flow requirements

After Phase 1 resume is verified, InstallationV0 must show a distinct Phase 2 section:

```text
Phase 1 API Authorization: Complete
Phase 2 Local Database Baseline: Not Started / Running / Complete / Blocked
Target DB classification
Manifest version: phase2-baseline-seed-v001
```

Provide an explicit operator action such as:

```text
Install Local Database Baseline
```

Do not auto-run Phase 2 merely because WPF starts.

During execution show safe proof items separately:

```text
Phase 1 prerequisite revalidated
Target DB eligibility verified
Schema prerequisite verified
Advisory lock acquired
Single transaction started
Tenant/POS rows verified
Lookup rows verified
Settings/parameters verified
Printer placeholders verified
Outbox row mapping verified
Completion marker verified
Transaction committed
Operational tables remained empty
```

On rollback show exact safe result code and confirm Phase 1 remains intact.

## Explicit exclusions

Do not seed or modify:

```text
TblEmployee or staff/PIN rows
TblServiceCategory / TblService / TblProduct
TblCustomer*
TblGiftCard*
TblInvoice*
TblOutputInfo*
bookings/appointments
terminal payment/runtime data
queue/turn/payroll runtime history
TblTurnSetting in v001
TblSetting in v001
historical TblLocalOutbox rows
PlatformEnrollment/permanent device credentials
```

## Testing requirements

Focused unit/contract tests must cover:

- prompt015 label;
- exact manifest table/row counts;
- exact six parameter keys and three printer types;
- deterministic row GUID/idempotency identity;
- Phase 1 prerequisite must pass before Phase 2;
- no second Pairing Code redeem;
- protected hello and bootstrap/me revalidation;
- target/reference/protected DB rejection;
- marker schema prerequisite;
- one transaction owns seed + outbox + marker;
- failure injection at every seed group rolls back everything;
- failure before marker leaves zero committed baseline/outbox rows;
- marker written last;
- readback before commit;
- same-version replay adds zero rows/outbox;
- partial rows without marker block;
- conflicting rows block;
- newer marker blocks;
- advisory-lock concurrency behavior;
- operational tables receive zero rows;
- Phase 1 checkpoint/credential remain unchanged on Phase 2 failure.

Run required builds/tests:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

Run any focused migration/schema tests required by the marker addition.

Do not run Phase 2 against the reference database.

## PostgreSQL physical test boundary

A disposable physical PostgreSQL E2E is allowed only if an already-approved test harness and non-production credential path exist.

Disposable DB name must start with:

```text
obm_pos_installationv0_phase2_test_
```

Hard reject all other names for automated prompt015 mutation.

Physical E2E must prove:

1. schema prerequisite applied through canonical test path;
2. empty target baseline counts;
3. one transaction commits 25 baseline rows, expected outbox rows, and one marker;
4. second run is idempotent with zero additional rows;
5. injected failure rolls back baseline/outbox/marker;
6. excluded operational tables remain zero.

If no approved disposable test credential exists, do not ask for or use administrator password. Finish with build/test evidence and verdict `READY_FOR_USER_PHYSICAL_TEST`, not full PASS.

Do not automatically drop or preserve a disposable DB ambiguously. Report the exact sanitized disposition and require explicit cleanup policy from the existing test harness.

## Runtime handoff

At the end:

- stop only stale WPF/ApiServer/PlatformAppV0 processes that Codex itself owns or that block the required build;
- do not leave WPF running;
- if ApiServer/PlatformAppV0 were changed or stopped, restart latest Debug instances and prove PID/path/start time/ports;
- hand WPF back for Visual Studio Debug physical testing;
- visible WPF label must be `prompt015`.

## Git safety

`E:\Project2026` remains a shared dirty parent repository with no source remote.

Do not:

```text
git add .
git add -A
reset
clean
stash
checkout/restore unrelated changes
push source
```

List exact source files changed. No source commit is required if safe isolation is not possible.

Commit only the coordination report:

```text
report/report015.md
```

## Report 015 — 100% detail

Create:

```text
report/report015.md
```

Must include:

1. Verdict.
2. Phase 1 freeze/prerequisite proof.
3. Exact operator decisions applied.
4. Exact six parameter keys and three printer types with evidence source.
5. Manifest files and hash/version.
6. Marker schema/migration details.
7. Exact files changed.
8. Exact row groups/counts/stable keys.
9. Transaction ownership proof.
10. Advisory-lock implementation.
11. Outbox writer and deterministic payload/idempotency design.
12. Marker-last/readback-before-commit proof.
13. Rollback/failure-injection proof.
14. Same-version replay proof.
15. Partial/conflicting/newer-state behavior.
16. Explicit excluded-table proof.
17. Build/test commands and counts.
18. Disposable physical E2E evidence or exact reason it was not run.
19. Runtime/process handoff.
20. Prompt015 label proof.
21. Confirmation no reference DB mutation/no secret leakage.
22. Source Git/no-push confirmation.
23. Exact user physical test steps.
24. Coordination commit SHA in final response.

## Valid verdicts

Implementation and focused tests complete, physical target test pending:

```text
PHASE2_ATOMIC_BASELINE_SEED_V001_READY_FOR_USER_PHYSICAL_TEST
```

Implementation plus approved disposable physical E2E complete:

```text
PHASE2_ATOMIC_BASELINE_SEED_V001_DISPOSABLE_E2E_PASS_READY_FOR_USER_TEST
```

Manifest keys unresolved:

```text
BLOCKED_PHASE2_V001_MANIFEST_KEYS_UNRESOLVED
```

Implementation blocked:

```text
BLOCKED_PHASE2_ATOMIC_BASELINE_SEED_IMPLEMENTATION
```
