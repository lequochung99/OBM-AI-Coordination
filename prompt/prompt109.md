# Prompt 109 — Remove legacy Firebase/email-password user-secret paths and prove the canonical Pairing Code → WpfJwt WPF authorization boundary

## Starting checkpoint

Report108 passed with:

```text
OBM_MAIN_API_DEV_RESET_MIGRATION_READY_FOR_SYNC_FLOW_AUDIT
```

Coordination references:

```text
report/report108.md
report108 commit: 3e09392d3b86304b92b3b6d4219d80172c5796fd
prompt108 private artifact aggregate SHA-256:
e9d8298486f31f40581cb4445fa0abac25030bd586303098c05e1a9225f0d0ea
```

Report108 proves:

```text
canonical API Development DB reset/recreated UTF8
ExternalDbContext migrations applied from zero
pending migrations = 0
TblEventLog/TblEventDelivery/TblEventDeliveryGroupAck physically proven
focused tests = 6 passed, 0 failed, 0 skipped
API build = 0 warnings, 0 errors
WPF Development DB unchanged and pending migrations = 0
```

Report108 also recorded an operational note that a local ApiServer user-secret appeared to resolve a noncanonical database target.

## Authoritative operator correction

The operator has clarified:

```text
The old user-secret path was legacy Firebase email/password authentication.
Firebase email/password authentication is no longer used.
Delete the retired Firebase/email-password user-secret path and related obsolete runtime paths so they cannot confuse future work.
The canonical WPF authorization flow is Pairing Code redemption followed by a WpfJwt token handoff.
```

This decision is authoritative.

Do not preserve Firebase/email-password compatibility.

Do not align, repair, or continue using the retired Firebase credential user-secret path.

## Canonical WPF authorization flow

The accepted installation/bootstrap flow remains:

```text
PlatformAppV0 administrator authorizes/selects tenant and POS
-> Platform/API creates Pairing Code
-> WPF submits Pairing Code to POST /api/platform-v0/wpf/pairing/redeem
-> API issues a token under the existing WpfJwt authorization contract
-> WPF stores the token through the existing protected machine-state/checkpoint mechanism
-> WPF calls protected GET /api/platform-v0/wpf/bootstrap/me
-> API validates the existing WpfJwt policy/scheme and returns authorized bootstrap identity
-> WPF can restart/resume without Firebase email/password login
```

Use the actual source names and existing storage contract.

Do not invent a new `WpJWT` database table or a second token store merely from informal wording. Audit the real implementation and preserve the accepted `WpfJwt` scheme/policy and protected WPF persistence boundary.

## Architectural locks

### Authentication

There must be no active WPF Firebase email/password path after this task.

Do not create:

```text
new WPF username/password login
new Firebase replacement login
new API-key bootstrap flow
new token-exchange service
new authentication database table
parallel JWT scheme
fallback Firebase credential path
```

Preserve active, unrelated authentication that is still required, including PlatformAppV0 administrator external login when proven active.

### Sync

Do not modify sync architecture in this task.

The only canonical sync flow remains:

```text
WPF domain Save + TblLocalOutbox in one local transaction
-> existing periodic WPF outbox bot
-> existing standard API sync pipeline
-> existing event/delivery transaction path
-> successful API commit
-> existing post-commit SignalR notification
```

Do not create or modify uploader, endpoint, ingest pipeline, bot, ACK path, delivery path, or SignalR publisher.

The transaction-group sync consolidation audit is deferred until this auth/configuration cleanup passes.

## Strict scope

Execute only:

```text
1. Read and verify accepted artifacts and current source/configuration.
2. Inventory legacy Firebase/email-password user-secret keys and every source/DI/runtime reference to them without exposing values.
3. Prove which user-secret/project store owns each retired key.
4. Remove all retired Firebase/email-password keys from the applicable local user-secret store(s).
5. Remove retired Firebase/email-password configuration bindings, options, services, fallback branches, login calls, and misleading tests/docs from active WPF/ApiServer runtime.
6. Audit the report108 noncanonical-DB note and prove whether it was caused by a retired user-secret override or by a different active configuration source.
7. Ensure normal ApiServer Development runtime resolves the canonical API Development database accepted by report108 through the existing current configuration/protected credential mechanism, not through retired Firebase credentials.
8. Prove the existing Pairing Code → redeem → WpfJwt → bootstrap/me boundary remains valid.
9. Run focused tests and build the affected projects.
```

Do not perform:

```text
WPF DB reset
API DB reset
migration generation
persistent seed
sync happy path
15-case sync matrix
POS2 pull/apply/ACK
Category Weight implementation
Customer/Booking Weight implementation
checkout changes
BookingConsole changes
cloud deployment
production deployment
```

## Required evidence intake

Read completely:

```text
prompt/prompt107.md
report/report107.md
prompt/prompt108.md
report/report108.md
OBM_POS_NewChat_Handoff_V001_2026-08-02.md when locally available
```

Read and verify:

```text
E:\Project2026\RecoveryReports\MainWpfDevResetExecutionV002
aggregate SHA-256:
47f68c634a5984611f3cb8b39ba3999f6005a558ad1e0d64bf998f7f4c2a0c58

E:\Project2026\RecoveryReports\MainApiDevResetExecutionV001
aggregate SHA-256:
e9d8298486f31f40581cb4445fa0abac25030bd586303098c05e1a9225f0d0ea
```

Read the active source under:

```text
WPF:
E:\Project2026\4POS\NailSalonNet8

ApiServer:
E:\Project2026\1ApiServer\ApiServer01

PlatformAppV0:
E:\Project2026\PlatformAppV0
```

At minimum inspect:

```text
all csproj UserSecretsId values
all configuration precedence code
all AddUserSecrets calls, explicit or framework-default
all Firebase package references
all Firebase options/configuration classes
all email/password credential models
all WPF login/session/bootstrap services
all HTTP calls containing Firebase/email/password login
all DI registrations for Firebase auth
all fallback authentication branches
PlatformV0WpfBootstrapController
pairing redeem endpoint implementation
WpfJwt authentication registration
WpfInstallBootstrap policy
/api/platform-v0/wpf/bootstrap/me authorization
WPF token persistence/checkpoint implementation
all tests/docs that still state Firebase email/password is required
ExternalDbContext runtime connection resolution
```

Record before editing:

```text
PROMPT107_ARTIFACT_VERIFIED=true
PROMPT108_ARTIFACT_VERIFIED=true
FIREBASE_EMAIL_PASSWORD_STATUS=RETIRED
CANONICAL_WPF_AUTH=PAIRING_CODE_TO_WPFJWT
SYNC_CHANGES=FORBIDDEN
DB_RESET=FORBIDDEN
```

## Phase 1 — Safe user-secret inventory

For each project with a `UserSecretsId`, list key names only.

Do not print values.

Create an inventory containing:

```text
Project
UserSecretsId present yes/no
Key name
Owning options/config class
Read call sites
Runtime reachable yes/no
Purpose
Classification
Removal decision
```

Allowed classifications:

```text
LEGACY_FIREBASE_EMAIL_PASSWORD_REMOVE
LEGACY_FIREBASE_SUPPORT_REMOVE
OBSOLETE_DB_OVERRIDE_REMOVE
ACTIVE_PLATFORM_ADMIN_AUTH_PRESERVE
ACTIVE_POSTGRESQL_PROTECTED_CONFIG_PRESERVE
UNRELATED_ACTIVE_SECRET_PRESERVE
UNKNOWN_BLOCK_AND_PROVE
```

The operator instruction to “delete all” applies to the retired Firebase/email-password user-secret path and its obsolete overrides, not to unrelated active secrets that are demonstrably required by current PlatformAppV0 administrator login or current protected PostgreSQL operation.

Do not delete an unrelated active secret merely because it shares the .NET user-secret store.

If a key cannot be classified from complete call-site evidence, stop before deletion with:

```text
BLOCKED_LEGACY_SECRET_CLASSIFICATION
```

and provide key name plus sanitized call-chain evidence, never its value.

## Phase 2 — Remove legacy Firebase/email-password secrets

Remove every key classified as:

```text
LEGACY_FIREBASE_EMAIL_PASSWORD_REMOVE
LEGACY_FIREBASE_SUPPORT_REMOVE
OBSOLETE_DB_OVERRIDE_REMOVE
```

Use the normal `dotnet user-secrets remove` or `clear` boundary for the exact project store, without echoing values.

Use `clear` only when 100% of keys in that exact project store are proven retired. Otherwise remove keys individually.

Required proof:

```text
retired key names existed before yes/no
retired key names absent after removal
no secret values captured
active unrelated key names preserved when required
```

Do not commit user-secret files or values.

## Phase 3 — Remove legacy source/runtime paths

Remove or disable all active production references whose only purpose is retired Firebase email/password WPF authentication.

This may include, when proven:

```text
Firebase auth packages
Firebase options classes
email/password credential DTOs
Firebase login services
WPF login dialogs or hidden startup branches
ApiServer Firebase token exchange/validation branches used only by the retired WPF path
DI registrations
configuration validation that requires Firebase email/password keys
fallback paths when Pairing Code/WpfJwt is unavailable
obsolete documentation and focused tests
```

Requirements:

```text
no active WPF startup path requests Firebase email/password
no API startup failure caused by missing retired Firebase keys
no normal WPF authorization fallback to Firebase
no source-embedded replacement credentials
no new authentication mechanism
```

Preserve unrelated active Firebase usage only if complete evidence proves it serves a different current product flow. Document it explicitly. Do not assume package name alone proves safe removal.

## Phase 4 — Resolve the report108 runtime DB note correctly

Audit exact ApiServer Development configuration precedence.

Determine which source currently selects the normal runtime database:

```text
appsettings.Development.json
environment variable
launchSettings.json
script-generated environment
protected local configuration
.NET user-secret override
other existing source
```

Required result:

```text
normal ApiServer Development runtime resolves the canonical API Development DB accepted by report108
provider = Npgsql/PostgreSQL
host = loopback or approved local Development
pending migrations = 0
runtime DB differs from WPF Development DB
runtime DB differs from maintenance DB
no retired Firebase/email-password secret participates in DB resolution
```

If an obsolete user-secret DB override is proven, remove it rather than aligning or preserving it.

Use the existing canonical prompt103/prompt108 configuration or script boundary to resolve the API Development DB. Do not create a second configuration framework.

Do not print the complete connection string, username, password, passfile contents, or protected credential value.

Start the actual ApiServer Development project on loopback only for a short-lived proof, then stop it cleanly.

If the canonical DB cannot be resolved without the obsolete override, stop with:

```text
BLOCKED_API_CANONICAL_RUNTIME_DB_RESOLUTION
```

and identify the exact missing active configuration boundary. Do not restore Firebase credentials.

## Phase 5 — Prove the canonical WpfJwt boundary

Audit and prove the existing call chain:

```text
Pairing Code creation
-> POST /api/platform-v0/wpf/pairing/redeem
-> WpfJwt issuance
-> required scope/tenant/POS/installation claims
-> WPF protected token persistence/checkpoint
-> Authorization: Bearer token on GET /api/platform-v0/wpf/bootstrap/me
-> WpfJwt authentication scheme/policy validation
-> authorized response
-> restart/resume without Firebase email/password
```

Use the actual existing endpoint and policy names.

Do not broaden token scope or lifetime.

Do not weaken authentication or mark protected endpoints AllowAnonymous.

Do not expose a token in artifacts or reports.

Focused proof may use existing tests and sanitized runtime markers. A new physical Pairing Code redemption is not required when the accepted current flow can be proven without mutating tenant/POS state; if a physical test is used, it must be isolated Development-only and must not expose identifiers or credentials.

Required negative proof:

```text
Firebase email/password absent from WPF authorization call chain
missing Firebase keys do not block WPF bootstrap flow
WpfJwt validation remains required
invalid/missing WpfJwt is rejected by bootstrap/me
```

## Phase 6 — Focused tests and builds

Run focused tests for at least:

```text
legacy Firebase key/config absence
API/WPF startup does not require retired Firebase email/password
no Firebase fallback registration
Pairing Code redeem issues WpfJwt under existing contract
bootstrap/me requires valid WpfJwt
invalid/missing token rejected
protected WPF token persistence/checkpoint remains wired
restart/resume contract does not request Firebase credentials
ApiServer Development runtime resolves canonical API DB
API pending migrations = 0
```

Run builds for affected projects:

```text
WPF
ApiServer
PlatformAppV0 only if changed
```

Report actual warnings, errors, passed, failed, and skipped totals.

Build success alone does not override an active legacy path or failed WpfJwt/runtime proof.

## End state

Leave the system in this state:

```text
legacy Firebase email/password user-secret keys removed
obsolete user-secret DB override removed when proven
legacy Firebase WPF auth source/DI/runtime paths removed
no startup dependency on retired Firebase credentials
normal ApiServer Development runtime resolves canonical API Dev DB
canonical API and WPF Development DBs remain migration-current
Pairing Code -> redeem -> WpfJwt -> bootstrap/me remains the only WPF installation/bootstrap authorization flow
no sync architecture changes
ready for the next canonical sync-flow consolidation audit
```

## Required private artifact

Create a new versioned artifact:

```text
E:\Project2026\RecoveryReports\LegacyFirebaseUserSecretRemovalV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
ARTIFACT_VERIFICATION.md
USER_SECRETS_KEY_INVENTORY.csv
USER_SECRETS_CLASSIFICATION.md
LEGACY_SECRET_REMOVAL_PROOF.md
FIREBASE_SOURCE_CALL_CHAIN_BEFORE.md
FIREBASE_SOURCE_CALL_CHAIN_AFTER.md
FIREBASE_PACKAGE_DI_REMOVAL.md
RUNTIME_CONFIG_PRECEDENCE.md
API_CANONICAL_RUNTIME_DB_PROOF.md
WPFJWT_CALL_CHAIN.md
WPFJWT_NEGATIVE_PROOF.md
FOCUSED_TEST_OUTPUT.txt
FINAL_STATE.md
BEFORE_CODE.md
AFTER_CODE.md
UNIFIED_DIFF.patch
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

Do not overwrite or delete earlier artifacts.

## Mandatory private evidence

For every changed production file/method include:

```text
repository-relative path
class/method or configuration key
line range
complete BEFORE body or relevant complete configuration section
complete AFTER body or relevant complete configuration section
callers/callees
DI/package/config registration evidence
unified diff
```

For user-secrets include key names and classification only. Never include values.

## Public report

Create and push only:

```text
report/report109.md
```

Include:

```text
Verdict
Prompt107 artifact SHA verified yes/no
Prompt108 artifact SHA verified yes/no
Projects with UserSecretsId count
Legacy Firebase/email-password key count before
Legacy Firebase/email-password key count after
Legacy Firebase support key count before
Legacy Firebase support key count after
Obsolete DB override present yes/no
Obsolete DB override removed yes/no/not-applicable
Active unrelated secrets preserved yes/no/not-applicable
Firebase packages removed count
Firebase options/services/DI paths removed count
Active WPF Firebase email/password runtime paths after task
ApiServer startup requires retired Firebase keys yes/no
Canonical API Development runtime DB proof yes/no
API pending migrations count
Pairing Code redeem path proof yes/no
WpfJwt issuance proof yes/no
WpfJwt bootstrap/me validation proof yes/no
Invalid/missing WpfJwt rejected yes/no
WPF protected token persistence/checkpoint proof yes/no
Sync flow code changed yes/no
WPF DB mutated yes/no
API DB destructively mutated yes/no
Persistent auth/sync test data seeded yes/no
Focused test totals
WPF build totals
API build totals
PlatformAppV0 build totals or not changed
Production/customer/reference DB mutated yes/no
Private artifact yes/no
Aggregate SHA-256
```

Do not expose secret values, complete connection strings, JWTs, raw tenant/device identifiers, or private business payloads.

## Verdicts

PASS only when the retired Firebase/email-password path is absent and the existing WpfJwt flow remains valid:

```text
OBM_LEGACY_FIREBASE_USER_SECRET_REMOVED_WPFJWT_CANONICAL_READY_FOR_SYNC_FLOW_AUDIT
```

Narrow blockers only:

```text
BLOCKED_LEGACY_SECRET_CLASSIFICATION
BLOCKED_LEGACY_SECRET_REMOVAL
BLOCKED_FIREBASE_RUNTIME_PATH_REMOVAL
BLOCKED_API_CANONICAL_RUNTIME_DB_RESOLUTION
BLOCKED_API_CANONICAL_RUNTIME_STARTUP
BLOCKED_WPFJWT_ISSUANCE_PROOF
BLOCKED_WPFJWT_BOOTSTRAP_VALIDATION
BLOCKED_AUTH_CLEANUP_FOCUSED_TESTS
```

A blocked result must identify the exact key name or source boundary, complete sanitized call chain, exact failed test/method, sanitized exception chain, and SQLSTATE when available.

Do not return a generic auth/configuration blocker.
