# Report 007 — WPF InstallationV0 redeem Pairing Code and verify API access

## 1. Verdict

`WPF_INSTALLATIONV0_PHASE1_REDEEM_VERIFY_READY_FOR_USER_PHYSICAL_TEST`

Implementation, build, and focused tests are complete for the WPF InstallationV0 Phase 1 redeem and API verification path. Full terminal PASS is pending operator physical test because Codex did not receive or redeem the one-time Pairing Code.

## 2. InstallationV0 pre-audit

Existing InstallationV0 boundary before this prompt was already small and clean-room:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0
```

Pre-existing files:

```text
README.md
InstallationV0.csproj
InstallationV0Module.cs
Application\Phase1InstallationResult.cs
Infrastructure\Phase1InstallationService.cs
Presentation\InstallationV0Window.cs
```

Existing tests:

```text
E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs
```

ApiServer PlatformAppV0 Phase 1 already had:

```text
POST /api/platform-v0/wpf/pairing/redeem
GET /api/platform-v0/wpf/bootstrap/me
WpfJwt authentication scheme
WpfInstallBootstrap policy
```

## 3. Exact architecture/call chain

```text
WPF InstallationV0Window
-> Phase1InstallationService.EnsurePendingAsync
-> writes phase1-pending.json before redeem
-> POST /api/platform-v0/wpf/pairing/redeem
-> validates WPF_JWT_ISSUED response
-> GET /api/platform-v0/wpf/bootstrap/me with Bearer WpfJwt
-> validates WPF_BOOTSTRAP_IDENTITY_VERIFIED response
-> compares scope/Tenant/POS/attempt/local-installation/expiration
-> protects WpfJwt with DPAPI
-> reads protected credential back
-> writes api-authorized.json atomically
-> reads checkpoint back
-> restart calls TryResumeAsync
-> revalidates /bootstrap/me without another redeem
```

## 4. Files changed

Prompt007 direct changes:

```text
E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0.Contracts\PlatformAppV0Contracts.cs
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Controllers\PlatformAppV0Phase1Controller.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\Phase1InstallationResult.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Infrastructure\Phase1InstallationService.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs
E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs
```

Pre-existing dirty/untracked PlatformAppV0 and InstallationV0 Phase1 files remain from earlier prompts. Source was not committed or pushed.

## 5. Redeem request/response contract implementation

WPF sends:

```text
POST /api/platform-v0/wpf/pairing/redeem
```

Request fields:

```text
pairingCode
redeemClientRequestGuid
localInstallationGuid
wpfApplicationVersion
safeMachineMetadata
```

Rules implemented:

```text
ClientRequestGuid persisted before request
LocalInstallationGuid persisted before request
Retry reuses the same pending checkpoint
Pairing Code is not written to pending checkpoint
HTTP 200 alone is insufficient
WpfJwt missing is rejected
wrong resultCode is rejected
wrong credentialType is rejected
wrong scope is rejected
empty Tenant/POS/attempt/local identity is rejected
expired credential is rejected
```

## 6. bootstrap/me implementation and comparison

WPF calls:

```text
GET /api/platform-v0/wpf/bootstrap/me
Authorization: Bearer <WpfJwt>
```

Raw token is never written to report/checkpoint/log output.

WPF compares:

```text
scope
TenantGuid
TenantCode
TenantName
PosStationId
PosGuid
PosName
SlotNumber
InstallationAttemptGuid
AttemptVersion
LocalInstallationGuid
CredentialExpiresAtUtc
ContractVersion
```

ApiServer correction:

```text
BootstrapIdentityResponse now includes CredentialExpiresAtUtc.
PlatformAppV0Phase1Controller resolves it from the WpfJwt exp claim.
```

## 7. Secret protection/checkpoint implementation

WPF local persistence order:

```text
1. /bootstrap/me PASS
2. protect WpfJwt via DPAPI CurrentUser
3. read protected credential back
4. write ApiAuthorized checkpoint atomically
5. read checkpoint back
6. compare checkpoint identity
7. mark Phase 1 authorized
```

Checkpoint contains only:

```text
ProtectedCredentialReference = dpapi:phase1-bootstrap
```

It does not contain the raw token or Pairing Code.

## 8. Restart/resume implementation

`TryResumeAsync`:

```text
reads ApiAuthorized checkpoint
unprotects DPAPI credential
calls /api/platform-v0/wpf/bootstrap/me
compares checkpoint identity with API identity
sets RestartResume=true only after revalidation
sets SecondRedeemAvoided=true
does not call redeem endpoint
```

## 9. Database non-touch proof

InstallationV0 source guard test asserts absence of:

```text
NpgsqlConnection
Host=
TblLocalOutbox
migration
DatabaseFacade
```

Runtime result model includes:

```text
LocalDatabaseTouched = false
```

No PostgreSQL database was created, opened, migrated, seeded, or probed during this task.

## 10. Build/test commands and counts

Command:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj
```

Result:

```text
PASS
0 warnings
0 errors
```

Command:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj
```

Result:

```text
PASS
0 warnings
0 errors
```

Command:

```text
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0" --logger "console;verbosity=minimal"
```

Result:

```text
PASS
Failed: 0
Passed: 14
Skipped: 0
Total: 14
```

Command:

```text
dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0" --logger "console;verbosity=minimal"
```

First run:

```text
FAILED due to stale ApiServer PID 61140 locking OBM.PlatformAppV0.Contracts.dll.
```

Correction:

```text
Stopped ApiServer PID 61140.
```

Second run:

```text
PASS
Failed: 0
Passed: 13
Skipped: 0
Total: 13
```

## 11. Runtime evidence available

Prompt007 did not run WPF physically because no Pairing Code was supplied to Codex. It prepared source/build for user Visual Studio Debug.

Stale ApiServer process stopped:

```text
ApiServer01 PID 61140 stopped
```

Observed PlatformAppV0 still running, not stopped because it appears user-owned and prompt007 only required avoiding stale WPF/debug binary confusion:

```text
OBM.PlatformAppV0 PID 57448
Port 7012
Path E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\bin\Debug\net8.0\OBM.PlatformAppV0.exe
```

No WPF `NailSalonNet8` process was left running by Codex.

## 12. Physical proof items

| Proof item | Status |
|---|---|
| Pairing Code redeemed | PENDING USER |
| WpfJwt received | PENDING USER |
| Protected API call succeeded | PENDING USER |
| WpfJwt scope verified | IMPLEMENTED, PENDING USER |
| Tenant identity verified | IMPLEMENTED, PENDING USER |
| POS station identity verified | IMPLEMENTED, PENDING USER |
| PosGuid verified | IMPLEMENTED, PENDING USER |
| Installation attempt verified | IMPLEMENTED, PENDING USER |
| Local installation identity verified | IMPLEMENTED, PENDING USER |
| Credential protected locally | IMPLEMENTED, PENDING USER |
| Protected credential readback passed | IMPLEMENTED, PENDING USER |
| Machine checkpoint persisted | IMPLEMENTED, PENDING USER |
| Checkpoint readback passed | IMPLEMENTED, PENDING USER |
| Restart/resume verified | IMPLEMENTED, PENDING USER |
| Local database touched = false | PASS |

## 13. Source Git/no push confirmation

Source repo:

```text
E:\Project2026
```

Confirmation:

```text
No source git add .
No source git add -A
No source reset/clean/stash/checkout/restore
No source commit
No source push
No local POS database activity
No Pairing Code plaintext recorded
No WPF instance left running by Codex
```

Only this coordination report is intended to be committed to GitHub.

## 14. Exact Visual Studio user test steps

1. In Visual Studio, start ApiServer with profile:

```text
ApiServer01 PlatformAppV0 Phase1
```

2. In Visual Studio, start WPF with profile:

```text
OBM-POS InstallationV0 Phase1
```

3. Confirm WPF shows:

```text
OBM-POS New Installation
API Base URL
Pairing Code
Redeem and Verify API Access
Local Database Installation - Not Started
```

4. Enter the current one-time Pairing Code manually.
5. Click `Redeem and Verify API Access`.
6. Verify each proof item is shown independently.
7. Confirm no raw token or Pairing Code is visible after success.
8. Close WPF.
9. Start the same WPF profile again with the same ProductRoot.
10. Verify resume without Pairing Code and without a second redeem.
11. Do not start Phase 2.

## 15. Remaining blockers/risks

Physical proof is pending because Codex did not receive a disposable Pairing Code.

Potential backend result codes to preserve if physical test fails:

```text
PAIRING_CODE_NOT_FOUND
PAIRING_CODE_EXPIRED
PAIRING_CODE_ALREADY_USED
WPF_JWT_INVALID
WPF_JWT_EXPIRED
WPF_JWT_SCOPE_INVALID
WPF_IDENTITY_MISMATCH
```

The source worktree is broadly dirty from pre-existing tasks; source commit remains intentionally out of scope.

## 16. Coordination commit

This report is intended to be committed only to:

```text
lequochung99/OBM-AI-Coordination
report/report007.md
```
