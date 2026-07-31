# Prompt 007 — WPF InstallationV0 redeem Pairing Code và verify API access

## Trạng thái đã PASS trước task này

PlatformAppV0 physical flow đã đạt:

```text
Administrator authorization = PASS
Approved identity = populated
Tenant/POS1 = PASS
Tenant Code = OBMDEVV0
POS name = POS1
Slot number = 1
Pairing Code = created and displayed once
```

Pairing Code plaintext không được ghi vào report hoặc GitHub.

## Mục tiêu

Quay lại WPF `InstallationV0` và hoàn tất phần cốt lõi của **Phase 1**:

```text
WPF nhập Pairing Code
-> POST /api/platform-v0/wpf/pairing/redeem
-> nhận WpfJwt
-> dùng WpfJwt gọi GET /api/platform-v0/wpf/bootstrap/me
-> verify exact Tenant/POS/InstallationAttempt identity
-> bảo vệ credential local
-> persist/readback checkpoint
-> chuẩn bị restart/resume proof
```

Task này chỉ làm Phase 1. Tuyệt đối không làm Phase 2 và không chạm local POS database.

## Canonical contract bắt buộc đọc

```text
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
contracts/v001/WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Các mục bắt buộc tuân thủ:

- Phase 1 mục tiêu và acceptance proof;
- WPF InstallationV0 UI;
- redeem request/response contract;
- WpfJwt scheme/policy;
- protected `/bootstrap/me` contract;
- DPAPI/secret storage;
- checkpoint atomic persistence;
- restart/resume without redeem;
- Phase 1 không được chạm database.

## Source boundaries được phép

### WPF

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0
```

### ApiServer PlatformAppV0 Phase 1 nếu cần correction tối thiểu

```text
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0
E:\Project2026\1ApiServer\ApiServer01.Tests\PlatformAppV0
```

Không dùng code installation cũ ngoài `InstallationV0`.

## Visual Studio canonical lane

- WPF code phải nằm trong solution/project Visual Studio canonical.
- Build bằng Visual Studio Build Solution hoặc exact equivalent `dotnet build` trên canonical solution/project.
- Tests phải được Test Explorer phát hiện.
- Launch bằng canonical startup project/launchSettings.
- Cuối task không tự giữ instance WPF chạy nền nếu điều đó làm người dùng có thể test nhầm binary.
- Chuẩn bị source/build sạch, dừng instance cũ và giao lại để người dùng chạy Debug thủ công bằng Visual Studio.

## WPF UI tối thiểu

WPF InstallationV0 phải có màn hình rõ ràng:

```text
OBM-POS New Installation

API Base URL: <approved configured value>
Pairing Code: [________________]

[ Redeem and Verify API Access ]
```

Hiển thị riêng từng proof:

```text
Pairing Code redeemed
WpfJwt received
Protected API call succeeded
WpfJwt scope verified
Tenant identity verified
POS station identity verified
PosGuid verified
Installation attempt verified
Local installation identity verified
Credential protected locally
Protected credential readback passed
Machine checkpoint persisted
Checkpoint readback passed
Restart/resume verified
Local database touched = false
```

Không được dùng một status chung chung thay thế các proof này.

## API Base URL

- Không hard-code production tenant/POS identity.
- Dùng approved Development Phase 1 API base URL từ configuration/launch profile.
- Runtime hiện tại expected ApiServer port là `7161`, nhưng code phải đọc config, không hard-code port trong business logic.

## Redeem request contract

```http
POST /api/platform-v0/wpf/pairing/redeem
```

Request tối thiểu:

```json
{
  "pairingCode": "<one-time-code>",
  "clientRequestGuid": "<persistent-guid>",
  "localInstallationGuid": "<machine-installation-guid>",
  "appVersion": "<safe-version>",
  "contractVersion": 1
}
```

Quy tắc:

- `ClientRequestGuid` phải persist trước request và reuse khi retry.
- `LocalInstallationGuid` phải persist stable cho cùng ProductRoot/machine installation.
- Timeout/retry không được tạo installation attempt thứ hai.
- Pairing Code không log, không screenshot evidence, không lưu plaintext sau success.

## Redeem response validation

HTTP 200 chỉ được coi là success khi có đủ:

```text
success = true
resultCode = WPF_JWT_ISSUED
credentialType = Bearer
wpfJwt present
scope = wpf.install.bootstrap
TenantGuid present
PosStationId present
PosGuid present
InstallationAttemptGuid present
AttemptVersion valid
LocalInstallationGuid exact match
contractVersion = 1
expiresAtUtc valid
```

Thiếu hoặc mismatch bất kỳ field nào phải fail closed.

## Protected API verification

Sau redeem, chính process WPF phải gọi:

```http
GET /api/platform-v0/wpf/bootstrap/me
Authorization: Bearer <WpfJwt>
```

Phải yêu cầu API dùng canonical scheme/policy:

```text
Authentication scheme: WpfJwt
Authorization policy: WpfInstallBootstrap
```

WPF phải so sánh response `/bootstrap/me` với redeem response theo exact fields:

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
credentialExpiresAtUtc
contractVersion
```

Mọi mismatch đều fail closed và không persist `ApiAuthorized` checkpoint.

## Secure local persistence

Chỉ sau `/bootstrap/me` PASS:

1. Protect raw `WpfJwt` bằng DPAPI hoặc approved InstallationV0 secret service.
2. Read protected credential back.
3. Validate readback token/identity safely.
4. Write checkpoint atomically.
5. Read checkpoint back.
6. Compare exact identity.
7. Mark Phase 1 authorized.

Raw token không được xuất hiện trong:

- checkpoint JSON;
- logs;
- report;
- screenshots;
- command line;
- exceptions;
- source code.

Checkpoint chỉ chứa protected reference, ví dụ:

```text
bootstrapCredentialReference = secrets/wpf-bootstrap-v1
```

## ProductRoot and restart/resume

- Dùng một explicit Development ProductRoot an toàn.
- Không dùng production ProductRoot.
- Sau first PASS, restart cùng WPF/cùng ProductRoot phải:
  - đọc checkpoint;
  - unprotect token;
  - verify checkpoint/token identity;
  - nếu API reachable, gọi lại `/bootstrap/me`;
  - không hỏi Pairing Code lại;
  - không redeem lần hai;
  - không tạo installation attempt thứ hai.

Nếu full physical restart proof cần operator, chuẩn bị app để người dùng chạy Debug/restart thủ công và báo exact steps.

## Phase 1 tuyệt đối không được làm

```text
No PostgreSQL connection
No local database creation
No migration
No seed
No TblLocalOutbox
No employees/services/customers/giftcards
No permanent device credential
No PlatformEnrollment
No heartbeat
No automatic Phase 2
```

Phải có code/test guard chứng minh `LocalDatabaseTouched = false`.

## Điều tra và reuse hiện trạng

Trước sửa:

- audit toàn bộ `InstallationV0` hiện có;
- xác định UI, services, contracts, checkpoint/secret abstractions đã tồn tại;
- reuse code mới trong chính `InstallationV0` nếu đúng contract;
- không kéo code từ installation folders cũ;
- không phá pre-existing source changes ngoài task.

## Build/tests bắt buộc

Tối thiểu:

```text
dotnet build <canonical WPF solution/project containing InstallationV0>
dotnet test <canonical WPF tests> --filter InstallationV0 or equivalent
dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0"
```

Tests bắt buộc bao phủ:

- redeem request serialization;
- persistent `ClientRequestGuid` retry reuse;
- persistent `LocalInstallationGuid`;
- invalid/expired/reused Pairing Code failure;
- HTTP 200 incomplete contract rejected;
- WpfJwt missing rejected;
- scope mismatch rejected;
- every identity mismatch rejected;
- `/bootstrap/me` 401/403 rejected;
- DPAPI protect/readback;
- checkpoint atomic write/readback;
- raw token absent from checkpoint/log;
- restart/resume without redeem;
- no database dependency/reference/activity in Phase 1.

## Runtime/physical proof

Codex may use a disposable Pairing Code only if operator explicitly supplies one during execution. Do not copy the code into report.

If no valid Pairing Code is available to Codex, verdict tối đa là ready for user physical test.

Người dùng sẽ chạy WPF bằng Visual Studio Debug thủ công để bảo đảm binary mới nhất.

Exact user test flow cần chuẩn bị:

```text
1. Start ApiServer Phase 1 from Visual Studio.
2. Start WPF InstallationV0 from Visual Studio Debug.
3. Enter current one-time Pairing Code manually.
4. Click Redeem and Verify API Access.
5. Verify every proof item individually.
6. Close WPF.
7. Start the same WPF/same ProductRoot again.
8. Verify resume without Pairing Code and without second redeem.
```

## Git safety

`E:\Project2026` là shared dirty parent repo, chưa có source remote.

- Không `git add .` hoặc `git add -A`.
- Không reset/clean/stash/checkout/restore.
- Không push source.
- Không commit source nếu không isolate được exact task files.
- Liệt kê exact files touched và provenance.
- Chỉ commit report vào coordination repository.

## Report 007

Tạo đúng:

```text
report/report007.md
```

Report lần đầu tối thiểu 80%, nhưng phải có:

1. Verdict.
2. InstallationV0 pre-audit.
3. Exact architecture/call chain.
4. Files changed.
5. Redeem request/response contract implementation.
6. `/bootstrap/me` implementation and comparison.
7. Secret protection/checkpoint implementation.
8. Restart/resume implementation.
9. Database non-touch proof.
10. Build/test commands and counts.
11. Runtime evidence available.
12. Physical proof items PASS/PENDING.
13. Source Git/no push confirmation.
14. Exact Visual Studio user test steps.
15. Remaining blockers/risks.
16. Coordination commit SHA in final response.

Nếu task phải chạy lại mà chưa PASS, report kế tiếp cho cùng lỗi phải 100% chi tiết.

## Verdict hợp lệ

Nếu implementation/build/tests hoàn tất, chờ người dùng nhập Pairing Code và restart test:

```text
WPF_INSTALLATIONV0_PHASE1_REDEEM_VERIFY_READY_FOR_USER_PHYSICAL_TEST
```

Nếu full physical proof và restart/resume PASS:

```text
PHASE1_WPF_API_AUTHORIZATION_AND_MACHINE_PERSISTENCE_PASS_DATABASE_NOT_STARTED
```

Nếu blocked:

```text
BLOCKED_WPF_INSTALLATIONV0_PHASE1_REDEEM_VERIFY
```
