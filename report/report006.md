# Report 006 — ApiServer Google ClientId and approved administrator local configuration

## 1. Verdict

`APISERVER_GOOGLE_ADMIN_CONFIG_READY_FOR_USER_RETEST`

ApiServer local configuration is now complete enough for the Google administrator exchange to leave the missing-ClientId branch and enter real Google ID-token validation. Physical Google browser retest is still required before declaring administrator authorization and Tenant/POS gate PASS.

## 2. Approved email precondition evidence

ApiServer user-secrets precondition:

```text
Authentication:Google:ApprovedAdminEmail present = true
masked suffix <= 4 chars = present:*.com
sha256 prefix = 93dedf75572c
```

No raw email was written to this report.

## 3. Source and target ClientId evidence

Source project:

```text
E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\OBM.PlatformAppV0.csproj
```

Target project:

```text
E:\Project2026\1ApiServer\ApiServer01\ApiServer01.csproj
```

Sanitized evidence after configuration:

```text
Source ClientId present = true
Source ClientId mask = present:*nt.com
Source ClientId sha256 prefix = ffa54b4d5ffd

Target ClientId present = true
Target ClientId mask = present:*nt.com
Target ClientId sha256 prefix = ffa54b4d5ffd

Source/target ClientId hash match = true
```

No raw ClientId was written to this report.

## 4. Exact user-secret keys changed

Changed exactly one ApiServer user-secret key:

```text
Authentication:Google:ClientId
```

The existing approved administrator key remained present and was not changed to another identity:

```text
Authentication:Google:ApprovedAdminEmail
```

No compatibility keys were required because the current ApiServer `PlatformAppV0Options` loader already resolves:

```text
Authentication:Google:ClientId
Authentication:Google:ApprovedAdminEmail
```

## 5. ClientSecret confirmation

ApiServer target evidence:

```text
Authentication:Google:ClientSecret present = false
```

No ClientSecret was copied or set in ApiServer.

## 6. Effective options verification

Runtime proof after restart:

```text
POST http://127.0.0.1:7161/api/platform-v0/admin/google/exchange
dummy token
HTTP 401
resultCode = GOOGLE_ID_TOKEN_VALIDATION_FAILED
```

This proves effective `PlatformAppV0Options.GoogleClientId` is now loaded by the running ApiServer process. If it were still absent, the controller would return:

```text
GOOGLE_CLIENT_ID_NOT_CONFIGURED
```

## 7. Synthetic exchange before/after

Before prompt006, from report005:

```text
HTTP 401
resultCode = GOOGLE_CLIENT_ID_NOT_CONFIGURED
```

After prompt006:

```text
HTTP 401
success = false
resultCode = GOOGLE_ID_TOKEN_VALIDATION_FAILED
contractVersion = platform-app-v0.phase1
```

The dummy token did not succeed, as required.

## 8. Build and test results

Command:

```text
dotnet build E:\Project2026\PlatformAppV0\PlatformAppV0.sln
```

Result:

```text
PASS
0 warnings
0 errors
```

Command:

```text
dotnet test E:\Project2026\PlatformAppV0\PlatformAppV0.sln --no-build --logger "console;verbosity=minimal"
```

Result:

```text
PASS
Failed: 0
Passed: 11
Skipped: 0
Total: 11
```

Command:

```text
dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0" --logger "console;verbosity=minimal"
```

Result:

```text
PASS
Failed: 0
Passed: 13
Skipped: 0
Total: 13
```

ApiServer focused tests were build-enabled.

## 9. Runtime process evidence

Stopped stale processes:

```text
ApiServer01 PID 51064
OBM.PlatformAppV0 PID 7084
```

Restarted canonical runtimes:

```text
ApiServer01 PID 6544
Path E:\Project2026\1ApiServer\ApiServer01\bin\Debug\net8.0\ApiServer01.exe
Port 7161

OBM.PlatformAppV0 PID 53404
Path E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\bin\Debug\net8.0\OBM.PlatformAppV0.exe
Port 7012
```

Listening evidence:

```text
0.0.0.0:7161 -> PID 6544
127.0.0.1:7012 -> PID 53404
::1:7012 -> PID 53404
```

## 10. Runtime readiness probes

ApiServer:

```text
GET http://127.0.0.1:7161/api/platform-v0/readiness
HTTP 200
resultCode = PLATFORM_V0_PHASE1_READY
implementationState = Phase1
```

PlatformAppV0:

```text
GET https://localhost:7012/
HTTP 200
```

## 11. User acceptance matrix

| Check | Status |
|---|---|
| ApiServer approved email pre-seeded | PASS |
| Source ClientId present | PASS |
| Target ClientId set | PASS |
| Source/target ClientId hash match | PASS |
| ApiServer ClientSecret absent | PASS |
| Synthetic exchange no longer returns `GOOGLE_CLIENT_ID_NOT_CONFIGURED` | PASS |
| Dummy token rejected | PASS |
| Google browser login retest | PENDING USER |
| Platform administrator authorization PASS | PENDING USER |
| Approved identity summary populated | PENDING USER |
| Create/Select Tenant and POS1 enabled | PENDING USER |
| Pairing Code created | NOT STARTED |
| WPF redeem / Phase 2 | NOT STARTED |

## 12. Remaining blocker and risk

Remaining user-side physical retest:

```text
Google login
Authorize Platform Administrator
Tenant/POS gate
```

If physical retest fails, preserve the exact safe backend result code, especially:

```text
GOOGLE_ID_TOKEN_INVALID_AUDIENCE
GOOGLE_EMAIL_NOT_VERIFIED
PLATFORM_ADMIN_BOOTSTRAP_IDENTITY_NOT_FOUND
PLATFORM_ADMIN_IDENTITY_MISMATCH
```

Do not map those failures back to a generic authentication result.

## 13. Source Git and no-push confirmation

No source commit or source push was performed under:

```text
E:\Project2026
```

No source files were edited in prompt006. The only permitted mutation was local ApiServer user-secrets.

The source worktree remains broadly dirty from pre-existing work and previous approved tasks. Prompt006 did not reset, clean, stash, checkout or restore anything.

## 14. Secret handling confirmation

No password, ClientSecret, raw ClientId, raw approved email, raw subject, token, cookie, authorization header or connection string was added to:

- source code;
- appsettings;
- coordination report;
- coordination Git commit.

Only presence, mask and SHA-256 prefix evidence is recorded here.

## 15. Exact user retest steps

1. Open:

```text
https://localhost:7012
```

2. Sign out to remove stale cookies if the UI shows an existing session.
3. Sign in again with the approved Google account.
4. Click:

```text
Authorize Platform Administrator
```

5. Confirm the URL does not contain:

```text
HTTP_401_GOOGLE_CLIENT_ID_NOT_CONFIGURED
```

6. Expected after successful authorization:

```text
Platform administrator authorization = PASS
Approved identity summary = populated
Create/Select Tenant and POS1 = enabled
```

7. Click `Create/Select Tenant and POS1` only after authorization is PASS.
8. Do not create a Pairing Code in this task.
9. Do not start WPF redeem or Phase 2 in this task.

## 16. Coordination commit

This report is intended to be committed only to:

```text
lequochung99/OBM-AI-Coordination
report/report006.md
```
