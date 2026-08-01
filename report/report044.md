# Report 044 - Runtime Connectivity and Operational PIN Model Audit

## 1. Verdict

OBM_POS_RUNTIME_CONNECTIVITY_AND_PIN_MODEL_AUDIT_COMPLETE

This was a read-only audit and documentation rewrite. No OBM source, PostgreSQL data, roles, passwords, employee PINs, tokens, environment variables, services, or WPF processes were changed.

## 2. Executive Conclusion

The current source is over-engineered for ordinary OBM-POS runtime startup.

Evidence:

- Local PostgreSQL runtime connectivity is already standard Npgsql connectivity built from host, port, database, username, and a DPAPI-read database password: `ApplicationStartupCoordinator` and `RuntimeProfileStartupAssessmentService`.
- MainWindow can be shown before API worker/sync startup. API token absence defers sync/SignalR rather than blocking already-opened local UI in `App.xaml.cs`.
- Employee `LoginNumber` is a local operational PIN/identifier field. It is not a database credential, API credential, tenant proof, device proof, or Platform authorization credential.
- The current V0 lane additionally requires InstallationV0 Phase 1 checkpoint, Phase 2 v002 marker, identity-spine proof, employee count, permission count, and outbox evidence before `InstalledHealthy`. These are installation completion proofs, not PostgreSQL authentication.
- Recent prompts 041-043 added ProductRoot, effective root, launch provenance, and development database guard predicates to fix debug handoff. They are useful development/diagnostic safeguards but should not become production runtime authentication.

Conclusion: keep local DB readiness and runtime identity consistency, but scope ProductRoot/provenance/Phase proof gates to installation diagnostics and development debug profiles. Normal installed runtime should be local-DB-first and API-offline tolerant.

## 3. Canonical Three-Domain Distinction

Domain 1 - WPF to local PostgreSQL:

- Genuine infrastructure credential.
- Uses host, port, database, username, and password.
- The password belongs to PostgreSQL role authentication, not employee login.

Domain 2 - WPF to API:

- Should use service credentials: short-lived access token plus long-lived refresh token.
- Current source has Phase1 `WpfJwt` bootstrap and station/app access-token shapes, but no complete access-token plus refresh-token runtime contract.
- Tokens must be protected on the machine and must not be stored in PostgreSQL business tables.

Domain 3 - employee operational PIN:

- Local POS convenience/control mechanism.
- Used for role-sensitive UI gates, Staff/non-Staff routing, employee present/session behavior, and actor attribution.
- It must not be described as database auth, API auth, device auth, tenant proof, or installation proof.

## 4. Current Employee PIN Physical Implementation and Purpose

There is no evidence that ordinary OBM-POS employee access is a full application-user account/password system.

Observed implementation:

- `TblEmployee.LoginNumber` is a required string column in WPF model and EF mapping.
- `LoginEmpPassword.xaml.cs` documents "Employee PIN Login" and treats `LoginNumber` as an in-memory PIN buffer.
- Staff/login route: after at least four digits, WPF looks up a RealEmployee by exact `LoginNumber`, then `LoginModeResolver.EvaluatePinLogin` applies working/deleted/present rules.
- Manager/non-Staff route: `ManagerPassword_Wd` looks for a RealEmployee where `PermissionName != Staff` and `LoginNumber` matches the entered value.
- MainWindow has a similar Staff/non-Staff split for manager account and employee account navigation.
- `CompanyInfo.InChargeGuid` is set after non-Staff manager PIN match, which supports actor attribution for manager/sensitive actions.
- `EmpAppController.Register` also uses `LoginNumber` for employee app registration, but this is EmpApp/mobile registration behavior, not PlatformAppV0 admin management.

Purpose classification:

- Employee login/present marking: Employee PIN.
- Manager area/sensitive action gate: Employee PIN with non-Staff role check.
- Actor attribution: Employee PIN resolves an EmployeeGuid/InChargeGuid.
- Platform administrator login: separate Google/admin/JWT path, not employee PIN.

## 5. Current PIN Lengths Versus Approved 4/6-Digit Policy

Approved product direction:

- Staff PIN: 4 digits.
- Non-Staff PIN: 6 digits.

Current source behavior:

- `LoginEmpPassword.xaml.cs` triggers PIN login when `LoginNumber.Length >= 4`. That is consistent with a 4-digit Staff/employee PIN flow, although the resolver allows any RealEmployee role in that window.
- `MainWindow.xaml.cs` Staff/Employee keypad path triggers at `LoginNumber.Length == 4`.
- `ManagerPassword_Wd.xaml.cs` triggers non-Staff lookup when `LoginNumber.Length > 4`. That means 5 or more digits can submit, not exactly 6.
- `EmployeePassword_Wd.xaml.cs` also triggers when `LoginNumber.Length > 4`, then calls `PosEmployeePinLoginAsync` with Staff role. This is not aligned with a strict Staff 4-digit rule.

Implementation gap:

- Non-Staff is not physically enforced as exactly 6 digits in the audited source.
- Some secondary windows use `>4` rather than exact 4 or exact 6.

Physical V0 DB evidence:

- `TblEmployee.LoginNumber` is non-null with max length 20.
- No unique constraint was found on `TblEmployee.LoginNumber`.
- Current V0 seeded employees have 20-character sanitized placeholder-style LoginNumber values, not operational 4/6-digit PINs.
- No raw PIN values were printed in this report.

## 6. Current PIN Seed / Reset / Edit Behavior

InstallationV0 Phase2 seed behavior:

- `Phase2LegacySeedMethodInventory` marks legacy `SeedEmployeeAsync` as excluded for employee/PIN/private data.
- `PostgreSqlPhase2ReferenceSeedExecutor.EmployeeResetValue` maps `LoginNumber` to a stable `UNCONFIGURED` placeholder shortened to the 20-character column limit.
- The reference-driven seed preserves employee row shape but intentionally avoids copying real reference PIN/private values.
- Current DB evidence shows seeded `LoginNumber` length 20 for all Phase2 v002 employees, matching placeholder/reset behavior.

Seed policy conclusion:

- Copying real reference PINs: not done in current InstallationV0 v002 path.
- Deterministic placeholder/reset values: yes.
- Null/unconfigured values: the column is not nullable; placeholder is used instead.
- User edits after installation: required for meaningful operational PINs.
- Uniqueness requirements: no DB unique constraint found on `LoginNumber`; current physical placeholders are not duplicated, but uniqueness is not schema-enforced.
- UI behavior when PIN is unconfigured: placeholder PINs are unlikely to match intended 4/6-digit keypad behavior; operator setup/edit flow is needed before normal PIN use is meaningful.

## 7. Current Local PostgreSQL Connection / Authentication Path

Runtime source path:

- `ApplicationStartupCoordinator.AssessAsync`
- `RuntimeBootstrapLocator.Locate`
- `RuntimeDatabaseCredentialProvider.LoadPassword`
- `RuntimeProfileStartupAssessmentService.BuildConnectionString`
- `NpgsqlConnection.OpenAsync`

The runtime connection string is built from:

- `DatabaseHost`
- `DatabasePort`
- `DatabaseName`
- `DatabaseUsername`
- DPAPI-unprotected database password

The password is used only as a PostgreSQL infrastructure credential. It is not an employee PIN.

Physical DB evidence:

- Database: `obm_pos_dev_v0_pg`
- Current/session user: `hung`
- Runtime state: `Activated`

## 8. Current Runtime DB Role and Privileges

Read-only privilege check for the current runtime DB user showed CRUD privileges on representative runtime tables:

| Table | SELECT | INSERT | UPDATE | DELETE |
| --- | --- | --- | --- | --- |
| `dbo.TblEmployee` | yes | yes | yes | yes |
| `dbo.TblEmployeePermission` | yes | yes | yes | yes |
| `dbo.TblLocalOutbox` | yes | yes | yes | yes |
| `dbo.TblPosLocal` | yes | yes | yes | yes |
| `dbo.TblPosRuntimeProfile` | yes | yes | yes | yes |
| `dbo.TblTenant` | yes | yes | yes | yes |

Required runtime privilege model:

- `CONNECT` on database.
- `USAGE` on application schema.
- `SELECT`, `INSERT`, `UPDATE`, `DELETE` on required runtime tables.
- Sequence privileges where generated values require them.

No Phase2 source evidence showed runtime GRANT/REVOKE/user creation.

## 9. Current Phase 2 Seed Side Effects

Source search under `InstallationV0\Phase2` found no:

- `CREATE ROLE`
- `CREATE USER`
- `ALTER ROLE`
- `GRANT`
- `REVOKE`
- API access-token writes
- refresh-token writes
- environment variable writes
- WpfJwt writes
- Pairing Code replay

It did find:

- `LoginNumber` reset logic.
- private column hints including `Password` and `LoginNumber`.
- legacy label `Login-With-Password` / `PIN/password login default`.

Conclusion: Phase2 seed changes business/runtime baseline rows and outbox rows. It does not alter PostgreSQL roles, DB credentials, API tokens, environment security, or employee PIN security semantics. It does reset/carry placeholder `LoginNumber` data for seeded employees.

## 10. Current API Credential Contract

PlatformAppV0 Phase1 contract:

- `POST /api/platform-v0/wpf/pairing/redeem` validates a one-time Pairing Code and returns a bearer `WpfJwt`.
- `GET /api/platform-v0/wpf/bootstrap/me` requires `WpfJwt` scheme and `WpfInstallBootstrap` policy.
- `PlatformAppV0TokenService.CreateWpfToken` creates claims for scope, tenant, POS station, POS guid, slot, installation attempt, and local installation guid.
- The token expiry in `Redeem` is currently `now.AddHours(4)`.
- WPF protects the `WpfJwt` with DPAPI CurrentUser in `Phase1InstallationService.ProtectCredential`, reads it back, and persists a checkpoint reference.

Runtime/station token shape:

- `PosStationIdentity` contains `AccessToken`, `TokenType`, and `TokenExpiresAt`.
- `PosStationLocalSettings` persists this station identity in JSON under the ProductRoot path.
- `AppJwtBootstrapper` can use a valid station access token and skip legacy AppJwt bootstrap.

Missing contract:

- No complete WPF runtime access-token plus refresh-token pair was found in the audited PlatformAppV0 Phase1 source.
- No refresh endpoint, refresh-token storage, refresh-token rotation, or refresh invalid/revoked handling was found in the V0 Phase1 contract.

## 11. Access-Token / Refresh-Token Gap Analysis

Target model:

- short-lived access token;
- long-lived refresh token;
- protected local storage;
- refresh rotation/revocation;
- local mode when API unavailable.

Current model:

- Phase1 bootstrap: single scoped `WpfJwt`, temporary bootstrap credential.
- Runtime/station: access token field exists in `PosStationIdentity`.
- Legacy app auth: `AppAuthSession` persists an `AppJwt`, but this is not the canonical access/refresh pair.
- Refresh token: not present in audited WPF station settings, PlatformAppV0 Phase1 endpoints, or WpfJwt redeem response.

Gap:

- V0 can prove bootstrap API identity, but it does not yet implement the final runtime API token lifecycle.

## 12. Current API-Offline Startup / Runtime Behavior

Current startup behavior supports local-first UI in important places:

- `App.xaml.cs` shows `MainWindow` before `IAppJwtBootstrapper.EnsureAppJwtAsync`.
- `AppHost.StartAsync` timeout/failure is logged as degraded and continues UI.
- Missing or expired POS station token defers Sync/SignalR startup after MainWindow, rather than reopening installer.
- SignalR connection failure is logged in a background task.
- LocalGateway comments say station-not-ready must be retryable and must not falsely force re-pairing.

Current caveat:

- `EnsureAppJwtAsync` exceptions are still shown as a "POS API login error" message and return from the post-MainWindow startup continuation. This does not prevent the already displayed MainWindow, but should be simplified into a non-blocking offline status.

Conclusion:

- Local UI can open with API offline after DB readiness passes.
- Cloud/sync workers are gated by token/identity and defer when token is missing/expired.
- The code should formalize this as offline local mode instead of treating API auth bootstrap as a startup error dialog.

## 13. Startup Gate Inventory and A-G Classification

| Gate / mechanism | Category | Current effect |
| --- | --- | --- |
| Runtime bootstrap locator present | A | Required to find local DB config. |
| DPAPI database password readable | A | Required to authenticate to PostgreSQL. |
| Npgsql open succeeds | A | Required for local DB connectivity. |
| `TblPosRuntimeProfile` exists | A | Required installed-state evidence. |
| RuntimeState `Activated` | A/D | Required usable local runtime state; also installation outcome. |
| Schema version minimum | A | Required schema readiness. |
| Station identity matches runtime profile | A/D | Local identity consistency check. |
| Phase1 `ApiAuthorized` checkpoint | D | Installation-only proof currently required for V0 lane. |
| Phase2 v002 marker | D | Installation-only proof currently required for V0 lane. |
| v002 permission/employee/outbox counts | D/F | Good installation verification; over-specific for every normal startup. |
| `SPACEPOS_PRODUCT_ROOT` | D/E | Machine state/config location; development route safety. |
| `SPACEPOS_INSTALLATION_MODULE` | D/E | Diagnostic mode selector. |
| `EffectiveProductRootContext` | E | Development/handoff safety. |
| `LaunchProvenanceContext` | E/F | Fixes diagnostic handoff, but should not be runtime auth. |
| `DevelopmentProfileLaunchPolicy` | E | Valid development safety guard. |
| API station access token | B | Required for sync/API, not local CRUD. |
| WpfJwt bootstrap credential | D/B | Required for Phase1 bootstrap proof, not ordinary local DB CRUD. |
| Employee LoginNumber/PIN | C | Local UI gate and audit attribution, not DB/API auth. |

Answer to the central audit question: yes, the V0 lane currently makes installation-only/development proof state mandatory for ordinary local POS startup. It does not make employee PIN mandatory for DB CRUD, and it does not make API reachability mandatory before MainWindow display.

## 14. ProductRoot / Environment / Provenance Simplification Analysis

Keep:

- One resolved ProductRoot path as the filesystem location for runtime bootstrap, local config, checkpoints, protected DB/API credential references, logs, and machine state.
- Development-only root and database guard to prevent opening production/Royal/legacy databases while debugging.
- InstallationV0 diagnostics ProductRoot verification before handoff.

Scope down:

- `LaunchProvenanceContext` should be limited to InstallationV0 diagnostics handoff and development debug profiles.
- `DevelopmentStartupGuard` should never be production runtime authentication.
- `SPACEPOS_INSTALLATION_MODULE` should only select diagnostics, not become a durable runtime state.

Remove from normal installed runtime:

- Requiring launch-profile provenance.
- Requiring exact Phase2 employee/outbox count checks on every startup.
- Treating ProductRoot provenance as authorization once the runtime bootstrap and local DB are healthy.

Replace with:

- Direct local DB config + DB auth + schema/readiness checks.
- Independent API token state check after MainWindow opens.
- Offline status/retry queue for API outage.

## 15. Canonical Minimal Startup Decision Tree

```text
Start OBM-POS
    |
    +-- local DB config missing / DB unreachable / schema not ready
    |      -> installation/recovery UI with exact local DB reason
    |
    +-- local DB healthy and installed state internally consistent
           -> open MainWindow
           -> initialize employee PIN/role services locally
           -> initialize API client independently
                 |
                 +-- access/refresh succeeds -> Online
                 +-- API unavailable -> Offline Local Mode
                 +-- refresh invalid -> Offline / Reauthorization Required
```

Comparison with physical source:

- The physical source already performs most local DB readiness checks.
- It currently adds V0-specific Phase1/Phase2 proof checks to `InstalledHealthy`.
- It initializes API workers after MainWindow and defers when token is absent/expired.
- It lacks the target refresh-token contract.

## 16. Mechanism Keep / Scope / Remove Table

| Current mechanism | Domain | Current purpose | Keep / Scope / Remove | Replacement |
| --- | --- | --- | --- | --- |
| Npgsql host/port/database/username/password | PostgreSQL | Local DB authentication | Keep | None |
| DPAPI DB password file | PostgreSQL | Protected local DB credential | Keep | None |
| Runtime profile Activated | Installation state | Installed local runtime state | Keep, simplify | Keep state but avoid unrelated proof checks |
| Phase1 checkpoint | Installation state | API bootstrap proof | Scope to installation/resume | Not ordinary local CRUD auth |
| Phase2 marker | Installation state | Seed completion proof | Scope to install/upgrade verification | Schema/readiness + version marker summary |
| v002 employee/outbox exact counts | Installation state | Trial seed verification | Remove from every startup | Use install-time regression tests/readiness audit |
| WpfJwt | API token | Bootstrap authorization proof | Scope to Phase1 | Runtime access/refresh pair |
| Station AccessToken | API token | Sync/API auth | Keep as current interim | Add refresh token lifecycle |
| Missing refresh token | API token | Gap | Implement | Protected refresh token + refresh endpoint |
| LoginNumber | Employee PIN | Local role gate and attribution | Keep terminology, normalize policy | Staff 4 / non-Staff 6 digit validation |
| ManagerPassword_Wd | Employee PIN | Manager/non-Staff UI gate | Keep behavior, rename later | Manager/non-Staff PIN window |
| Login-With-Password label | Employee PIN | Legacy DB login method key | Correct visible docs | Login-With-PIN terminology |
| ProductRoot | Installation state | Machine state root | Keep as location | One resolved runtime root |
| EffectiveProductRootContext | Development safety | Handoff/root validation | Scope to dev/diagnostics | ProductRoot resolver only |
| LaunchProvenanceContext | Redundant runtime gate | Same-process handoff proof | Scope to diagnostics | Do not use as runtime auth |
| DevelopmentProfileLaunchPolicy | Development safety | Prevent unsafe debug DB/root | Keep only dev/debug | No production dependency |

## 17. Canonical OBM-POS Runtime Connectivity and Operational PIN Model

OBM-POS has three separate credential/control domains. They must not be conflated.

### PostgreSQL Infrastructure Credential

OBM-POS connects to the local POS database using a standard PostgreSQL credential:

- host;
- port;
- database;
- username;
- password.

The PostgreSQL username/password authenticates WPF to the local database engine. It is the only credential that authorizes local SQL access. Runtime database privileges should be limited to the CRUD and sequence operations the app actually needs. Installer/admin credentials may be used temporarily for database creation, migration, backup/restore, and GRANT operations, but must not become hidden runtime dependencies.

### API Access and Refresh Tokens

OBM-POS API connectivity must use machine-protected API credentials:

- short-lived access token;
- long-lived refresh token.

The access token authorizes online API/sync operations. The refresh token renews access without asking for a Pairing Code again. Both must be protected locally with the approved Windows protection mechanism and must never be stored in PostgreSQL business/config tables or printed in logs.

If API is unavailable, MainWindow still opens when the local DB is healthy. Sync/cloud functions enter Offline Local Mode and retry later. If refresh is invalid or revoked, local checkout/business data remains usable and cloud functions require reauthorization.

### Employee Operational PIN

Employee `LoginNumber` is an employee operational PIN. It supports:

- Staff/non-Staff UI routing;
- manager-area and sensitive-action gates;
- employee present/session behavior;
- actor attribution and audit/log correlation.

It is not:

- an application account password;
- a PostgreSQL credential;
- an API credential;
- a device credential;
- tenant proof;
- installation proof;
- a substitute for access/refresh tokens.

Approved PIN policy is Staff 4 digits and non-Staff 6 digits. Existing source should be corrected to enforce that policy without creating a new password framework.

### Installation State Versus Runtime Authentication

InstallationV0 Phase1 and Phase2 markers prove that setup completed. They are not runtime credentials. ProductRoot is the filesystem location for machine state, not an authorization credential. Development launch provenance protects developers from unsafe debug targets, but normal production runtime should rely on local DB authentication and local installed-state consistency.

### Seed Boundary

Baseline seed must not:

- create or alter PostgreSQL users/roles;
- GRANT or REVOKE runtime privileges;
- store DB usernames/passwords in business tables;
- store API tokens in PostgreSQL;
- set User/Machine environment variables;
- require API connectivity;
- use employee PIN as database authorization;
- encode launch provenance as database authorization.

### Terminology to Avoid

Avoid "employee password", "manager password", "database PIN", and "WPF password". Use precise domain terms: PostgreSQL password, API access token, API refresh token, employee operational PIN, manager/non-Staff PIN.

## 18. Terminology Correction Table and Prior-Doc Correction List

| Incorrect or ambiguous term | Correct term | Meaning |
| --- | --- | --- |
| employee password | employee operational PIN | local role gate + actor attribution |
| manager password | manager/non-Staff PIN | manager-area/sensitive-action gate + audit |
| database PIN | PostgreSQL username/password | infrastructure authentication |
| WPF password | API access/refresh token or DB credential, depending on context | must be named precisely |
| Login-With-Password | Login-With-PIN, unless preserving DB key name | visible POS login mode label |
| PIN/password login default | employee operational PIN default | local login mode default |
| WpfJwt as runtime credential | bootstrap WpfJwt | Phase1 API authorization proof |

Prior phrases to revise later:

- `InstallationV0\Phase2\Phase2TrialTemplateV001.cs`: `PIN/password login default`.
- `Utilities\MyAppProperties.cs`: `Login-With-Password (PIN via LoginNumber)` should clarify DB key versus visible term.
- `ManagerPassword_Wd` class/window name: rename/document as manager/non-Staff PIN gate.
- Historical reports 007-015 and canonical docs that present `WpfJwt` as the main token should be updated to distinguish bootstrap JWT from runtime access/refresh tokens.
- Reports 040-043 should be followed by a simplification note that ProductRoot/provenance guards are development/diagnostic safety, not production runtime authentication.

## 19. Phased Implementation Plan S1-S6

S1 - Local DB startup independent of API and PIN provenance:

- Files: `ApplicationStartupCoordinator.cs`, `RuntimeProfileStartupAssessmentService.cs`, `App.xaml.cs`.
- Rollback risk: medium; startup routing is sensitive.
- Tests: local DB healthy opens MainWindow with API unavailable; DB missing routes recovery.
- DB schema impact: none.
- API contract impact: none.
- PIN behavior: documentation only.

S2 - Standard access/refresh token contract:

- Files: PlatformAppV0 Phase1/API token services, WPF station/auth session services.
- Rollback risk: medium-high; changes API auth lifecycle.
- Tests: access expiry refreshes; refresh revoked requires reauthorization without blocking local DB.
- DB schema impact: Platform token/session store may need refresh-token hash/revocation storage.
- API contract impact: add refresh endpoint/contract.
- PIN behavior: none.

S3 - Offline local mode and API reconnect:

- Files: `App.xaml.cs`, `AppJwtBootstrapper.cs`, sync/outbox workers, SignalR client services.
- Rollback risk: medium.
- Tests: API DNS/server outage still opens MainWindow; outbox remains pending; reconnect starts workers.
- DB schema impact: none unless adding sync status table.
- API contract impact: none or status endpoint only.
- PIN behavior: none.

S4 - Scope/remove legacy startup provenance guards:

- Files: `EffectiveProductRootContext.cs`, `LaunchProvenanceContext.cs`, `DevelopmentProfileLaunchPolicy.cs`, launch profiles.
- Rollback risk: medium.
- Tests: dev profile still rejects protected DB/root; production runtime ignores launch provenance after DB auth.
- DB schema impact: none.
- API contract impact: none.
- PIN behavior: none.

S5 - Normalize PIN terminology and approved 4/6-digit policy:

- Files: `LoginEmpPassword.xaml(.cs)`, `ManagerPassword_Wd.xaml(.cs)`, `EmployeePassword_Wd.xaml(.cs)`, settings/help/docs/tests.
- Rollback risk: low-medium; affects operator workflow.
- Tests: Staff exactly 4; non-Staff exactly 6; no raw PIN logs; placeholder PIN blocks with clear setup message.
- DB schema impact: optional constraint only after migration decision; current report recommends no immediate schema change.
- API contract impact: EmpApp PIN registration may need alignment.
- PIN behavior: yes, length normalization; still not a password framework.

S6 - Clean-install regression from empty DB to MainWindow and offline operation:

- Files: InstallationV0 tests, WPF startup tests, PlatformAppV0 integration tests.
- Rollback risk: low.
- Tests: empty DB install; Phase1 proof; Phase2 seed; restart; API offline; MainWindow local CRUD; no second pairing redeem.
- DB schema impact: none beyond existing install schema.
- API contract impact: test coverage only unless S2 incomplete.
- PIN behavior: verify seeded placeholders and post-install PIN setup path.

## 20. Exact Source Files / Methods Inspected

Employee PIN:

- `E:\Project2026\4POS\NailSalonNet8\MyData\TblEmployee.cs`
- `E:\Project2026\4POS\NailSalonNet8\MyData\eNailSalonDbContext.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Login\LoginModeResolver.cs`
- `E:\Project2026\4POS\NailSalonNet8\MyWindows\LoginEmpPassword.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\MyWindows\ManagerPassword_Wd.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\MyWindows\EmployeePassword_Wd.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\MainWindow.xaml.cs`
- `E:\Project2026\1ApiServer\ApiServer01\EmpApp\Controllers\EmpAppController.cs`

Local DB:

- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\ApplicationStartupCoordinator.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\RuntimeProfileStartupAssessmentService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\RuntimeDatabaseCredentialProvider.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\RuntimeBootstrapLocator.cs`
- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`

API authentication:

- `E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Controllers\PlatformAppV0Phase1Controller.cs`
- `E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Security\PlatformAppV0TokenService.cs`
- `E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\PlatformAppV0Module.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Infrastructure\Phase1InstallationService.cs`
- `E:\Project2026\4POS\NailSalonNet8\ConnectService\AppAuthSession.cs`
- `E:\Project2026\4POS\NailSalonNet8\ConnectService\AppJwtBootstrapper.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Station\PosStationIdentity.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Station\PosStationLocalSettings.cs`

Startup gates:

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\DatabaseStartupAssessment.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\InstallationV0CompletedReadinessService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\DevelopmentProfileLaunchPolicy.cs`
- `E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\EffectiveProductRootContext.cs`
- `E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\LaunchProvenanceContext.cs`
- `E:\Project2026\4POS\NailSalonNet8\Properties\launchSettings.json`

Phase2 seed:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\PostgreSqlPhase2ReferenceSeedExecutor.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2LegacySeedMethod.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2ReferenceDrivenManifest.cs`

Documents/reports:

- `report/report037.md` through `report/report043.md`
- `E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md`

## 21. Read-Only DB Evidence

All PostgreSQL checks ran inside `BEGIN TRANSACTION READ ONLY` followed by `ROLLBACK`.

Runtime state and counts:

- database: `obm_pos_dev_v0_pg`
- current user/session user: `hung`
- runtime state: `Activated`
- employee count: 20
- permission count: 7
- local outbox count: 62
- phase2 v002 marker count: 1

Representative CRUD privilege evidence for current user:

- `TblEmployee`: SELECT/INSERT/UPDATE/DELETE yes
- `TblEmployeePermission`: SELECT/INSERT/UPDATE/DELETE yes
- `TblLocalOutbox`: SELECT/INSERT/UPDATE/DELETE yes
- `TblPosLocal`: SELECT/INSERT/UPDATE/DELETE yes
- `TblPosRuntimeProfile`: SELECT/INSERT/UPDATE/DELETE yes
- `TblTenant`: SELECT/INSERT/UPDATE/DELETE yes

PIN column evidence:

- `TblEmployee.LoginNumber`: not nullable.
- max length: 20.
- no unique constraint found on `LoginNumber`.
- current values grouped by role all have length 20.
- duplicate LoginNumber groups: 0.

No raw PIN values were selected or printed.

## 22. No Mutation / No Raw PIN / No Secrets / No Source Push Proof

No mutation performed:

- no OBM source edit;
- no PostgreSQL write;
- no migration;
- no role/password/GRANT/REVOKE command;
- no employee PIN update;
- no token creation/rotation;
- no Pairing Code creation/redeem;
- no WPF launch;
- no service stop/start;
- no environment variable write;
- no OBM source commit/push.

No secrets printed:

- no PostgreSQL password;
- no connection string;
- no raw PIN;
- no Pairing Code;
- no WpfJwt;
- no access token;
- no refresh token;
- no protected credential contents;
- no private identity values.

## 23. Coordination Commit SHA

This report is intended to be committed and pushed as `report/report044.md` in `lequochung99/OBM-AI-Coordination`.

The final pushed commit SHA is returned by Codex after commit and push. Embedding the commit's own SHA inside this file before commit would change the SHA.

