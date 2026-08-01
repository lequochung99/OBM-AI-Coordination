# Report 050 — API-Independent Local POS Startup

## 1. Verdict

OBM_POS_API_INDEPENDENT_LOCAL_STARTUP_READY_FOR_USER_RETEST

Prompt050 corrected the InstallationV0 diagnostics dependency that treated an expired/unauthorized Phase 1 API bootstrap credential as local Phase 2 incompleteness. Local installed-runtime status is now rendered and used independently from API/cloud status.

## 2. Physical prompt049 HTTP 401 evidence

Operator evidence for build `prompt049`:

- ProductRoot: `E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot`.
- Target DB: `obm_pos_dev_v0_pg`.
- Protected API proof showed `Protected hello failed: WPF_HELLO_HTTP_401`.
- UI showed `Phase 2 Local DB Baseline: blocked until Phase 1 passes`.
- `Open OBM-POS` was disabled.
- `Redeem Pairing Code` was enabled.

The same local DB had already been verified as installed and Activated, so the HTTP 401 represented API/cloud reauthorization need, not local install failure.

## 3. Exact incorrect Phase1-to-Phase2 UI dependency

The incorrect dependency was in `InstallationV0Window.HydratePhase2Async`:

- if `phase1.Passed == false` or `phase1.Identity == null`, it returned immediately;
- it set Phase 2 text to `blocked until Phase 1 passes`;
- it disabled `Open OBM-POS`;
- it never called `Phase2StartupHydrationService`.

`Phase1InstallationService.TryResumeAsync` also returned `Identity = null` when protected hello failed with HTTP 401, even though the durable `ApiAuthorized` checkpoint still contained safe Tenant/POS/attempt metadata. This made a cloud credential failure overwrite the local database status.

## 4. Local POS status model

Added/exposed `InstallationV0LocalPosStatus` with:

- `LocalDatabaseConfigResolved`
- `LocalDatabaseAuthenticationSucceeded`
- `SchemaReady`
- `RuntimeProfileCount`
- `RuntimeState`
- `TenantIdentityConsistent`
- `PosIdentityConsistent`
- `LocalPosReady`
- `LocalPosResultCode`

The UI now renders `Local POS status: Ready / Activated` when Phase2 hydration proves v002 complete and runtime state is `Activated`.

## 5. API/cloud status model

Added/exposed `InstallationV0ApiCloudStatus` and `InstallationV0ApiStatusKind`:

- `Online`
- `OfflineDeferred`
- `ReauthorizationRequired`
- `Unknown`

For `WPF_HELLO_HTTP_401`, the UI reports API status as `Reauthorization Required` while preserving local installed-runtime status. No refresh-token implementation was added.

## 6. Corrected hydration ordering and source of truth

Corrected sequence:

1. Render API/cloud status from Phase1 resume result.
2. Choose local hydration identity from `phase1.Identity ?? phase1.LocalCheckpointIdentity`.
3. Hydrate Phase2 from durable local sources with `Phase2StartupHydrationService`.
4. Render Phase2/local POS status from local DB evidence.
5. Only enable mutating Phase2 install action when Phase1 passes and local Phase2 status says v002 is available.
6. Enable `Open OBM-POS` when local Phase2 status is complete, independent of API 401.

`Phase1InstallationResult` now carries `LocalCheckpointIdentity`, built from safe `ApiAuthorizedCheckpointV0` metadata. It contains no token, pairing code, password, connection string, or secret.

## 7. Open OBM-POS enablement rule

New effective rule:

- `Open OBM-POS` is enabled when local Phase2 hydration reports `IsV002Complete == true` and runtime state is `Activated`.
- It no longer requires current Phase1 protected hello success.
- The button still calls the prompt049 structured router and succeeds only when `RouteDecision=OpenMainPos` and `MainWindowVisible=true`.

The mutating Phase2 install/upgrade button remains Phase1-gated to preserve initial installation semantics.

## 8. Direct runtime route behavior

Direct runtime startup remains local-DB-first through the prompt049 router. API/bootstrap/sync failures after local readiness are deferred and must not open InstallationV0 or block MainWindow when local runtime is healthy.

No WPF physical launch was performed in this task, per prompt050 physical execution policy.

## 9. Initial installation versus installed-runtime distinction

Initial clean installation still requires Phase1 to redeem Pairing Code, verify API identity, protect the bootstrap credential, and persist checkpoint identity before Phase2 can mutate the local database.

After durable Phase2 completion and `TblPosRuntimeProfile.RuntimeState=Activated`, a later expired bootstrap/API credential is an API reauthorization problem, not evidence that local installation disappeared.

## 10. No-mutation proof

Prompt050 source changes are UI/status/handoff logic only. During this task:

- no Pairing Code was redeemed;
- no WPF was launched;
- no PostgreSQL query or mutation was performed;
- no seed, migration, marker rewrite, runtime-state/history update, outbox insert, employee/PIN update, role/password change, or environment-variable write was performed.

## 11. Exact source files changed

OBM source files changed locally only:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\Phase1InstallationResult.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0StatusModels.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Infrastructure\Phase1InstallationService.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

No OBM source commit or push was performed.

## 12. Build/test commands and counts

Commands run:

```powershell
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal -clp:ErrorsOnly
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal -clp:ErrorsOnly
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap|FullyQualifiedName~OpenObmPos|FullyQualifiedName~RuntimeState|FullyQualifiedName~Offline" -v minimal -clp:ErrorsOnly
```

Results:

- InstallationV0 build: passed, 0 warnings, 0 errors.
- WPF build: passed, 0 warnings, 0 errors in the final incremental verification.
- Filtered tests: passed, 110 passed, 0 failed, 0 skipped.

Note: an earlier parallel build attempt hit an `InstallationV0.dll` file lock because InstallationV0 and WPF builds ran at the same time. The subsequent sequential WPF build passed.

## 13. Prompt050 label proof

`InstallationV0BuildInfo.CoordinationPromptLabel` is now `prompt050`.

The InstallationV0 title remains:

`OBM InstallationV0 Phase 1/2 - {InstallationV0BuildInfo.CoordinationPromptLabel}`

The visible build label remains:

`Build label: {InstallationV0BuildInfo.CoordinationPromptLabel}`

## 14. Exact operator retest steps

Route A — InstallationV0 with expired WpfJwt:

1. Start prompt050 InstallationV0 with the existing V0 ProductRoot.
2. Confirm `Protected hello failed: WPF_HELLO_HTTP_401` may still appear as API status.
3. Confirm `Phase 2 Local DB Baseline: Phase 2 v002 Complete` remains visible.
4. Confirm `Local POS status: Ready / Activated`.
5. Confirm `API status: Reauthorization Required` or `OfflineDeferred`.
6. Confirm `Open OBM-POS` is enabled.
7. Click `Open OBM-POS` once.
8. Require `OPEN_POS_MAINWINDOW_SHOWN` and MainWindow visible.

Route B — Runtime Development with API unavailable/401:

1. Start `NailSalonNet8` using `OBM-POS Runtime Development`.
2. Confirm local MainWindow opens directly from `RuntimeState=Activated`.
3. Confirm API state is deferred/reauthorization-required rather than installation-required.

## 15. Deferred refresh-token work

Still deferred:

- access-token plus refresh-token runtime lifecycle;
- refresh endpoint and rotation/revocation handling;
- reauthorization UX beyond optional Pairing Code availability;
- online/offline reconnect status polish.

## 16. No secrets/no DB mutation/no source push proof

No secrets were printed or persisted in report evidence:

- no PostgreSQL password;
- no connection string;
- no Pairing Code;
- no WpfJwt;
- no access token;
- no refresh token;
- no raw credential;
- no customer/employee data.

No DB mutation was performed. No OBM source was committed or pushed. Only this coordination report is intended to be committed and pushed.

## 17. Coordination commit SHA

Final pushed coordination commit SHA is reported in the Codex final response; embedding the commit's own SHA into this file would change that SHA.
