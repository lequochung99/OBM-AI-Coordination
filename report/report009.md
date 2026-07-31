# Report 009 — WpfJwt protected hello endpoint and latest runtime handoff

## 1. Verdict

`WPFJWT_PROTECTED_HELLO_AND_PHASE1_FLOW_READY_FOR_USER_PHYSICAL_TEST`

Implementation, focused tests, builds, and runtime handoff are complete. ApiServer and PlatformAppV0 are running from the latest Debug output. WPF is not running; operator should run WPF from Visual Studio Debug and create/use a fresh Pairing Code manually.

## 2. Endpoint, controller, method, authorization

Endpoint:

```http
GET /api/platform-v0/wpf/bootstrap/hello
Authorization: Bearer <WpfJwt>
```

Controller:

```text
ApiServer01.PlatformAppV0.Controllers.PlatformAppV0Phase1Controller
```

Method:

```text
GetHelloWorld
```

Authorization mask:

```csharp
[Authorize(
    AuthenticationSchemes = PlatformAppV0Contract.WpfJwtScheme,
    Policy = PlatformAppV0Contract.WpfInstallBootstrapPolicy)]
```

Exact mapping:

```text
PlatformAppV0Contract.WpfJwtScheme = WpfJwt
PlatformAppV0Contract.WpfInstallBootstrapPolicy = WpfInstallBootstrap
```

No `AllowAnonymous`, Google token, Platform administrator cookie, or manual token parsing is used for this endpoint.

## 3. Sanitized controller and contract

```csharp
public const string WpfJwtProtectedControllerReached =
    "WPF_JWT_PROTECTED_CONTROLLER_REACHED";

public const string WpfJwtProtectedControllerMessage =
    "HELLO_FROM_PLATFORMAPPV0_WPFJWT_PROTECTED_CONTROLLER";

public sealed record WpfProtectedHelloResponse(
    bool Success,
    bool Authenticated,
    string ResultCode,
    string Message,
    DateTimeOffset ServerUtc,
    string ContractVersion);
```

```csharp
[HttpGet("wpf/bootstrap/hello")]
[Authorize(
    AuthenticationSchemes = PlatformAppV0Contract.WpfJwtScheme,
    Policy = PlatformAppV0Contract.WpfInstallBootstrapPolicy)]
public ActionResult<WpfProtectedHelloResponse> GetHelloWorld()
{
    var authenticated = User.Identity?.IsAuthenticated == true;
    return Ok(new WpfProtectedHelloResponse(
        true,
        authenticated,
        PlatformAppV0Contract.WpfJwtProtectedControllerReached,
        PlatformAppV0Contract.WpfJwtProtectedControllerMessage,
        DateTimeOffset.UtcNow,
        PlatformAppV0Contract.Version));
}
```

`/api/platform-v0/wpf/bootstrap/me` was also made explicit with the same `AuthenticationSchemes = WpfJwt` and `Policy = WpfInstallBootstrap` mask.

## 4. WPF call order

Initial redeem flow:

```text
POST /api/platform-v0/wpf/pairing/redeem
-> validate redeem contract and WpfJwt
-> GET /api/platform-v0/wpf/bootstrap/hello with Bearer WpfJwt
-> require HTTP 200 success contract
-> require resultCode WPF_JWT_PROTECTED_CONTROLLER_REACHED
-> require authenticated true
-> require exact marker from API response
-> require non-default serverUtc
-> GET /api/platform-v0/wpf/bootstrap/me with same Bearer WpfJwt
-> exact identity comparison
-> DPAPI protect/readback
-> ApiAuthorized checkpoint write/readback
```

Resume flow:

```text
read ApiAuthorized checkpoint
-> DPAPI-unprotect WpfJwt
-> GET protected hello with stored WpfJwt
-> GET /bootstrap/me with stored WpfJwt
-> compare checkpoint identity
-> mark RestartResume verified
-> no second redeem
```

## 5. UI proof implementation

WPF proof items now include:

```text
Protected WpfJwt controller reached
Protected controller resultCode verified
Protected controller authenticated = true
Protected controller message received from API
Protected controller server UTC received
```

The displayed marker is read from deserialized HTTP response:

```text
HELLO_FROM_PLATFORMAPPV0_WPFJWT_PROTECTED_CONTROLLER
```

WPF does not create this marker locally as a fake PASS string.

## 6. Failure matrix

| Failure | Result |
|---|---|
| No/invalid token to hello | `WPF_HELLO_HTTP_401` |
| Policy/scope rejected by hello | `WPF_HELLO_HTTP_403` |
| Other failed hello HTTP result | `WPF_HELLO_HTTP_FAILED` |
| Unreadable hello contract | `WPF_HELLO_CONTRACT_FAILED` |
| `authenticated=false` | `WPF_HELLO_NOT_AUTHENTICATED` |
| wrong resultCode | `WPF_HELLO_RESULT_CODE_MISMATCH` |
| missing marker | `WPF_HELLO_MESSAGE_MISSING` |
| missing/default serverUtc | `WPF_HELLO_SERVER_TIME_MISSING` |

When protected hello fails, WPF does not DPAPI-protect the credential and does not write `ApiAuthorized`.

## 7. Exact files changed

Prompt009 source files touched:

```text
E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0.Contracts\PlatformAppV0Contracts.cs
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Controllers\PlatformAppV0Phase1Controller.cs
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Controllers\PlatformAppV0ReadinessController.cs
E:\Project2026\1ApiServer\ApiServer01.Tests\PlatformAppV0\PlatformAppV0Phase1Tests.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\Phase1InstallationResult.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Infrastructure\Phase1InstallationService.cs
E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs
```

Readiness note: Phase1 startup had both mapped readiness and a readiness controller at the same route. The controller was marked `[NonController]` so the canonical mapped readiness endpoint returns 200 instead of ambiguous 500.

## 8. Tests added

ApiServer focused tests:

```text
GetHelloWorld_RequiresWpfJwtSchemeAndBootstrapPolicy
GetHelloWorld_ReturnsExactProtectedMarkerFromController
```

WPF focused tests:

```text
ProtectedHelloUnauthorized_BlocksPhase1Persistence
ProtectedHelloContractFailures_BlockPhase1Persistence
```

Existing WPF tests were updated so successful and resume flows use:

```text
redeem -> protected hello -> bootstrap/me
```

## 9. Builds and tests

Command:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj
```

Result:

```text
PASS, 0 warnings, 0 errors
```

Command:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj
```

Result:

```text
PASS, 0 warnings, 0 errors
```

Command:

```text
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0" --logger "console;verbosity=minimal"
```

Result:

```text
PASS
Failed: 0
Passed: 21
Skipped: 0
Total: 21
```

Command:

```text
dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0" --logger "console;verbosity=minimal"
```

Result:

```text
PASS
Failed: 0
Passed: 15
Skipped: 0
Total: 15
```

Additional runtime project build:

```text
dotnet build E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\OBM.PlatformAppV0.csproj
PASS, 0 warnings, 0 errors
```

The larger WPF/API test builds still emit pre-existing warnings outside the InstallationV0/PlatformAppV0 focused code.

## 10. Prompt008 regression proof

The correction remains in place:

```text
ToJwtSecondPrecision(now.AddHours(4))
DateTimeOffset.FromUnixTimeSeconds(value.ToUnixTimeSeconds())
```

Regression test still passes:

```text
JwtExpirationPrecisionMismatch_IsReportedSeparatelyFromAttemptIdentity
```

Credential expiration precision remains separated from `InstallationAttemptGuid` and `AttemptVersion`.

## 11. Runtime stop/restart evidence

Initial port state:

```text
7161: no listener
7012: no listener
```

Temporary first restart found readiness 500 due duplicate readiness route. ApiServer PID `52972` and PlatformAppV0 PID `39648` were stopped, source was corrected, tests/builds reran, and final runtime was restarted.

Final running processes:

```text
ApiServer01 PID 56720
Path E:\Project2026\1ApiServer\ApiServer01\bin\Debug\net8.0\ApiServer01.exe
StartTimeUtc 2026-07-31T21:51:40.0349477Z
Binary LastWriteTimeUtc 2026-07-31T21:51:23.8333740Z
Endpoint http://127.0.0.1:7161
Port owner 56720

OBM.PlatformAppV0 PID 51168
Path E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\bin\Debug\net8.0\OBM.PlatformAppV0.exe
StartTimeUtc 2026-07-31T21:51:40.0709617Z
Binary LastWriteTimeUtc 2026-07-31T21:51:30.6036314Z
Endpoint https://localhost:7012
Port owner 51168
```

## 12. Latest-instance flags

```text
APISERVER_LATEST_INSTANCE_PROVEN = true
PLATFORMAPPV0_LATEST_INSTANCE_PROVEN = true
LATEST_RUNTIME_INSTANCE_PROVEN = true
```

Both processes started after their final binary LastWriteTimeUtc.

## 13. Runtime probes

Probe:

```http
GET http://127.0.0.1:7161/api/platform-v0/readiness
```

Result:

```text
HTTP 200
resultCode PLATFORM_V0_PHASE1_READY
implementationState Phase1
```

Probe:

```http
GET https://localhost:7012/
```

Result:

```text
HTTP 200
```

Probe:

```http
GET http://127.0.0.1:7161/api/platform-v0/wpf/bootstrap/hello
```

Result without token:

```text
HTTP 401
```

## 14. Runtime left running

```text
ApiServer running on port 7161 = true
PlatformAppV0 running on port 7012 = true
No NailSalonNet8/WPF process running = true
```

## 15. Security and database guardrails

Confirmed:

```text
No Pairing Code in report.
No WpfJwt in report.
No Authorization header value in report.
No Google token, ClientSecret, cookie, password, or connection string in report.
No PostgreSQL connection.
No local database creation.
No migration.
No seed.
No TblLocalOutbox.
No PlatformEnrollment.
No permanent device credential.
No Phase 2.
```

## 16. Physical user retest steps

1. Open PlatformAppV0 at:

```text
https://localhost:7012
```

2. Login/authorize if required.
3. Create a fresh Pairing Code for the intended Tenant/POS1.
4. Run WPF `OBM-POS InstallationV0 Phase1` from Visual Studio Debug.
5. Enter the Pairing Code manually.
6. Click redeem.
7. Confirm WPF displays:

```text
HELLO_FROM_PLATFORMAPPV0_WPFJWT_PROTECTED_CONTROLLER
```

8. Confirm protected `/bootstrap/me` identity proof PASS.
9. Confirm DPAPI and checkpoint proof PASS.
10. Close WPF.
11. Start the same WPF ProductRoot again.
12. Confirm protected hello and `/bootstrap/me` are called by stored credential.
13. Confirm no second redeem and no second InstallationAttempt.
14. Do not start Phase 2.

## 17. Git safety

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
Only coordination report is committed/pushed
```

The source worktree remains broadly dirty from pre-existing tasks; prompt009 touched only the files listed above.

## 18. Coordination commit

This report is intended to be committed to:

```text
lequochung99/OBM-AI-Coordination
report/report009.md
```
