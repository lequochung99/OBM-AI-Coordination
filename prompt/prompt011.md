# Prompt 011 — Sửa PAIRING_REDEEM_CONFLICT do stale RedeemClientRequestGuid

## Physical evidence từ người dùng

Người dùng chạy đúng WPF build mới:

```text
Build label: prompt010
```

Trước khi bấm redeem, UI hiển thị trạng thái request gần nhất/chưa gửi:

```text
PAIRING_CODE_INPUT_PRESENT: false
PAIRING_CODE_INPUT_LENGTH: 0
PAIRING_CODE_REQUEST_SENT: false
```

Sau khi nhập Pairing Code mới và bấm redeem, UI chứng minh WPF đã đọc và gửi input hiện tại:

```text
HTTP 409
resultCode = PAIRING_REDEEM_CONFLICT
PAIRING_CODE_INPUT_PRESENT: true
PAIRING_CODE_INPUT_LENGTH: 14
PAIRING_CODE_REQUEST_SENT: true
```

Không có raw Pairing Code trong evidence.

## Kết luận boundary hiện tại

- WPF đang chạy đúng binary `prompt010`.
- WPF đã đọc current PasswordBox value và đã gửi request.
- Lỗi không còn là cached Pairing Code hoặc request không được gửi.
- API conflict guard đã tìm thấy:
  - Pairing Code hiện tại khớp một PairingAuthorization mới;
  - `RedeemClientRequestGuid` persisted từ attempt cũ khớp một PairingAuthorization khác.
- Vì hai identity trỏ tới hai authorization khác nhau, API trả `PAIRING_REDEEM_CONFLICT` trước khi redeem code mới.
- `LocalInstallationGuid` phải tiếp tục giữ nguyên.
- Root cause cần sửa là lifecycle/idempotency scope của `RedeemClientRequestGuid` khi operator thay Pairing Code sau một terminal failure.

## Mục tiêu

Thiết kế và implement đúng retry/idempotency semantics:

```text
LocalInstallationGuid
= stable machine-side installation identity across all retries/codes

RedeemClientRequestGuid
= stable only within one logical redeem submission/retry group
```

Khi một logical submission kết thúc bằng deterministic terminal failure và operator nhập một Pairing Code khác, WPF phải bắt đầu logical submission mới với `RedeemClientRequestGuid` mới, nhưng giữ nguyên `LocalInstallationGuid`.

Sau correction, fresh Pairing Code phải đi qua:

```text
redeem HTTP 200
WpfJwt issued
protected hello PASS
/bootstrap/me PASS
DPAPI protect/readback PASS
checkpoint write/readback PASS
```

Chưa làm Phase 2 hoặc local database.

## Prompt label bắt buộc

Vì prompt này sửa WPF, label phải đổi thành:

```text
Window title: OBM InstallationV0 Phase 1 - prompt011
Visible UI: Build label: prompt011
```

Dùng một canonical constant duy nhất, ví dụ:

```csharp
public const string CoordinationPromptLabel = "prompt011";
```

Focused test phải chứng minh `prompt010` không còn trong active WPF title/header/build-info source.

## Bắt buộc đọc

```text
prompt/prompt009.md
report/report009.md
prompt/prompt010.md
report/report010.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

## Source boundaries được phép

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0
E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0
E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0.Contracts
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0
E:\Project2026\1ApiServer\ApiServer01.Tests\PlatformAppV0
```

## Retry/idempotency contract bắt buộc

Phân loại outcome thành hai nhóm:

### A. Unknown/transient outcome — reuse same RedeemClientRequestGuid

Ví dụ:

```text
network timeout
connection reset
request canceled after send with unknown server outcome
HTTP 5xx
response body unreadable after request may have reached server
```

Trong các trường hợp này:

- giữ same `RedeemClientRequestGuid`;
- giữ same `LocalInstallationGuid`;
- retry cùng logical operation để API replay/idempotency hoạt động;
- không tạo attempt trùng.

### B. Deterministic terminal outcome — close old logical submission

Ví dụ:

```text
PAIRING_CODE_EXPIRED
PAIRING_CODE_NOT_FOUND
PAIRING_CODE_ALREADY_USED
PAIRING_REDEEM_CONFLICT
other explicit 4xx business result proving no successful redeem for current logical submission
```

Trong các trường hợp này:

- mark pending redeem attempt terminal/closed using safe resultCode;
- không lưu Pairing Code plaintext;
- lần operator nhập/click code tiếp theo phải tạo `RedeemClientRequestGuid` mới;
- giữ nguyên `LocalInstallationGuid`;
- increment safe attempt generation/counter if useful;
- do not reuse old request GUID for a different Pairing Code.

## PAIRING_REDEEM_CONFLICT recovery

Preferred behavior:

1. API conflict guard remains fail-closed; do not remove it.
2. WPF receives `PAIRING_REDEEM_CONFLICT`.
3. WPF atomically closes stale logical attempt.
4. WPF creates a new `RedeemClientRequestGuid` while preserving `LocalInstallationGuid`.
5. WPF may automatically retry the same current in-memory Pairing Code exactly once.
6. No infinite retry loop.
7. If second attempt fails, surface exact resultCode.

If Codex chooses not to auto-retry, UI must clearly state that the stale request identity was reset and operator must click once more. Automatic one-time retry is preferred because the current fresh code is already in memory and conflict proves stale request identity.

## Pending checkpoint requirements

Pending checkpoint may persist only safe data such as:

```text
LocalInstallationGuid
RedeemClientRequestGuid
AttemptGeneration
AttemptState = Active|Terminal|UnknownOutcome
LastSafeResultCode
CreatedAtUtc / UpdatedAtUtc
```

Must not persist:

```text
Pairing Code plaintext
reversible code fingerprint
WpfJwt
Authorization header
```

Atomic write/readback remains required.

## API behavior requirements

- Keep separated lookup by current Pairing Code and replay request GUID.
- Keep `PAIRING_REDEEM_CONFLICT` when both resolve to different records.
- Conflict response should include safe correlation only:
  - correlationId;
  - current-code PairingAuthorizationGuid;
  - replay PairingAuthorizationGuid;
  - statuses/timestamps if useful;
  - never raw code/digest/token.
- Code matched by conflict must remain unredeemed.
- A retry with a new request GUID and same active code should redeem successfully exactly once.
- Old request GUID replay must continue to resolve its original authorization.

## Investigation/evidence matrix

Report must show safe matrix:

```text
Step | LocalInstallationGuid | RedeemClientRequestGuid | AttemptGeneration | Outcome | Next GUID action
```

At minimum include:

1. prior expired/terminal attempt;
2. fresh code sent with stale request GUID -> 409 conflict;
3. request GUID rotation;
4. retry with new request GUID;
5. same LocalInstallationGuid throughout;
6. fresh authorization redeemed once.

GUID values are safe to report; Pairing Code is not.

## UI diagnostics

WPF UI must display safe proof items:

```text
Redeem logical attempt generation
Redeem request GUID rotated: true|false
LocalInstallationGuid preserved: true|false
Conflict auto-retry attempted: true|false
Conflict auto-retry count: 0|1
Final HTTP status
Final resultCode
```

Do not display Pairing Code.

Initial pre-click diagnostics should be labeled clearly as current/last request state so `INPUT_PRESENT=false` before click is not mistaken for a defect.

## Preserve existing Phase 1 proofs

Do not regress:

```text
prompt008 JWT expiration second precision correction
protected GetHelloWorld WpfJwt proof
HELLO_FROM_PLATFORMAPPV0_WPFJWT_PROTECTED_CONTROLLER marker
/bootstrap/me exact identity verification
DPAPI protect/readback
atomic ApiAuthorized checkpoint
restart/resume without second redeem
local database touched = false
```

## Tests bắt buộc

Add/maintain tests for:

1. expired code terminal response closes logical submission;
2. next fresh code uses new RedeemClientRequestGuid;
3. LocalInstallationGuid remains unchanged across request-GUID rotation;
4. network timeout/unknown response reuses same request GUID;
5. HTTP 5xx retry reuses same request GUID;
6. fresh code + stale request GUID returns conflict and does not consume fresh code;
7. WPF conflict auto-rotates and retries exactly once;
8. retry with new GUID redeems fresh code successfully;
9. no infinite retry;
10. same request GUID replay still returns original authorization;
11. no Pairing Code stored in pending checkpoint;
12. successful flow continues through protected hello, bootstrap/me, DPAPI and checkpoint;
13. prompt label is exactly prompt011 and prompt010 is absent.

Build/test commands:

```text
dotnet build E:\Project2026\PlatformAppV0\PlatformAppV0.sln

dotnet build E:\Project2026\1ApiServer\ApiServer01\ApiServer01.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0"

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"
```

## Runtime handoff

At task end:

1. Stop only relevant stale ApiServer/PlatformAppV0/WPF processes.
2. Build/test latest source.
3. Restart ApiServer on port 7161 from latest Debug output.
4. Restart PlatformAppV0 on port 7012 from latest Debug output.
5. Prove PID, start UTC, binary LastWriteTimeUtc, executable path and port owner.
6. Readiness must return HTTP 200 / `PLATFORM_V0_PHASE1_READY`.
7. PlatformAppV0 root must return HTTP 200.
8. Protected hello without token must return HTTP 401.
9. Leave ApiServer and PlatformAppV0 running.
10. Do not leave WPF running; user launches WPF from Visual Studio.

## Physical retest

1. Open PlatformAppV0 and create one fresh Pairing Code.
2. Start WPF from Visual Studio.
3. Confirm title/header label `prompt011`.
4. Enter fresh code and click redeem once.
5. Expected: no terminal `PAIRING_REDEEM_CONFLICT` visible to operator, or conflict is internally recovered by exactly one request-GUID rotation/retry.
6. Confirm protected hello marker.
7. Confirm bootstrap identity, DPAPI and checkpoint PASS.
8. Close/reopen same ProductRoot.
9. Confirm restart/resume and no second redeem.
10. Do not start Phase 2.

## Git/security guardrails

- No `git add .` or `git add -A`.
- No reset/clean/stash/checkout/restore.
- No source commit/push.
- Only commit `report/report011.md` to coordination repository.
- No Pairing Code, WpfJwt, cookie, secret, digest or connection string in report.
- No PostgreSQL/local DB/Phase 2 work.

## Report 011 — bắt buộc 100% chi tiết

Create:

```text
report/report011.md
```

Must include:

1. Verdict.
2. Exact physical evidence from prompt010.
3. Exact root cause.
4. Idempotency state-machine definition.
5. Safe GUID/generation/outcome matrix.
6. Exact files/methods changed.
7. Exact correction.
8. Why prompt010 introduced/exposed conflict.
9. Tests and counts.
10. Runtime instance evidence.
11. Prompt011 label proof.
12. Confirmation no raw code/token/secret.
13. Confirmation no DB/Phase 2.
14. Exact physical retest steps.
15. Coordination commit SHA.

## Verdict hợp lệ

If corrected and ready for user test:

```text
WPF_REDEEM_REQUEST_IDENTITY_ROTATION_READY_FOR_USER_RETEST
```

If full physical Phase 1 including restart/resume passes:

```text
PHASE1_WPF_API_AUTHORIZATION_AND_MACHINE_PERSISTENCE_PASS_DATABASE_NOT_STARTED
```

If blocked:

```text
BLOCKED_WPF_PAIRING_REDEEM_CONFLICT
```
