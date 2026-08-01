# Report 045 - Local-DB-First Runtime Startup Simplification

## 1. Verdict

OBM_POS_LOCAL_DB_FIRST_RUNTIME_READY_FOR_USER_RETEST

Prompt executed: `prompt/prompt045.md`

This implements S1 only from report044: ordinary OBM-POS runtime startup is now local-DB-first. No refresh-token lifecycle, PIN-length normalization, Phase 2 redesign, PostgreSQL schema change, role/GRANT change, or database mutation was implemented.

## 2. Pre-change over-engineered startup decision tree

Pre-change normal startup followed this shape:

1. `App.xaml.cs` resolved ProductRoot/config and ran `DevelopmentProfileLaunchPolicy`.
2. `ApplicationStartupCoordinator` loaded runtime bootstrap and DPAPI database password.
3. `RuntimeProfileStartupAssessmentService` opened PostgreSQL and loaded singleton `TblPosRuntimeProfile`.
4. It verified `RuntimeState=Activated`, schema version, and station/profile identity.
5. For the InstallationV0 lane, it called `InstallationV0CompletedReadinessService`.
6. That service required Phase1 `ApiAuthorized` checkpoint, Phase2 v002 marker, tenant/POS/profile match, exact permission count 7, exact employee count 20, exact outbox evidence 4/20, and v002 marker proof.
7. Only then did it return `InstalledHealthy`.

That made installation/audit proof behave like normal runtime authentication.

## 3. Exact gates removed from normal runtime

Removed from ordinary `InstalledHealthy` routing:

- Phase1 checkpoint requirement.
- Protected WpfJwt/bootstrap token requirement.
- API token availability/expiry requirement.
- Exact Phase2 v002 marker gate as startup auth.
- Exact employee count 20.
- Exact permission count 7.
- Exact outbox counts/evidence.
- Launch provenance as local runtime authorization.
- Employee operational PIN configuration.

The detailed Phase proof service remains available for InstallationV0 diagnostics.

## 4. Exact gates retained for local DB readiness

Normal runtime still requires:

- runtime bootstrap/config resolved;
- DPAPI database password readable;
- PostgreSQL connection/authentication succeeds;
- `dbo."TblPosRuntimeProfile"` exists;
- singleton runtime profile exists;
- `RuntimeState=Activated`;
- supported schema version `20260730.004`;
- runtime profile database matches bootstrap database;
- non-empty runtime TenantGuid/PosGuid/DeviceGuid;
- saved station identity, when present, matches runtime profile;
- `TblTenant` table exists and contains exactly one matching TenantGuid/TenantCode row;
- `TblPosLocal` table exists and contains exactly one matching TenantGuid/PosGuid/slot row.

New safe evidence string: `LOCAL_DB_FIRST_RUNTIME_READY`.

## 5. Development-only guard scope

`DevelopmentProfileLaunchPolicy` still rejects unsafe Development lanes:

- missing explicit ProductRoot;
- non-approved V0 ProductRoot;
- ProgramData/Royal/Production/2Platform paths;
- non-approved Development database.

It no longer treats launch provenance as ordinary runtime authentication once the explicit root and database lane are approved.

## 6. InstallationV0 diagnostics/upgrade proof scope

`InstallationV0CompletedReadinessService` was intentionally preserved. It still verifies Phase1 checkpoint, Phase2 marker, identity spine, exact seed counts, and outbox evidence for diagnostics/upgrade proof and `Open OBM-POS` preflight context.

Normal runtime no longer calls it before every MainWindow route.

## 7. New local-DB-first startup decision tree

```text
Start OBM-POS
  -> resolve ProductRoot/config
  -> apply Development-only protected-lane guard
  -> locate runtime DB bootstrap
  -> read DPAPI DB password
  -> open PostgreSQL
  -> verify runtime profile/schema/state
  -> verify local Tenant/POS/profile identity consistency
  -> InstalledHealthy / MainWindow
  -> API/sync bootstrap runs after MainWindow and may defer
```

## 8. API-offline and expired-token behavior

After `MainWindow.Show`, `IAppJwtBootstrapper.EnsureAppJwtAsync()` exceptions are now logged as:

`localMode=OfflineDeferred`

The old modal title `POS API login error` was removed from `App.xaml.cs`. Token missing/expired still defers sync/SignalR startup without blocking local POS UI.

## 9. Employee operational PIN boundary

No PIN schema, values, validators, or manager/Staff behavior were changed. Normal startup technical summary now explicitly records:

`EmployeeOperationalPinRequired=False`

Employee `LoginNumber` remains an operational PIN, not PostgreSQL auth, API auth, tenant proof, device proof, or installation proof.

## 10. Open OBM-POS same-process handoff behavior

The same-process handoff still verifies and applies the explicit InstallationV0 ProductRoot. After that it reruns the same minimal local DB readiness assessment and opens exactly one `MainWindow` when healthy.

It no longer depends on normal runtime calling detailed Phase2 proof as an authentication gate.

## 11. No-mutation startup proof

Code changes are read-only at startup:

- no seed execution added;
- no migration execution added;
- no Pairing Code redeem added;
- no employee PIN update added;
- no outbox insert added;
- no runtime state transition added;
- no PostgreSQL role/user/GRANT/REVOKE added.

Physical DB evidence was collected in `BEGIN TRANSACTION READ ONLY` and ended with `ROLLBACK`.

## 12. Exact source files changed

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Startup\RuntimeProfileStartupAssessmentService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\DevelopmentProfileLaunchPolicy.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\Startup\RuntimeProfileStartupAssessmentServiceTests.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\R4DevelopmentProfileLauncherTests.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

## 13. Build/test commands and counts

Command:

```powershell
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj
```

Result: PASS, 0 warnings, 0 errors.

Command:

```powershell
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj
```

Result: PASS, 176 warnings, 0 errors. Warnings are existing nullable/analyzer/source warnings in the broader dirty tree.

Command:

```powershell
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap|FullyQualifiedName~DevelopmentProfile"
```

Result: PASS, 113 passed, 0 failed, 0 skipped.

## 14. Read-only DB evidence

Target database: `obm_pos_dev_v0_pg`

Evidence mode:

- bootstrap metadata read from an existing local ProductRoot config that targets `obm_pos_dev_v0_pg`;
- DPAPI password used only in process memory;
- no password or connection string printed;
- SQL transaction opened as `READ ONLY`;
- transaction ended with `ROLLBACK`.

Observed values:

- current database: `obm_pos_dev_v0_pg`
- runtime DB user shown by PostgreSQL: `postgres`
- `TblEmployeePermission`: 7
- `TblEmployee`: 20
- `TblLocalOutbox`: 62
- `Phase2TrialCompletionMarker`: 2
- `TblPosRuntimeProfile`: 1
- `TblPosRuntimeStateHistory`: 1
- runtime profile state: `Activated`, count 1

## 15. Exact operator physical retest steps

Route A:

1. Start Visual Studio with `NailSalonNet8` as startup project.
2. Select `OBM-POS Runtime Development`.
3. Press F5.
4. Expected: `MainWindow` opens; no InstallationV0Window; no Pairing Code request.

Route B:

1. Start InstallationV0 diagnostics.
2. Click `Open OBM-POS` only after diagnostics shows completed local baseline.
3. Expected: same process opens `MainWindow` after local DB readiness.

Route C:

1. Stop/unreachable API only.
2. Start `OBM-POS Runtime Development`.
3. Expected: `MainWindow` opens in local mode; API/sync deferred; local CRUD remains available.

## 16. Risks/deferred work

Deferred intentionally:

- formal access-token plus refresh-token runtime contract;
- API reconnect/offline status UX formalization;
- Staff 4-digit and non-Staff 6-digit PIN normalization;
- historical outbox/marker cleanup;
- production-vs-development launch profile cleanup beyond protected-lane safety.

## 17. No secrets/no DB mutation/no source push proof

No secrets printed:

- no PostgreSQL password;
- no connection string;
- no raw employee PIN;
- no Pairing Code;
- no WpfJwt;
- no access/refresh token.

No DB mutation:

- read-only evidence used `BEGIN TRANSACTION READ ONLY`;
- final SQL operation was `ROLLBACK`;
- no schema, role, user, GRANT/REVOKE, seed, marker, runtime-state, employee, outbox, or token mutation.

No OBM source push:

- OBM source files were edited locally only;
- no OBM source commit or push was performed.

## 18. Coordination commit SHA

This report is committed and pushed as `report/report045.md` in `lequochung99/OBM-AI-Coordination`.

The final pushed commit SHA is returned by Codex after commit and push.
