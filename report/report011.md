# REPORT 011 - WPF Redeem Request Identity Rotation

## 1. Verdict

WPF_REDEEM_REQUEST_IDENTITY_ROTATION_READY_FOR_USER_RETEST

## 2. Physical Evidence From Prompt010

Operator evidence showed the WPF binary was current for prompt010:

```text
Build label: prompt010
```

Before click, WPF correctly showed no current request had been sent:

```text
PAIRING_CODE_INPUT_PRESENT: false
PAIRING_CODE_INPUT_LENGTH: 0
PAIRING_CODE_REQUEST_SENT: false
```

After entering a fresh Pairing Code and clicking redeem, WPF proved it read and sent current input:

```text
HTTP 409
resultCode = PAIRING_REDEEM_CONFLICT
PAIRING_CODE_INPUT_PRESENT: true
PAIRING_CODE_INPUT_LENGTH: 14
PAIRING_CODE_REQUEST_SENT: true
```

No raw Pairing Code was present in the evidence.

## 3. Root Cause

Prompt010 correctly separated API lookup by current Pairing Code digest from replay lookup by `RedeemClientRequestGuid`. That exposed the real lifecycle issue: WPF kept the same `RedeemClientRequestGuid` across a terminal failed logical submission and a later different Pairing Code.

The stale request GUID matched an old PairingAuthorization while the fresh Pairing Code matched a different active PairingAuthorization. The API correctly returned fail-closed `PAIRING_REDEEM_CONFLICT`.

Correct semantics are:

- `LocalInstallationGuid` is stable for the machine-side installation.
- `RedeemClientRequestGuid` is stable only for one logical redeem submission/retry group.
- Terminal business outcomes close the logical submission and require a new `RedeemClientRequestGuid`.
- Unknown outcomes keep the same GUID for safe replay.

## 4. Idempotency State Machine

Unknown or transient outcome:

```text
network timeout
connection reset
HTTP 5xx
unreadable response body after possible server receipt
```

Action:

```text
keep same LocalInstallationGuid
keep same RedeemClientRequestGuid
AttemptState remains replay-safe
```

Deterministic terminal business outcome:

```text
PAIRING_CODE_EXPIRED
PAIRING_CODE_NOT_FOUND
PAIRING_CODE_ALREADY_USED
PAIRING_REDEEM_CONFLICT
PAIRING_CODE_CANCELLED
REQUEST_INVALID
```

Action:

```text
write terminal safe state
preserve LocalInstallationGuid
create new RedeemClientRequestGuid
increment AttemptGeneration
do not persist Pairing Code plaintext
```

Conflict recovery:

```text
first request returns PAIRING_REDEEM_CONFLICT
WPF rotates RedeemClientRequestGuid
WPF retries the same current in-memory Pairing Code exactly once
no infinite loop
```

## 5. Safe GUID Generation Outcome Matrix

| Step | LocalInstallationGuid | RedeemClientRequestGuid | AttemptGeneration | Outcome | Next GUID action |
| --- | --- | --- | --- | --- | --- |
| Prior terminal attempt | preserved | old logical submission GUID | 1 | `PAIRING_CODE_EXPIRED` or equivalent terminal result | rotate request GUID |
| Fresh code with stale request GUID | same local GUID | stale old GUID | 1 | `PAIRING_REDEEM_CONFLICT` | rotate request GUID |
| Rotation | same local GUID | new GUID | 2 | pending active submission | retry once if conflict |
| Retry with new GUID | same local GUID | new GUID | 2 | HTTP 200 / `WPF_JWT_ISSUED` | no further rotation |
| API state | same local GUID | new GUID | 2 | fresh authorization redeemed exactly once | old GUID still maps to old authorization |

GUID values are safe to report, but this report does not print physical Pairing Codes.

## 6. Exact Files And Methods Changed

Prompt011 source changes:

- `E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0.Contracts\PlatformAppV0Contracts.cs`
  - Added safe conflict metadata to `PlatformAppV0ProblemResponse`.
  - Added pending checkpoint fields: `AttemptGeneration`, `AttemptState`, `LastSafeResultCode`.
- `E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Controllers\PlatformAppV0Phase1Controller.cs`
  - `Redeem`: kept separated lookup and now returns current/replay safe conflict metadata.
  - `ConflictProblem`: added safe conflict response helper.
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
  - Updated label to `prompt011`.
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\Phase1InstallationResult.cs`
  - Added safe request-identity diagnostics.
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Infrastructure\Phase1InstallationService.cs`
  - Added terminal/unknown outcome state machine.
  - Added conflict one-time auto-retry.
  - Added atomic pending checkpoint rotation.
- `E:\Project2026\1ApiServer\ApiServer01.Tests\PlatformAppV0\PlatformAppV0Phase1Tests.cs`
  - Added conflict metadata/retry/replay tests.
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`
  - Added request GUID rotation, unknown outcome, 5xx, conflict auto-retry, and prompt011 label tests.

## 7. Exact Correction

WPF now:

- persists pending safe state only;
- preserves `LocalInstallationGuid`;
- rotates `RedeemClientRequestGuid` after deterministic terminal business failures;
- retries exactly once after `PAIRING_REDEEM_CONFLICT`;
- keeps the same `RedeemClientRequestGuid` for timeout/unknown/5xx cases;
- exposes safe proof items:
  - redeem logical attempt generation;
  - redeem request GUID rotated;
  - LocalInstallationGuid preserved;
  - conflict auto-retry attempted;
  - conflict auto-retry count;
  - final HTTP/resultCode;
- does not store Pairing Code plaintext, reversible fingerprint, WpfJwt, or Authorization header in pending checkpoint.

API now:

- keeps the prompt010 separated lookup guard;
- returns `PAIRING_REDEEM_CONFLICT` when current-code authorization and replay authorization differ;
- includes safe current/replay authorization GUID/status metadata;
- leaves the fresh active authorization unredeemed on conflict;
- allows a retry with a new request GUID to redeem the fresh code exactly once.

## 8. Why Prompt010 Introduced Or Exposed The Conflict

Prompt010 fixed the earlier broad OR lookup. That was correct, but it revealed the WPF lifecycle bug: terminal failed logical submissions were not closed before the operator used a new Pairing Code.

Without request GUID rotation, prompt010's correct API guard would consistently reject a fresh code when the stale request GUID pointed to a different prior authorization.

## 9. Tests And Counts

Builds:

- `dotnet build E:\Project2026\PlatformAppV0\PlatformAppV0.sln` PASS.
- `dotnet build E:\Project2026\1ApiServer\ApiServer01\ApiServer01.csproj` PASS.
- `dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj` PASS.
- `dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj` PASS.

Focused tests:

- `dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0"` PASS: 25 passed.
- `dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"` PASS: 27 passed.

New/maintained proof areas:

- expired terminal response closes logical submission;
- next fresh code uses new request GUID;
- LocalInstallationGuid remains unchanged across rotation;
- timeout/unknown response reuses request GUID;
- HTTP 5xx reuses request GUID;
- fresh code plus stale request GUID returns conflict and does not consume fresh code;
- WPF conflict auto-rotates and retries exactly once;
- retry with new GUID redeems fresh code successfully;
- no infinite retry;
- same request GUID replay returns original authorization;
- no Pairing Code stored in pending checkpoint;
- successful flow continues through protected hello, bootstrap/me, DPAPI and checkpoint;
- prompt label is exactly `prompt011`, and `prompt010` is absent from active WPF title/header/build-info source.

Existing unrelated warnings remain in broader projects.

## 10. Runtime Instance Evidence

Stopped before final restart:

```text
StoppedPids: 55324
```

Final ApiServer:

```text
PID: 53244
StartTimeUtc: 2026-07-31T22:37:37.7604549Z
Path: E:\Project2026\1ApiServer\ApiServer01\bin\Debug\net8.0\ApiServer01.exe
Binary LastWriteTimeUtc: 2026-07-31T22:35:19.4247074Z
Endpoint: http://127.0.0.1:7161
Port owner: 53244
```

Final PlatformAppV0:

```text
PID: 56388
StartTimeUtc: 2026-07-31T22:37:39.8294013Z
Path: E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\bin\Debug\net8.0\OBM.PlatformAppV0.exe
Binary LastWriteTimeUtc: 2026-07-31T22:37:18.3624554Z
Endpoint: https://localhost:7012
Port owner: 56388
```

Runtime probes:

```text
GET http://127.0.0.1:7161/api/platform-v0/readiness
HTTP 200
resultCode = PLATFORM_V0_PHASE1_READY
implementationState = Phase1
```

```text
GET https://localhost:7012/
HTTP 200
```

```text
GET http://127.0.0.1:7161/api/platform-v0/wpf/bootstrap/hello without token
HTTP 401
```

Runtime left running:

```text
ApiServer running on 7161 = true
PlatformAppV0 running on 7012 = true
WPF/NailSalonNet8 process count = 0
```

## 11. Prompt011 Label Proof

Exact constant:

```csharp
public const string CoordinationPromptLabel = "prompt011";
```

Exact title expression:

```csharp
OBM InstallationV0 Phase 1 - prompt011
```

Exact visible UI label:

```text
Build label: prompt011
```

Focused test verifies `prompt010` is absent from active WPF title/header/build-info source.

## 12. No Raw Code Token Secret

Confirmed:

- no raw Pairing Code in report;
- no WpfJwt in report;
- no Authorization header value in report;
- no cookie, Google token, ClientSecret, password, digest, or connection string in report;
- pending checkpoint persists only safe request identity state and never Pairing Code plaintext.

## 13. No DB Phase 2

Confirmed:

- no PostgreSQL/local POS database connection;
- no database creation;
- no migrations;
- no seed;
- no `TblLocalOutbox`;
- no permanent device credential;
- no PlatformEnrollment;
- no Phase 2.

## 14. Physical Retest Steps

1. Open PlatformAppV0 at `https://localhost:7012`.
2. Login/authorize if required.
3. Create one fresh Pairing Code for the intended Tenant/POS1.
4. Start WPF from Visual Studio Debug.
5. Confirm title/header label `prompt011`.
6. Enter the fresh Pairing Code and click redeem once.
7. Expected: no terminal `PAIRING_REDEEM_CONFLICT` visible to the operator, or the conflict is internally recovered by exactly one request-GUID rotation/retry.
8. Confirm protected hello marker:

```text
HELLO_FROM_PLATFORMAPPV0_WPFJWT_PROTECTED_CONTROLLER
```

9. Confirm `/bootstrap/me` identity proof PASS.
10. Confirm DPAPI/checkpoint PASS.
11. Close and reopen WPF with the same ProductRoot.
12. Confirm restart/resume and no second redeem.
13. Do not start Phase 2.

## 15. Coordination Commit

This report is the only coordination artifact intended for prompt011 commit.
