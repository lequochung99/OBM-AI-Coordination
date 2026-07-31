# REPORT 010 - Fresh Pairing Code Expiration Correction

## 1. Verdict

FRESH_PAIRING_CODE_EXPIRATION_CORRECTION_READY_FOR_USER_RETEST

## 2. Physical Symptom

The operator observed that a freshly generated Pairing Code still failed in WPF with HTTP 400 and safe resultCode `PAIRING_CODE_EXPIRED`.

## 3. Lifecycle Timeline

- Prompt009 established a working admin/UI path and left the system ready for a physical retry.
- Prompt010 investigated the fresh-code failure without printing raw Pairing Code, token, cookie, or secrets.
- ApiServer and PlatformAppV0 were restarted after correction.
- WPF was not launched during this task.

## 4. Safe Timestamp And Correlation Matrix

Safe state source: `E:\Project2026\1ApiServer\ApiServer01\bin\Debug\net8.0\PlatformAppV0\platform-app-v0-state.json`.

Server UTC sampled: `2026-07-31T22:14:30.0752928Z`.

Pairing authorization count: 6.

Observed safe rows:

| Status | CreatedAtUtc | ExpiresAtUtc | RedeemedAtUtc | RedeemClientRequestGuid | InstallationClientGuid | ExpiredAtSample |
| --- | --- | --- | --- | --- | --- | --- |
| Active | 2026-07-31T21:17:32.7557586+00:00 | 2026-07-31T21:32:32.7167246+00:00 | absent | absent | absent | true |
| Redeemed | 2026-07-31T21:30:57.7558169+00:00 | 2026-07-31T21:45:57.4032385+00:00 | present | present | absent | true |
| Active | 2026-07-31T21:55:56.6123881+00:00 | 2026-07-31T22:10:56.5971016+00:00 | absent | absent | absent | true |
| Active | 2026-07-31T21:57:09.1024885+00:00 | 2026-07-31T22:12:09.1000396+00:00 | absent | absent | absent | true |
| Active | 2026-07-31T22:00:12.0131411+00:00 | 2026-07-31T22:15:12.0095066+00:00 | absent | absent | absent | false |
| Active | 2026-07-31T22:00:25.2660716+00:00 | 2026-07-31T22:15:25.2641704+00:00 | absent | absent | absent | false |

No pairing code digest, raw code, token, cookie, or credential value is recorded here.

## 5. Exact Expired Branch

File: `E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Controllers\PlatformAppV0Phase1Controller.cs`.

Method: `Redeem`.

Condition after correction:

```csharp
if (authorization.ExpiresAtUtc <= now)
```

Safe expired response now includes safe correlation fields through `PlatformAppV0ProblemResponse`: pairing authorization identity, created timestamp, expiration timestamp, and server UTC. It does not include code, digest, token, cookie, or secret material.

## 6. WPF Current Code Versus Cached Code

WPF reads the current PasswordBox value on each click in `InstallationV0Window` and clears the PasswordBox after the call.

Focused test added: `FailedAttemptThenNewInput_SendsCurrentPairingCodeOnly`.

The test proves the first submitted value and second submitted value are distinct operator inputs, and the second redeem sends the second value rather than a cached previous value.

## 7. Runtime State Consistency

ApiServer and PlatformAppV0 were restarted from current debug binaries after the expanded prompt010 test coverage:

- ApiServer PID: 39192
- ApiServer start time UTC: 2026-07-31T22:19:11.7898519Z
- ApiServer path: `E:\Project2026\1ApiServer\ApiServer01\bin\Debug\net8.0\ApiServer01.exe`
- ApiServer binary LastWriteTimeUtc: 2026-07-31T22:12:52.5770495Z
- PlatformAppV0 PID: 33828
- PlatformAppV0 start time UTC: 2026-07-31T22:19:13.8426545Z
- PlatformAppV0 path: `E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\bin\Debug\net8.0\OBM.PlatformAppV0.exe`
- PlatformAppV0 binary LastWriteTimeUtc: 2026-07-31T22:11:27.5699176Z
- PlatformAppV0 API base URL: `http://127.0.0.1:7161`

Readiness probe:

- `GET http://127.0.0.1:7161/api/platform-v0/readiness`
- HTTP 200
- resultCode `PLATFORM_V0_PHASE1_READY`
- implementationState `Phase1`

PlatformAppV0 root probe:

- `GET https://localhost:7012/`
- HTTP 200

Unauthenticated protected WPF endpoint probe:

- `GET http://127.0.0.1:7161/api/platform-v0/wpf/bootstrap/hello`
- HTTP 401 as expected without WpfJWT.

No `NailSalonNet8` WPF process was running during restart/probe.

## 8. Root Cause

Two defects created the fresh-code failure risk:

1. Pairing Code creation split expiration authority between PlatformAppV0 caller input and ApiServer. The ApiServer now owns the expiration by assigning server UTC plus 15 minutes.
2. Redeem lookup used a broad OR match between Pairing Code digest and RedeemClientRequestGuid. That allowed stale request identity to shadow a newly entered code path. Redeem now separates code lookup from replay lookup and returns safe `PAIRING_REDEEM_CONFLICT` if both match different authorizations.

The safe state matrix showed newer active records existed, so the failure was not a global inability to create fresh Pairing Codes.

## 9. Why Previous Tests Missed It

Previous tests did not cover:

- caller-supplied stale expiration during Pairing Code creation;
- before-expiry versus at/after-expiry behavior;
- local-timezone caller expiry being ignored in favor of server UTC;
- fresh create request not returning an expired prior authorization record;
- an old redeem request identity coexisting with a fresh code;
- WPF retry after one failed code followed by a different operator input;
- safe parsing of non-200 problem responses;
- visible prompt label proving the active WPF binary is prompt010.

## 10. Files Changed

Source files changed for prompt010:

- `E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0.Contracts\PlatformAppV0Contracts.cs`
- `E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Controllers\PlatformAppV0Phase1Controller.cs`
- `E:\Project2026\1ApiServer\ApiServer01.Tests\PlatformAppV0\PlatformAppV0Phase1Tests.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\Phase1InstallationResult.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Infrastructure\Phase1InstallationService.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

Only this report is intended to be committed in the coordination repository.

## 11. Correction

Implemented corrections:

- ApiServer owns Pairing Code expiration using server time.
- Redeem lookup separates code lookup from replay lookup.
- Conflict between a fresh code and stale request identity returns safe `PAIRING_REDEEM_CONFLICT`.
- Expired response includes safe timestamp/correlation fields.
- WPF safely parses non-200 problem responses and reports safe resultCode instead of crashing on the wrong contract.
- WPF adds visible prompt010 build labeling.
- WPF records safe input diagnostics: present, length, and request-sent boolean.
- Added API tests for before-expiry PASS, at/after-expiry EXPIRED, UTC/local mismatch regression, and fresh create not returning expired prior records.

## 12. Tests And Builds

Builds:

- `dotnet build E:\Project2026\PlatformAppV0\PlatformAppV0.sln` PASS.
- `dotnet build E:\Project2026\1ApiServer\ApiServer01\ApiServer01.csproj` PASS.
- `dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj` PASS.
- `dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj` PASS.

Focused tests:

- `dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0"` PASS: 22 passed.
- `dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"` PASS: 23 passed.

Existing unrelated warnings remain in the broader projects.

## 13. Runtime Evidence

Runtime evidence after restart:

- ApiServer PID 39192 listening on 7161.
- ApiServer start time UTC 2026-07-31T22:19:11.7898519Z.
- ApiServer binary LastWriteTimeUtc 2026-07-31T22:12:52.5770495Z.
- PlatformAppV0 PID 33828 listening on 7012.
- PlatformAppV0 start time UTC 2026-07-31T22:19:13.8426545Z.
- PlatformAppV0 binary LastWriteTimeUtc 2026-07-31T22:11:27.5699176Z.
- Readiness HTTP 200 with `PLATFORM_V0_PHASE1_READY`.
- PlatformAppV0 HTTPS root HTTP 200.
- WPF protected hello endpoint HTTP 401 without token.
- No WPF process running.

## 14. Prompt Label

Exact label constant:

```csharp
CoordinationPromptLabel = "prompt010"
```

Exact WPF title expression:

```csharp
OBM InstallationV0 Phase 1 - prompt010
```

Exact visible header:

```text
Build label: prompt010
```

## 15. Stale Prompt Label

Focused test verifies `prompt009` is absent from active `InstallationV0Window` and `InstallationV0BuildInfo` source.

## 16. Secret Handling

No raw Pairing Code, WpfJWT, cookie, secret, digest, or connection string was printed in this report.

The WPF diagnostic fields are safe only:

- Pairing Code input present.
- Pairing Code input length.
- Pairing Code request sent.

## 17. Database And Phase 2

No local POS PostgreSQL database was created, connected, migrated, seeded, or mutated.

No Phase 2 implementation or runtime operation was performed.

No WPF process was started in this task.

## 18. User Retest Steps

1. Open PlatformAppV0 at `https://localhost:7012`.
2. Login as approved Platform administrator.
3. Open the V0 tenant/POS area.
4. Create one fresh canonical Pairing Code.
5. Start WPF InstallationV0 from Visual Studio or the approved profile.
6. Confirm the WPF window title/header shows `prompt010`.
7. Enter the fresh Pairing Code manually.
8. Redeem once.
9. Expected pass path: WPF receives WpfJWT, calls protected bootstrap endpoints, persists protected credential, persists checkpoint, and stops before any database activity.
10. On failure, record only HTTP status and safe resultCode.

## 19. Coordination Commit

Initial report commit: `01eac678de3662c350d70d5682ea18fe58180faf`.

This report update covers the expanded prompt010 requirements from commit `95ff75050af6f279322de8d10dfe88682db3dc4c`.
