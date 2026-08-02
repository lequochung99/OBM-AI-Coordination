# Prompt 118 Addendum — Start from the operator ApiServer failure, remove API UserSecrets dependency, then prove WPF token persistence/resume

This addendum is binding and must be read together with:

```text
prompt/prompt118.md
prompt/prompt118_MINIMAL_WPF_STARTUP_RECOVERY_ADDENDUM.md
```

Where this addendum conflicts with a broader interpretation, this addendum controls.

## Authoritative operator evidence

The operator has provided a physical Visual Studio screenshot from the actual ApiServer debug launch. `Program.cs` throws:

```text
System.InvalidOperationException:
Local API is using placeholder DB credentials.
Start with start-api-local.ps1 or set user-secrets / environment variables.
```

The exception is raised by the Development placeholder-credential guard before normal ApiServer startup completes.

The operator has made these authoritative decisions:

```text
Do not use .NET User Secrets for ApiServer runtime configuration.
Start investigation from this ApiServer startup failure.
WPF Pairing Code redeem previously returned an access token and a refresh token.
Before redesigning InstallationV0, prove whether those protected WPF credentials still exist, were deleted, cannot be decrypted, are being read from the wrong ProductRoot, or are merely unusable because ApiServer is not running.
```

Do not assume token deletion solely because WPF cannot enter the application. The screenshot directly proves an ApiServer startup/configuration failure that can make bootstrap/resume calls unreachable. Close that boundary first, then audit the protected WPF token state.

## Hard priority order

Execute strictly in this order:

```text
1. Reproduce the exact operator-equivalent ApiServer Visual Studio launch.
2. Remove ApiServer runtime dependency on .NET User Secrets.
3. Reuse one existing protected machine-local PostgreSQL runtime configuration source.
4. Prove ApiServer starts non-interactively on loopback against obm_api_dev_v0_pg.
5. Prove health/readiness and pending migrations = 0 through the already accepted boundaries.
6. Audit the exact WPF access-token/refresh-token protected persistence and deletion lifecycle.
7. Launch WPF against the now-running ApiServer and prove whether token resume works.
8. Apply only the smallest directly proven startup/token-persistence correction.
9. Physically prove normal WPF startup reaches MainWindow when local installation state is complete.
```

Do not begin Category Weight, Booking Weight, TblTenantPosDevice, routing, sync E2E, CompanionApp, terminal modeling, or any other architecture task.

## Separate the credential boundaries

Do not conflate these four independent concepts:

```text
A. ApiServer PostgreSQL runtime credential
B. ApiServer migration/provisioning/admin credential
C. WPF installation/bootstrap access token and refresh token
D. WPF canonical Provider runtime API credential/session
```

This task may repair A and audit/fix C only as required for startup. It must not redesign B or D.

The Pairing Code/WpfJwt flow remains the installation authorization boundary. The canonical Provider remains the owner of normal WPF-to-API request authentication/header behavior. Do not move token/header logic into feature code or the periodic worker.

## Frozen work

Do not inspect, design, implement, or modify:

```text
Service Category Weight
Booking Weight
Price Weight save/business semantics
TblTenantPosDevice entity/writer/schema/migration
TblPosLocal/TblTenantPosDevice routing
API destination routing
POS1-POS10 Platform UI or Pairing Code UI/API behavior
canonical Provider request-auth behavior
API grouped-sync happy path
CompanionApp registration model
payment-terminal model
API database reset/drop/recreate
new migration generation
PostgreSQL role redesign already accepted by prompt114
```

Do not create a second configuration framework, second secret store, second startup script, second ProductRoot, second token store, or second authentication scheme.

## Phase 1 — Reproduce and classify the exact ApiServer operator launch

Use the exact ApiServer startup project/profile the operator starts from Visual Studio.

Capture direct evidence:

```text
STARTUP_PROJECT=<exact project>
STARTUP_PROFILE=<exact profile>
ENVIRONMENT=<exact environment>
PROGRAM_CS_GUARD=<exact class/method/line>
CONNECTION_CONFIGURATION_KEY_NAMES=<names only>
CONFIGURATION_PRECEDENCE=<ordered source names only>
DATABASE_NAME_RESOLVED=<safe name>
HOST_CLASSIFICATION=<loopback/approved local>
PLACEHOLDER_DETECTED=yes/no
USER_SECRETS_PROVIDER_REGISTERED=yes/no
USER_SECRETS_KEY_NAMES_READ=<names only>
EXACT_EXCEPTION_TYPE
EXACT_SANITIZED_EXCEPTION_MESSAGE
PROCESS_EXIT_CODE
```

Prove whether the operator launch differs from `start-api-local.ps1`, and exactly why the script can resolve credentials while Visual Studio launch cannot.

Do not ask the operator to paste a password, token, connection string, or secret value.

## Phase 2 — Remove ApiServer UserSecrets dependency

The operator does not want .NET User Secrets used for ApiServer runtime configuration.

Audit and remove from the ApiServer project when present and runtime-reachable:

```text
<UserSecretsId>
AddUserSecrets(...)
configuration reads whose only source is User Secrets
startup messages instructing the operator to set User Secrets
focused tests/docs that require User Secrets for ApiServer DB startup
obsolete User Secrets DB override keys
```

Requirements:

```text
ApiServer normal Development/LocalDevelopment startup does not read User Secrets
ApiServer can start from Visual Studio without running dotnet user-secrets
no password or complete connection string is committed to appsettings, launchSettings, source, scripts, reports, or artifacts
placeholder guards remain fail-closed but reference only the canonical protected source/setup boundary
obsolete noncanonical DB overrides remain absent
```

This task is scoped to ApiServer User Secrets. Do not delete active unrelated PlatformAppV0 external-login configuration merely because another project also has a `UserSecretsId`.

Report key names only; never report values.

## Phase 3 — Reuse the existing protected machine-local runtime DB source

Reuse the already established protected local PostgreSQL runtime configuration/import boundary used by the accepted local ApiServer startup work. Do not create a new secret framework.

Allowed existing mechanisms include only what current source/artifacts prove, such as:

```text
existing central protected environment import
existing protected local configuration file/DPAPI boundary
existing PGPASSFILE/passfile integration
existing machine-local environment boundary
```

Requirements:

```text
normal Visual Studio ApiServer launch and start-api-local.ps1 resolve the same canonical runtime DB identity
runtime DB = obm_api_dev_v0_pg
provider = Npgsql/PostgreSQL
host = loopback or approved local Development
startup is non-interactive
no hidden password prompt
missing protected source fails immediately with a named sanitized error
normal ApiServer process receives only runtime DB credential, not admin/provisioning credential
no User Secrets fallback
```

Do not use `OBM_PLATFORM_V2_P6_POS_PG_ADMIN` as the normal runtime credential unless complete existing source proves that is already the intended runtime contract. The accepted role separation must remain intact.

If the existing protected runtime source is genuinely absent, stop with:

```text
BLOCKED_API_PROTECTED_RUNTIME_CONFIG_ABSENT
```

and report only the exact source/key name and setup boundary required.

## Phase 4 — Physical ApiServer acceptance

From the exact operator-equivalent Visual Studio launch, prove:

```text
User Secrets dependency after task = no
placeholder exception = no
ApiServer starts non-interactively
loopback binding only
canonical DB = obm_api_dev_v0_pg
provider = Npgsql
health/readiness succeeds
pending migrations = 0 through the already accepted migration/provisioning proof boundary
no secret value logged
```

Do not continue to WPF token conclusions until this physical API proof passes.

## Phase 5 — Audit the exact WPF access/refresh token persistence

After ApiServer startup is stable, audit the actual Pairing Code redeem response and WPF protected persistence implementation.

Do not invent a refresh-token contract if current source does not contain one. Prove the actual DTO/property/file/key names and behavior.

Inspect completely:

```text
Pairing Code redeem response DTO
access token property
refresh token property when present
credential expiry metadata
protected machine-state/checkpoint writer
protected machine-state/checkpoint reader
DPAPI/protection scope
resolved ProductRoot
all credential file names and paths
all Save/Replace/Rotate/Clear/Delete methods
all startup/resume call sites
all exception paths that clear or invalidate credentials
all ProductRoot/profile precedence
```

Capture only safe markers:

```text
ACCESS_TOKEN_RETURNED_BY_REDEEM=yes/no
REFRESH_TOKEN_RETURNED_BY_REDEEM=yes/no/not-supported
ACCESS_TOKEN_STATE_FILE_PRESENT=yes/no
REFRESH_TOKEN_STATE_PRESENT=yes/no/not-supported
STATE_DECRYPTABLE=yes/no
STATE_PRODUCT_ROOT=<exact path>
CURRENT_LAUNCH_PRODUCT_ROOT=<exact path>
PRODUCT_ROOT_MATCH=yes/no
CREDENTIAL_EXPIRY_METADATA_PRESENT=yes/no
ACCESS_TOKEN_EXPIRED=yes/no/unknown
REFRESH_TOKEN_USABLE=yes/no/not-supported/unknown
EXACT_CLEAR_DELETE_METHODS=<class/method list>
CLEAR_DELETE_CALL_REACHED_DURING_FAILED_STARTUP=yes/no
LAST_WRITE_TIMESTAMP=<safe timestamp>
FILE_OR_STATE_DISAPPEARED=yes/no
```

Never output token values, hashes derived from token values, refresh-token values, authorization headers, or protected file contents.

## Phase 6 — Determine the exact token/startup classification

Select exactly one primary classification from direct evidence:

```text
T1_TOKEN_STATE_PRESENT_AND_VALID_API_WAS_ONLY_OFFLINE
T2_ACCESS_TOKEN_EXPIRED_REFRESH_TOKEN_PRESENT_AND_EXISTING_REFRESH_PATH_FAILED
T3_TOKEN_STATE_DELETED_BY_PROVEN_CLEAR_DELETE_CALL
T4_TOKEN_STATE_EXISTS_BUT_CURRENT_LAUNCH_USES_WRONG_PRODUCTROOT
T5_TOKEN_STATE_PRESENT_BUT_DPAPI_DECRYPTION_OR_CONTRACT_VALIDATION_FAILS
T6_REDEEM_DID_NOT_PERSIST_THE_ACTUAL_RETURNED_TOKEN_CONTRACT
T7_NO_REFRESH_TOKEN_CONTRACT_EXISTS_IN_CURRENT_SOURCE
T8_OTHER_EXACTLY_PROVEN_TOKEN_STATE_DEFECT
```

Do not return a generic “token missing” conclusion.

If T1, do not rotate/redeem or rewrite token storage. Resume the existing credential after ApiServer starts.

If T2, reuse the existing refresh/resume helper. Do not add a new auth service or endpoint.

If T3, remove only the incorrect clear/delete behavior and add regression tests proving API/bootstrap/network failure does not delete protected credentials.

If T4, repair only the existing normal profile/ProductRoot selection. Do not copy protected token files between roots.

If T5 or T6, repair only the exact existing protected-state serialization/protection/contract boundary.

If T7, report the actual contract accurately and do not fabricate refresh tokens.

## Phase 7 — WPF physical startup acceptance

Use the exact operator-equivalent normal WPF Visual Studio launch with visible label:

```text
prompt118
```

Required proof when local installation state is complete:

```text
same canonical ProductRoot that owns the persisted credential state
canonical WPF DB = obm_pos_dev_v0_pg
provider = Npgsql/PostgreSQL
ApiServer health is available
existing protected credential loads/resumes without a new Pairing Code redeem
MainWindow opens directly
InstallationV0 does not replace MainWindow
process remains alive and responsive for at least 60 seconds
close normally and launch a second time
second launch again reaches MainWindow without re-redeem
```

If local Phase2 installation is genuinely incomplete, do not fabricate completion. Keep InstallationV0 alive and precise, but prove the access/refresh token state is retained and not deleted by failed startup/API calls.

API/network failure must never delete access/refresh protected state automatically unless the server has returned a specifically proven terminal revocation/invalid-contract result and the existing contract requires removal.

## Minimal source-change rule

Production changes are limited to the directly proven boundaries:

```text
ApiServer configuration registration/startup guard/profile integration
existing protected runtime DB configuration resolver
WPF protected credential persistence/load/clear logic
WPF normal ProductRoot selection when directly proven wrong
exact startup/resume exception owner
```

Do not modify unrelated installation, sync, routing, Weight, POS-device, CompanionApp, or terminal code.

## Required tests

Run focused tests for:

```text
ApiServer normal Visual Studio profile does not register/read User Secrets
ApiServer resolves canonical protected runtime DB source
placeholder config fails fast without hidden prompt
operator-equivalent ApiServer startup succeeds
access/refresh token redeem contract is persisted exactly as returned
failed API/bootstrap call does not delete credential state
wrong ProductRoot cannot silently appear as missing token
existing refresh/resume path behavior when supported
WPF restart loads retained protected credential
normal installed-local startup reaches MainWindow without re-redeem
prompt117 HttpRequestException regression
```

All focused tests must pass with 0 skipped. Build ApiServer and WPF with 0 errors.

Physical startup behavior overrides build/test results.

## Hard acceptance locks

Until PASS:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=false
MANUAL_POS1_TEST_READY=false
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
```

PASS may set:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=true
MANUAL_POS1_TEST_READY=false
```

PASS verdict:

```text
OBM_API_USERSECRETS_REMOVED_WPF_TOKEN_RESUME_MAINWINDOW_PHYSICALLY_RESTORED_READY_FOR_OPERATOR_SCREENSHOT
```

Narrow blockers only:

```text
BLOCKED_API_OPERATOR_EQUIVALENT_STARTUP
BLOCKED_API_PROTECTED_RUNTIME_CONFIG_ABSENT
BLOCKED_API_USERSECRETS_REMOVAL
BLOCKED_API_CANONICAL_DB_RUNTIME_PROOF
BLOCKED_WPF_TOKEN_STATE_AUDIT
BLOCKED_WPF_TOKEN_STATE_DELETED
BLOCKED_WPF_TOKEN_DPAPI_OR_CONTRACT
BLOCKED_WPF_PRODUCTROOT_TOKEN_STATE_MISMATCH
BLOCKED_WPF_REFRESH_RESUME
BLOCKED_WPF_MAINWINDOW_PHYSICAL_PROOF
BLOCKED_WPF_PHASE2_EXISTING_COMPLETION_BOUNDARY
```

Do not return another generic startup, installation, credential, or token blocker.

## Required private artifact additions

Preserve all earlier artifacts. Add to the prompt118 versioned artifact:

```text
OPERATOR_API_SCREENSHOT_INTAKE.md
API_OPERATOR_EQUIVALENT_LAUNCH.md
API_USERSECRETS_INVENTORY.md
API_USERSECRETS_REMOVAL_PROOF.md
API_PROTECTED_RUNTIME_CONFIG_PROOF.md
API_VISUAL_STUDIO_STARTUP_PROOF.md
WPF_REDEEM_TOKEN_CONTRACT.md
WPF_PROTECTED_TOKEN_STATE_INVENTORY.md
WPF_TOKEN_CLEAR_DELETE_CALL_CHAIN.md
WPF_TOKEN_STATE_CLASSIFICATION.md
WPF_TOKEN_RESUME_PROOF.md
MAINWINDOW_WITH_RETAINED_TOKEN_PROOF.md
SECOND_LAUNCH_WITHOUT_REDEEM_PROOF.md
```

## Public report additions

`report/report118.md` must include:

```text
Verdict
Operator ApiServer screenshot reproduced yes/no
Exact ApiServer startup guard class/method/line
ApiServer UserSecretsId before/after
Runtime AddUserSecrets/read path count before/after
ApiServer User Secrets DB key count before/after (names only, no values)
Protected runtime DB source name only
Operator-equivalent ApiServer startup yes/no
Canonical API DB runtime proof yes/no
ApiServer health/readiness yes/no
API pending migrations count
Access token returned by redeem yes/no
Refresh token returned by redeem yes/no/not-supported
Access token protected state present yes/no
Refresh token protected state present yes/no/not-supported
Protected token state decryptable yes/no
Token-state ProductRoot matches current launch yes/no
Exact token classification T1-T8
Proven credential clear/delete call reached yes/no
ApiServer/API failure deletes credential state after fix yes/no
WPF reused retained credential without new redeem yes/no
MainWindow opens directly yes/no
InstallationV0 shown on installed-local launch yes/no
60-second MainWindow stability yes/no
Second launch without redeem yes/no
Production files changed count and paths
ApiServer build/test totals
WPF build/test totals
API DB reset/migration/schema changed no
TblTenantPosDevice changed no
Sync/Provider behavior changed no
Category Weight changed no
Booking Weight changed no
Operator MainWindow screenshot ready true/false
Manual POS1 test ready false
Private artifact and aggregate SHA-256
```
