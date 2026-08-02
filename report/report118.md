# Prompt118 Coordination Report

## Verdict

BLOCKED_WPF_REFRESH_RESUME.

The ApiServer Visual Studio startup failure is fixed: full API no longer depends on .NET User Secrets, starts from the protected machine-local runtime source, and aligns with WPF on the canonical full API endpoint. WPF still cannot open MainWindow because the retained bootstrap credential exists and decrypts, but the full API rejects it at protected hello with `WPF_HELLO_HTTP_401`; the current PlatformAppV0 installation contract has no refresh-token field/path.

## API Diagnosis And Config

- DB-credential-vs-WpfJwt diagnosis confirmed: yes
- Operator ApiServer screenshot reproduced: yes
- Exact ApiServer startup guard class/method/line: `Program.<Main>$`, `Program.cs:103` before fix
- ApiServer UserSecretsId before/after: present / removed
- Runtime AddUserSecrets/read path count before/after: `2 / 0`
- ApiServer User Secrets DB key count before/after: `ConnectionStrings:PostgreSqlConnection` absent before / no active user-secret DB read after
- ApiServer active user-secret key reads after task: `0`
- Canonical protected runtime source name only: `OBM Platform env.production`
- Full API required secret keys classified: yes
- Visual Studio full API noninteractive startup: yes
- start-api-local full API noninteractive startup: yes
- Visual Studio/script same protected source: yes
- FINAL_SHA_RETURNED_BY_CODEX guard triggered after fix: no

## Endpoint And API Proof

- Canonical full API base URL sanitized: `http://127.0.0.1:7161`
- Full API Visual Studio URL: `http://127.0.0.1:7161`
- Full API start script URL: `http://127.0.0.1:7161`
- WPF normal API base URL matches: yes
- Phase1-only profile used by normal WPF: no
- Canonical API DB runtime proof: yes, `obm_api_dev_v0_pg`
- API health/readiness: yes, `/health`, `/health/ready`, and `/api/platform-v0/readiness` returned `200`
- API pending migrations count: `0` from accepted `MainApiDevResetExecutionV001` proof
- Full grouped API routes loaded: yes, full host started
- Platform/bootstrap routes required by WPF loaded: yes

## WPF Token State

- Redeem access/bootstrap token returned: yes by current source contract (`WpfJwt`)
- Redeem refresh token returned: no / not-supported by current PlatformAppV0 contract
- Durable token persistence proven: yes
- ProductRoot used at redeem: `E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot`
- ProductRoot used at current startup: `E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot`
- Same ProductRoot: yes
- Access-token protected record exists: yes
- Refresh-token protected record exists: not-supported
- Checkpoint exists: yes
- DPAPI access-token read result: success
- DPAPI refresh-token read result: not-supported
- Access token expired/rejected: yes, full API protected hello returned `WPF_HELLO_HTTP_401`
- Refresh token usable: not-supported
- Refresh path invoked: no
- Phase1 resume result: failed at `ProtectedHello`
- WPF token-state classification: `T7_NO_REFRESH_TOKEN_CONTRACT_EXISTS_IN_CURRENT_SOURCE`
- Primary classification T1-T9: `T7`
- Proven credential clear/delete call reached: no
- ApiServer/API failure deletes credential state after fix: no
- Redeem repeated during task: no
- New redeem required: yes, because the retained bootstrap WpfJwt is rejected and no refresh-token contract exists

## WPF Physical Startup

- WPF reused retained credential without new redeem: yes
- WPF ProductRoot match proof: yes
- MainWindow opens directly: no
- InstallationV0 shown on installed-local launch: yes
- Physical WPF title observed: `OBM InstallationV0 Phase 1/2 - prompt118`
- Process crash/exit after fix: no
- 60-second MainWindow stability: no, MainWindow did not open
- Second launch without redeem: no, not applicable until MainWindow opens
- InstallationV0 shown on valid installed-local launch: not proven valid; current retained credential cannot resume

## Build / Test Counts

- ApiServer build: errors `0`, warnings `60`
- ApiServer focused runtime probes: `/health` `200`, `/health/ready` `200`, `/api/platform-v0/readiness` `200`
- WPF build: errors `0`, warnings `0` from earlier prompt118 proof
- WPF resume-only runner: Phase1 failed at `ProtectedHello`, no Phase2 execution

## Scope Locks

- Production files changed count and paths: `5` API files: `Program.cs`, `PlatformAppV0Module.cs`, `ApiServer01.csproj`, `start-api-local.ps1`, `Properties/launchSettings.json`
- API DB reset/migration/schema changed: no
- WPF DB reset performed: no
- TblTenantPosDevice changed: no
- Sync/Provider behavior changed: no
- Category Weight changed: no
- Booking Weight changed: no
- Price Weight save semantics changed: no
- Redeem/token values exposed: no
- Passwords/connection strings exposed: no

## Operator Status

- Operator MainWindow screenshot ready: false
- Manual POS1 test ready: false
- Required operator-safe next step: use the existing authorized recovery path to obtain a fresh bootstrap credential, or authorize a future task to implement a real refresh-token contract for PlatformAppV0 installation credentials. Do not hand-write completion markers or fabricate token state.

## Private Artifact

- Private artifact version: `WpfCanonicalInstalledLocalStartupV001`
- Private artifact aggregate SHA-256: `fa9ec103ee8f733f99fed9e8aedb301fd775b5c7d08c0bab9d5aa95379ee826c`

## Coordination Commit

FINAL_SHA_RETURNED_BY_CODEX
