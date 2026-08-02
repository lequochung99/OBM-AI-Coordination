# Prompt 118 Addendum — `.env.local` / `.env.production` are the canonical ApiServer secret sources

This addendum is binding and must be read together with all current `prompt118` files.

## Authoritative operator clarification

The ApiServer repository already contains a committed placeholder template:

```text
.env.example
```

The real secret-bearing environment files are intentionally separate:

```text
.env.local       — local Development/LocalDevelopment runtime secrets
.env.production  — Production runtime secrets
```

Real passwords, signing keys, OAuth client secrets, service-account JSON, license keys, and other protected values remain only in the appropriate uncommitted environment file or deployment-protected equivalent.

The canonical ApiServer configuration direction is therefore:

```text
.env.example = committed placeholder/key-name contract only
.env.local = local full-ApiServer protected runtime source
.env.production = production protected runtime source
.NET User Secrets = not the canonical ApiServer runtime source
```

Do not solve the current ApiServer startup failure by adding `ConnectionStrings:PostgreSqlConnection` back to .NET User Secrets.

## Important distinction

The environment-file source is for ApiServer infrastructure/application secrets such as:

```text
PostgreSQL runtime connection credential
JWT/signing keys
Google/platform authentication configuration
Firebase Admin/service-account configuration when still active
Syncfusion license
SignalR server configuration
other proven server-owned secrets
```

It is not the WPF access token, WPF refresh token, Pairing Code, WpfJwt protected state, or canonical WPF Provider session store.

## Existing `.env.example` problems that must be audited

The current template contains a mixture of active, stale, and retired keys. At minimum audit each of these by complete current call-site evidence:

```text
DatabaseProvider
ConnectionStrings__DefaultConnection
ConnectionStrings__MyConnectionString
ConnectionStrings__PostgreSqlConnection
PLATFORM_JWT_SIGNING_KEY
PlatformJwt__Issuer
PlatformJwt__Audience
Authentication__Google__Authority
Authentication__Google__ClientId
JwtSettings__Secret
JwtSettings__Issuer
JwtSettings__Audience
Jwt__SecretKey
PosStationToken__SecretKey
PosStationToken__Issuer
PosStationToken__Audience
FirebaseJson
FirebaseJsonPath
AppConfigs__FIREBASE_API_KEY
AppConfigs__FirebaseProjectId
FirebaseLoginSettings__Email
FirebaseLoginSettings__Password
AppConfigs__Syncfusionlicense
AppConfigs__SignalRBaseUrl
OutboundSignalR__HubBaseUrl
OutboundSignalR__HubBearerToken
EmpApp__ReturnOtpInResponse
EmpApp__EnableDevConfirmFirebaseEndpoint
```

Do not print their values.

The template currently also contains misleading/stale guidance and values, including:

```text
comments preferring .NET User Secrets
PostgreSQL example DB name recovery_api_day16_pg instead of the canonical main Development target
legacy Firebase email/password keys already retired for the WPF login path
SQL Server-style DefaultConnection/MyConnectionString placeholders in a PostgreSQL-selected runtime
Development-only EmpApp flags that must not silently become Production defaults
```

Do not remove or retain any key based on its name alone. Classify every key from actual active readers and runtime reachability.

Allowed classifications:

```text
ACTIVE_API_LOCAL_AND_PRODUCTION
ACTIVE_API_LOCAL_ONLY
ACTIVE_API_PRODUCTION_ONLY
ACTIVE_PLATFORM_ADMIN_AUTH
ACTIVE_FIREBASE_ADMIN_SUPPORT
ACTIVE_SIGNALR_OR_LICENSE_CONFIG
LEGACY_WPF_FIREBASE_LOGIN_REMOVE
LEGACY_SQLSERVER_CONFIG_REMOVE_OR_DOCUMENT
OBSOLETE_NONCANONICAL_DB_VALUE_CORRECT
DEVELOPMENT_ONLY_FLAG
UNKNOWN_BLOCK_AND_PROVE
```

## Canonical environment-file loading contract

Audit the current source/scripts and identify the one existing `.env` loader/import boundary.

Required final behavior:

```text
LocalDevelopment/Development full ApiServer
-> load .env.local or the existing protected equivalent

Production ApiServer
-> load .env.production or deployment-injected equivalent

.env.example
-> never loaded as runtime secrets
```

Both of these full-ApiServer launch methods must use the same effective source and resolve the same configuration:

```text
Visual Studio Start/Debug
start-api-local.ps1
```

Do not create two independent secret/configuration pipelines.

Environment isolation requirements:

```text
Development must never load .env.production
Production must never load .env.local
Phase1-only profile must not masquerade as the normal full-ApiServer profile
normal WPF runtime must point to the canonical full-ApiServer endpoint
```

Configuration precedence must be documented and deterministic. Preserve explicit host/deployment environment variables as the highest-priority override when that is the existing deployment contract; the environment-file loader must not unexpectedly overwrite already supplied protected process variables.

## Secret-file safety requirements

Prove:

```text
.env.local is gitignored and not tracked
.env.production is gitignored and not tracked
.env.example contains placeholders only
no secret value appears in source, launchSettings, appsettings, reports, logs, or Git history introduced by this task
normal runtime logs key/source names and safe target metadata only
```

Apply least-privilege local filesystem access to the real environment files using the existing machine/deployment model. Do not copy `.env.production` into WPF or PlatformAppV0 runtime folders.

Do not commit or upload either real environment file to the coordination repository.

## Required `.env.example` correction

After the call-site audit, update `.env.example` so it accurately documents the current canonical contract without real values.

At minimum:

```text
remove guidance that tells operators to prefer .NET User Secrets for ApiServer runtime
use canonical PostgreSQL placeholder metadata for the main Development lane
never include a real username
keep only proven active keys or clearly mark legacy/deprecated keys for removal
remove retired FirebaseLoginSettings email/password keys when no active non-WPF contract remains
mark Firebase Admin/service-account variables separately from retired Firebase client login
mark Development-only flags explicitly and make Production-safe defaults clear
keep placeholder-only values
```

For the canonical local API database example, the safe placeholder should reflect:

```text
Database=obm_api_dev_v0_pg
Username=YOUR_DB_USER
Password=YOUR_DB_PASSWORD
```

Do not embed the actual local role name or password.

## Full ApiServer startup proof

Before inspecting WPF token persistence, physically prove the full API twice:

```text
A. Visual Studio full-ApiServer profile
B. start-api-local.ps1
```

For each prove:

```text
non-interactive startup
correct Development environment
PostgreSQL/Npgsql provider
canonical DB = obm_api_dev_v0_pg
same runtime role classification
same canonical full-API loopback URL/port
pending migrations = 0
health/readiness succeeds
full runtime routes loaded
bootstrap routes loaded
no placeholder-credential exception
no .NET User Secrets dependency
no secret values logged
```

If the two launch methods currently use different ports or profiles, reconcile them to one canonical full-ApiServer endpoint. Keep a Phase1-only profile only as an explicit test-only lane; normal installed WPF must not use it.

## WPF retained-token investigation follows API proof

Only after full ApiServer startup succeeds, inspect the WPF protected token state without redeeming again.

Record only safe metadata:

```text
access token present yes/no
refresh token present yes/no
protected state/checkpoint file exists yes/no
DPAPI decrypt succeeds yes/no
saved ProductRoot
current ProductRoot
credential expiry metadata
last-save timestamp
exact clear/delete/rotation call sites
```

Classify exactly one:

```text
T1 retained token is valid; prior API outage caused false installation routing
T2 access token expired; refresh token remains but existing refresh/resume path fails
T3 token state was deleted by an exact clear/delete call
T4 token state exists under a different ProductRoot
T5 DPAPI or contract validation fails
T6 redeem succeeded but persistence did not complete
T7 current source has no refresh-token contract
T8 another exactly proven defect
```

Do not redeem a new Pairing Code before this evidence is captured unless direct proof shows no recoverable retained state exists and the existing installation contract requires operator re-pairing.

Network/API failure must never automatically clear protected WPF credential state or downgrade a completed local installation.

## Physical MainWindow acceptance

After correcting only the proven API configuration and WPF retained-state/startup defect, prove:

```text
full API starts from Visual Studio using .env.local, not User Secrets
WPF uses the canonical ProductRoot and canonical local DB
retained protected token/resume path is used when available
no unnecessary Pairing Code redeem
MainWindow opens directly for valid installed-local state
InstallationV0 does not replace MainWindow
MainWindow remains alive and responsive for at least 60 seconds
close and launch again
second launch again reaches MainWindow
```

## Frozen work

Do not modify or resume in this task:

```text
Category Weight
Booking Weight
Price Weight business/save behavior
TblTenantPosDevice
TblPosLocal/device routing
CompanionApp/terminal modeling
sync E2E
API schema/migrations/reset/role redesign
new authentication provider or token service
```

## Required evidence additions

Add to the prompt118 private artifact:

```text
ENV_FILE_INVENTORY.md
ENV_KEY_CALLSITE_CLASSIFICATION.md
ENV_LOADING_BEFORE.md
ENV_LOADING_AFTER.md
ENV_PRECEDENCE.md
ENV_GITIGNORE_AND_ACL_PROOF.md
ENV_EXAMPLE_BEFORE.md
ENV_EXAMPLE_AFTER.md
VISUAL_STUDIO_FULL_API_PROOF.md
START_API_LOCAL_FULL_API_PROOF.md
FULL_API_ENDPOINT_ALIGNMENT.md
NO_USERSECRETS_DEPENDENCY_PROOF.md
WPF_RETAINED_TOKEN_STATE_PROOF.md
WPF_TOKEN_CLEAR_DELETE_CALL_CHAIN.md
```

## Public report additions

`report/report118.md` must include:

```text
.env.example audited yes/no
.env.local canonical local source yes/no
.env.production canonical production source yes/no
.env.local tracked by Git yes/no
.env.production tracked by Git yes/no
ApiServer .NET User Secrets dependency after task yes/no
Visual Studio full API startup yes/no
start-api-local full API startup yes/no
same canonical DB/port/profile yes/no
canonical DB name
API pending migrations count
placeholder guard triggered after task yes/no
retired Firebase email/password env keys remaining count
active Firebase Admin/support env keys preserved yes/no/not-applicable
WPF access token present yes/no
WPF refresh token present yes/no/not-proven
WPF protected state decryptable yes/no
WPF token-state classification T1-T8
new Pairing Code redeem used yes/no
MainWindow opens directly yes/no
second-launch MainWindow proof yes/no
Category Weight changed no
Booking Weight changed no
```

## PASS verdict

```text
OBM_ENV_LOCAL_PRODUCTION_CANONICAL_FULL_API_AND_WPF_RETAINED_RESUME_MAINWINDOW_READY_FOR_OPERATOR_SCREENSHOT
```

Do not return PASS from API startup alone; MainWindow physical resume proof is required.
