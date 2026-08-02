# Prompt 118 Addendum — Replace ApiServer .NET User Secrets with one protected full-runtime configuration boundary, align the canonical loopback endpoint, then prove WPF token resume

This addendum is binding and must be read together with:

```text
prompt/prompt118.md
prompt/prompt118_MINIMAL_WPF_STARTUP_RECOVERY_ADDENDUM.md
prompt/prompt118_API_NO_USER_SECRETS_AND_WPF_TOKEN_RESUME_ADDENDUM.md
```

Where this addendum conflicts with earlier prompt118 language, this addendum controls.

## Operator-approved diagnosis

The latest direct diagnosis is accepted in the following narrow form:

```text
The exception shown in Program.cs is not a WpfJwt client-token failure.
It is the full ApiServer PostgreSQL runtime-credential guard.
```

Current observed behavior:

```text
full ApiServer starts from Visual Studio
-> DatabaseProvider resolves PostgreSQL
-> ConnectionStrings:PostgreSqlConnection falls back to appsettings.json
-> appsettings.json contains placeholder user/password values
-> Program.cs detects placeholder credentials
-> Program.cs throws InvalidOperationException
-> ApiServer exits before it can serve WPF
```

Observed guard message:

```text
Local API is using placeholder DB credentials.
Start with start-api-local.ps1 or set user-secrets / environment variables.
```

The current .NET User Secrets store does not contain:

```text
ConnectionStrings:PostgreSqlConnection
```

Therefore the full API currently cannot obtain its PostgreSQL runtime credential when launched directly through the operator's normal Visual Studio profile.

## Authoritative correction to the diagnosis

The conclusion that the API needs protected secret material is correct.

The conclusion that `.NET User Secrets` must remain the canonical mechanism is **not accepted**.

The operator decision is:

```text
Do not use .NET User Secrets for normal ApiServer runtime configuration.
```

This does not mean storing secrets in source. It means replacing the development-only `.NET User Secrets` dependency with one existing or minimally extended protected machine-local configuration boundary that works consistently for:

```text
Visual Studio Start/Debug
start-api-local.ps1
other canonical local full-API launches
```

The final design must still securely supply the active ApiServer secrets that the selected full-runtime profile genuinely requires, for example when proven:

```text
PostgreSQL runtime credential
JWT signing/validation material
hash/signing keys
Google/admin/platform configuration required by the same host/profile
```

Do not conflate these server-side secrets with WPF Pairing Code, WpfJwt, access token, refresh token, or runtime Provider authentication.

## Credential-boundary separation

Preserve these separate purposes:

```text
1. ApiServer PostgreSQL runtime credential
   - allows ApiServer to connect to obm_api_dev_v0_pg

2. ApiServer migration/provisioning credential
   - protected administration/migration boundary
   - never used by normal request processing

3. WPF installation credential
   - Pairing Code -> redeem -> WpfJwt -> bootstrap/me

4. WPF retained access/refresh/session state
   - protected machine-side state used by the canonical Provider/resume path

5. Platform administrator authentication
   - separate external/admin login boundary
```

A credential from one category must not silently replace another.

## Canonical full-API endpoint drift to resolve

The current local lanes appear inconsistent:

```text
start-api-local.ps1: http://127.0.0.1:5193
WPF/Phase1 history: http://127.0.0.1:7161
PlatformAppV0 Phase1 profile: 7161, Phase1-only behavior
```

The full ApiServer and WPF must use one proven canonical loopback base URL for normal Development runtime.

Do not guess from port numbers alone. Audit every active caller and profile, then choose the smallest correction that produces exactly one normal full-API endpoint.

The final normal WPF runtime must not point to a Phase1-only API profile that bypasses full database startup.

A Phase1-only profile may remain only when it is explicitly test-only and cannot be selected by the operator's normal WPF/ApiServer startup lane.

## Architecture locks

### No .NET User Secrets for ApiServer runtime

After this task, normal ApiServer startup must have:

```text
active ApiServer runtime reads from .NET User Secrets = 0
required ApiServer runtime keys supplied by the canonical protected machine-local source
```

Do not solve the current crash by running:

```text
dotnet user-secrets set ConnectionStrings:PostgreSqlConnection ...
```

Do not add or restore any secret in `.NET User Secrets`.

Do not store a password or complete connection string in:

```text
appsettings.json
appsettings.Development.json
launchSettings.json
source code
PowerShell source
Git repository
coordination report
```

### Preserve the canonical WPF Provider

The caller does not manage token details.

```text
periodic worker / WPF feature
-> canonical Provider
-> Provider resolves/resumes credential internally
-> Provider attaches auth/application-identity headers internally
-> API authenticates the application
```

Do not add manual token reads, manual Authorization headers, a new Provider, a new auth endpoint, or a direct HttpClient bypass.

### Freeze unrelated work

Do not inspect, design, implement, or modify:

```text
Category Weight
Booking Weight
Price Weight save semantics
TblTenantPosDevice
TblPosLocal routing
CompanionApp/terminal modeling
API destination routing
sync E2E
POS2 pull/apply/ACK
API schema/migrations/role contract except read-only verification
```

Do not reset/drop/recreate WPF or API databases.

## Strict scope

Execute only:

```text
1. Reproduce the full ApiServer Visual Studio crash exactly.
2. Audit ApiServer configuration precedence and all active secret requirements by key name only.
3. Identify the existing protected machine-local source already used by start-api-local.ps1 or other accepted local tooling.
4. Replace the ApiServer .NET User Secrets runtime dependency with that one protected source.
5. Make Visual Studio and start-api-local.ps1 resolve the same provider, host, DB, runtime role, protected source, and full-API port.
6. Remove obsolete ApiServer UserSecretsId/AddUserSecrets dependency only after every required active key has a protected replacement or is proven profile-inapplicable.
7. Start the full ApiServer directly from the operator-equivalent Visual Studio profile and prove health/readiness.
8. Only after the full API is healthy, inspect the retained WPF access/refresh/session state without redeeming again.
9. Correct the smallest token-persistence/ProductRoot/resume defect when direct evidence proves one.
10. Physically prove normal WPF startup reaches MainWindow and survives restart.
```

Do not stop after merely making `/health` return 200. Complete the WPF token-resume/startup proof unless a narrow blocker is reached.

## Phase 1 — Capture the exact current full-API failure

Use the operator-equivalent Visual Studio full-API profile.

Record direct sanitized evidence:

```text
API_STARTUP_PROJECT=<exact project>
API_VISUAL_STUDIO_PROFILE=<exact profile>
API_ENVIRONMENT=<exact environment>
PROGRAM_GUARD_CLASS_METHOD_LINE=<exact location>
DATABASE_PROVIDER=<safe provider>
CONNECTION_STRING_SOURCE_ORDER=<source names only>
APPSETTINGS_VALUE_CLASSIFICATION=PLACEHOLDER/REAL/MISSING
USERSECRETSID_PRESENT=yes/no
USER_SECRETS_KEY_NAMES=<names only, no values>
CONNECTIONSTRINGS_POSTGRES_KEY_PRESENT_IN_USER_SECRETS=yes/no
EXACT_EXCEPTION_TYPE=<type>
EXACT_SANITIZED_MESSAGE=<message>
PROCESS_EXIT_CODE=<value>
```

Prove that the exception occurs before normal API readiness and before any WPF request can be served.

## Phase 2 — Inventory active server-side secret requirements

Inventory every secret/config key read by the selected full ApiServer profile.

For each key record:

```text
key name
owning class/method/options type
configuration source currently used
required at full-API startup yes/no
required only by a separate profile yes/no
active/retired/unknown classification
replacement protected-source decision
```

At minimum audit:

```text
ConnectionStrings:PostgreSqlConnection
JWT signing/validation keys
hash/signature keys
Google/admin/platform keys used by the same host
FirebaseAdmin support only when still active and distinct from retired Firebase email/password
```

Never output values.

Allowed classifications:

```text
ACTIVE_FULL_API_RUNTIME_PROTECTED
ACTIVE_SEPARATE_PROFILE_PROTECTED
RETIRED_REMOVE
NOT_REQUIRED_FOR_SELECTED_PROFILE
UNKNOWN_BLOCK_AND_PROVE
```

Do not remove a still-active key before its protected replacement is proven.

## Phase 3 — Identify and reuse one protected machine-local source

Audit the exact source used by:

```text
start-api-local.ps1
accepted protected environment import tooling
DPAPI-protected local state when present
PGPASSFILE/protected PostgreSQL integration when present
other already accepted noninteractive local credential loaders
```

Required decision record:

```text
PROTECTED_SOURCE_NAME=<name only>
PROTECTED_SOURCE_IMPLEMENTATION=<exact script/class/helper>
SOURCE_IS_MACHINE_LOCAL=yes/no
SOURCE_IS_NONINTERACTIVE=yes/no
SOURCE_IS_NOT_GIT_TRACKED=yes/no
SOURCE_PROTECTS_VALUES=yes/no
VISUAL_STUDIO_CAN_USE_SOURCE=yes/no
START_SCRIPT_CAN_USE_SOURCE=yes/no
```

Prefer reuse of the already accepted protected source. Do not create a second secret store or parallel configuration framework.

If the current source is PowerShell-only, make the smallest existing-boundary change needed so the operator-equivalent Visual Studio process receives or reads the same protected configuration without copying secret values into source or launchSettings.

Acceptable implementations only when directly supported by current project conventions include:

```text
an existing protected local configuration loader shared by script and host
an existing central environment import invoked before ApiServer startup
an existing DPAPI-protected local file referenced by a non-secret path
an existing PGPASSFILE-based PostgreSQL credential boundary combined with non-secret DB metadata
```

Do not invent a broad new secrets platform.

If no protected source exists for an active required key, stop with:

```text
BLOCKED_API_PROTECTED_RUNTIME_SECRET_SOURCE_MISSING
```

Report the missing key name and required setup boundary only. Do not request the operator to paste a secret into chat.

## Phase 4 — Remove ApiServer .NET User Secrets dependency

After replacement is proven:

```text
remove active ApiServer AddUserSecrets/runtime dependency
remove ApiServer UserSecretsId when it no longer owns any active required key
remove misleading startup text that instructs the canonical operator path to use user-secrets
retain fail-closed placeholder detection
replace the message with the exact canonical protected-source setup instruction
```

Requirements:

```text
ApiServer active user-secret key reads after task = 0
ApiServer full runtime succeeds with user-secret store empty/absent
missing protected source fails fast with a precise sanitized error
no hidden password prompt
no fallback to placeholder credentials
```

Do not change PlatformAppV0's separate project secret mechanism in this task unless the same full ApiServer host directly owns those keys and a complete protected replacement is proven. Record any remaining separate-project dependency explicitly rather than deleting it blindly.

## Phase 5 — Align one canonical full-API Development endpoint

Audit:

```text
launchSettings.json profiles
start-api-local.ps1 URL binding
WPF API Base URL resolver
InstallationV0 API base URL
canonical Provider base address
PlatformAppV0 Phase1-only profile
health/readiness callers
```

Produce:

```text
FULL_API_CANONICAL_BASE_URL=<one loopback URL>
FULL_API_VISUAL_STUDIO_URL=<URL>
FULL_API_START_SCRIPT_URL=<URL>
WPF_NORMAL_RUNTIME_API_URL=<URL>
PHASE1_ONLY_PROFILE_URL=<URL or NOT_RETAINED>
```

The first three normal full-runtime values must be identical.

Choose the canonical port by complete call-chain evidence and smallest regression risk. Do not preserve 5193 and 7161 as two interchangeable normal full-API endpoints.

Requirements:

```text
one full API process
one normal full API port
one WPF normal API base URL
no normal caller targets the Phase1-only bypass profile
```

## Phase 6 — Physical full-API acceptance

Start the full API in both ways, one at a time:

```text
A. operator-equivalent Visual Studio Start/Debug
B. start-api-local.ps1
```

For both prove:

```text
same canonical protected source
same PostgreSQL provider
same loopback host/port
same DB = obm_api_dev_v0_pg
same normal runtime role
pending migrations = 0 through the accepted proof boundary
health/readiness succeeds
full grouped API routes are loaded
Platform/bootstrap routes required by WPF are loaded
no placeholder guard exception
no user-secrets dependency
no secret values logged
```

Stop stale duplicate API instances before each proof. Do not run a Phase1-only process while claiming full-API readiness.

## Phase 7 — Audit WPF retained token/session state only after API is healthy

Do not redeem another Pairing Code before completing this audit.

Inspect without exposing values:

```text
resolved WPF ProductRoot
protected token/session file names and existence
access-token presence marker
refresh-token presence marker
last-write timestamp
DPAPI decrypt success/failure
credential expiration metadata
installation checkpoint status
current launch ProductRoot matches save ProductRoot yes/no
exact load method
exact save method
exact clear/delete methods and callers
```

Search every code path that clears or deletes token/session state.

Network/API/bootstrap failure must not be treated as revocation and must not clear retained credentials.

Classify exactly one primary outcome:

```text
T1_TOKEN_STATE_PRESENT_AND_VALID_API_WAS_ONLY_OFFLINE
T2_ACCESS_EXPIRED_REFRESH_PRESENT_REFRESH_PATH_DEFECT
T3_TOKEN_STATE_DELETED_BY_EXACT_CALL
T4_TOKEN_STATE_PRESENT_WRONG_PRODUCTROOT_SELECTED
T5_TOKEN_STATE_PRESENT_DPAPI_OR_CONTRACT_FAILURE
T6_REDEEM_RETURNED_BUT_PERSISTENCE_FAILED
T7_CURRENT_CONTRACT_HAS_NO_REFRESH_TOKEN
T8_OTHER_EXACTLY_PROVEN_TOKEN_STATE_DEFECT
```

Do not assume token deletion from the InstallationV0 screen alone.

If token state is genuinely absent and no safe retained resume path exists, stop with:

```text
BLOCKED_WPF_RETAINED_CREDENTIAL_ABSENT
```

Do not automatically redeem or fabricate completion state.

## Phase 8 — Restore normal WPF MainWindow startup

Apply only the smallest proven correction, for example:

```text
correct the ProductRoot selected by the normal profile
prevent recoverable API outage from clearing token state
repair existing refresh/resume call ownership
correct a protected-state load/save path mismatch
correct startup routing after valid local installation state
```

Preserve:

```text
Pairing Code -> WpfJwt initial bootstrap authorization
canonical Provider ownership of runtime auth headers
local-first MainWindow operation
```

Physical acceptance:

```text
visible label = prompt118
full API is the canonical full runtime on the aligned loopback endpoint
WPF does not redeem again when retained credential/session state is valid
normal WPF launch opens MainWindow directly
InstallationV0 does not replace MainWindow on valid installed-local state
MainWindow remains alive and responsive for at least 60 seconds
close normally
second launch again reaches MainWindow using retained state
```

## Tests and guards

Run focused tests for:

```text
full ApiServer Visual Studio startup without User Secrets
full ApiServer script startup without User Secrets
placeholder config fails closed
missing protected source fails fast
same canonical DB and port across Visual Studio/script/WPF
normal WPF does not target Phase1-only API profile
retained token/session save-load round trip
API outage does not clear retained token/session state
wrong ProductRoot detection
refresh/resume behavior when present
normal installed-local startup reaches MainWindow
```

Build:

```text
ApiServer
WPF
PlatformAppV0 only if directly changed
```

Expected:

```text
all focused tests pass
0 skipped
build errors = 0
```

Physical behavior overrides build/test results.

## Required private evidence additions

Add to the prompt118 artifact:

```text
API_PLACEHOLDER_GUARD_FAILURE.md
API_SECRET_KEY_INVENTORY.csv
API_USERSECRETS_DEPENDENCY_BEFORE.md
CANONICAL_PROTECTED_RUNTIME_SOURCE.md
API_USERSECRETS_DEPENDENCY_AFTER.md
FULL_API_ENDPOINT_INVENTORY.md
FULL_API_ENDPOINT_DECISION.md
VISUAL_STUDIO_FULL_API_STARTUP_PROOF.md
SCRIPT_FULL_API_STARTUP_PROOF.md
WPF_TOKEN_STATE_INVENTORY.md
WPF_TOKEN_CLEAR_CALL_CHAIN.md
WPF_TOKEN_STATE_CLASSIFICATION.md
WPF_RETAINED_RESUME_PROOF.md
```

## Public report additions

`report/report118.md` must include:

```text
Verdict
DB-credential-vs-WpfJwt diagnosis confirmed yes/no
ApiServer UserSecretsId before/after
ApiServer active user-secret key reads after task
Canonical protected runtime source name only
Full API required secret keys classified yes/no
Visual Studio full API noninteractive startup yes/no
start-api-local full API noninteractive startup yes/no
Visual Studio/script same protected source yes/no
Canonical full API base URL sanitized
WPF normal API base URL matches yes/no
Phase1-only profile used by normal WPF yes/no
Canonical API DB proof yes/no
API health/readiness yes/no
API pending migrations count
Placeholder guard triggered after fix yes/no
WPF access-token presence marker
WPF refresh-token presence marker or contract-not-present
WPF token-state classification T1-T8
Token/session clear caused by API outage yes/no
Redeem repeated during task yes/no
WPF ProductRoot match proof yes/no
MainWindow opens directly yes/no
InstallationV0 shown on valid installed-local launch yes/no
60-second stability yes/no
Second-launch proof yes/no
Category Weight changed no
Booking Weight changed no
TblTenantPosDevice changed no
Sync/routing/schema changed no
Operator MainWindow screenshot ready true/false
Manual POS1 test ready false
Private artifact and aggregate SHA-256
```

## Verdicts

PASS:

```text
OBM_FULL_API_PROTECTED_CONFIG_NO_USERSECRETS_AND_WPF_RETAINED_RESUME_MAINWINDOW_READY_FOR_OPERATOR_SCREENSHOT
```

Narrow blockers only:

```text
BLOCKED_API_PROTECTED_RUNTIME_SECRET_SOURCE_MISSING
BLOCKED_API_USERSECRETS_REMOVAL
BLOCKED_API_FULL_RUNTIME_STARTUP
BLOCKED_API_FULL_RUNTIME_ENDPOINT_ALIGNMENT
BLOCKED_API_CANONICAL_DB_PROOF
BLOCKED_WPF_RETAINED_CREDENTIAL_ABSENT
BLOCKED_WPF_TOKEN_PERSISTENCE
BLOCKED_WPF_TOKEN_REFRESH_RESUME
BLOCKED_WPF_PRODUCTROOT_TOKEN_STATE_MISMATCH
BLOCKED_WPF_MAINWINDOW_PHYSICAL_PROOF
```

Every blocker must name the exact class/script/method/config key, sanitized exception, source availability, process/write state, and whether token state was preserved. Do not return a generic auth, user-secrets, API-startup, or WPF-startup blocker.
