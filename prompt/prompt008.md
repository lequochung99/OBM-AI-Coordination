# Prompt 008 — Sửa WPF_IDENTITY_MISMATCH sau redeem HTTP 200

## Physical runtime evidence từ người dùng

Người dùng chạy WPF InstallationV0 Phase 1 bằng Visual Studio và nhập Pairing Code hợp lệ.

Kết quả thực tế:

```text
Pairing Code redeemed: HTTP 200
WpfJwt received: WPF_IDENTITY_MISMATCH
Protected API call succeeded: WPF_BOOTSTRAP_IDENTITY_VERIFIED
WpfJwt scope verified: wpf.install.bootstrap
Tenant identity verified: OBMDEVV0
POS station identity verified: POS1
PosGuid verified: PASS
Local installation identity verified: PASS
Installation attempt verified: PENDING / mismatch
Credential protected locally: not started
Checkpoint persisted: not started
Restart/resume: not started
Local database touched = false
```

UI hiển thị:

```text
Identity mismatch. Phase 1 is blocked.
```

## Kết luận boundary hiện tại

- Pairing Code redeem endpoint đã trả HTTP 200.
- WpfJwt đã đủ hợp lệ để gọi protected `/api/platform-v0/wpf/bootstrap/me`.
- `/bootstrap/me` trả success contract.
- Scope, Tenant, POS, PosGuid và LocalInstallationGuid đều khớp.
- Mismatch nằm trong một hoặc nhiều field còn lại, nhiều khả năng thuộc installation-attempt identity hoặc field mapping liên quan.
- Không được đoán field. Phải in safe comparison matrix từ redeem response và bootstrap/me response.

## Mục tiêu

Xác định và sửa chính xác field nào khác nhau giữa:

```text
POST /api/platform-v0/wpf/pairing/redeem response
GET /api/platform-v0/wpf/bootstrap/me response
```

Sau correction, cùng một physical flow phải đạt:

```text
Pairing Code redeemed = PASS
WpfJwt received = PASS
Protected API call succeeded = PASS
Scope verified = PASS
Tenant verified = PASS
POS verified = PASS
PosGuid verified = PASS
InstallationAttemptGuid verified = PASS
AttemptVersion verified = PASS
LocalInstallationGuid verified = PASS
Credential expiration verified = PASS
DPAPI protect/readback = PASS
Checkpoint write/readback = PASS
LocalDatabaseTouched = false
```

Chưa làm Phase 2.

## Bắt buộc đọc

```text
prompt/prompt007.md
report/report007.md
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

## Điều tra bắt buộc

Trace và compare từng field, không dùng một generic boolean:

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

Report phải có safe comparison matrix:

```text
Field | Redeem value | Bootstrap/me value | Equal | Notes
```

Đối với GUID/code/name/version có thể ghi full value nếu không phải secret.
Đối với token không được ghi raw value.
Đối với expiration ghi UTC timestamp.

Phải xác định:

1. Exact field mismatch.
2. Field đó được tạo ở redeem service/controller nào.
3. Field đó được lấy từ JWT claim hay persisted state trong `/bootstrap/me`.
4. Claim name có đồng bộ không.
5. Có mismatch kiểu dữ liệu/string casing/GUID format/time precision không.
6. `InstallationAttemptGuid` và `AttemptVersion` có được persist và issue từ cùng một record không.
7. `/bootstrap/me` có đang tạo mới hoặc đọc nhầm attempt không.
8. Redeem response có map nhầm `pairingAuthorizationGuid`, `installationAttemptGuid`, `localInstallationGuid` hoặc `posStationId` không.
9. JWT claims có dùng đúng canonical names không.
10. WPF DTO có deserialize đúng JSON property names không.
11. WPF comparison có nhầm field hoặc compare default value không.
12. Có stale ApiServer/WPF binary không.

## Safe diagnostics

Có thể thêm diagnostic UI/result model cho từng mismatch, ví dụ:

```text
INSTALLATION_ATTEMPT_GUID_MISMATCH
ATTEMPT_VERSION_MISMATCH
CREDENTIAL_EXPIRATION_MISMATCH
CONTRACT_VERSION_MISMATCH
```

Không log Pairing Code hoặc WpfJwt.

## Yêu cầu sửa

- Sửa root cause tối thiểu.
- Không bỏ qua comparison.
- Không đổi mismatch thành PASS giả.
- Không dùng wildcard/default acceptance.
- Redeem response, JWT claims, persisted attempt state và `/bootstrap/me` phải cùng một canonical identity.
- WPF chỉ DPAPI-protect token sau khi toàn bộ identity comparison PASS.
- Nếu mismatch xảy ra, không để lại protected credential hoặc ApiAuthorized checkpoint giả.
- Pairing Code one-time behavior phải giữ nguyên.
- Retry/idempotency theo pending `redeemClientRequestGuid` phải giữ nguyên.

## Build/test bắt buộc

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"

dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0"
```

Bổ sung tests cho:

- exact redeem/bootstrap identity equality;
- installation attempt GUID mismatch detection;
- attempt version mismatch detection;
- local installation mismatch detection;
- expiration mismatch handling;
- DTO JSON property mapping;
- no credential/checkpoint persistence on mismatch;
- successful full identity comparison proceeds to DPAPI/checkpoint.

## Runtime/Visual Studio handoff

Cuối task:

- dừng stale ApiServer/WPF process gây lock;
- build/test xong;
- không để WPF instance chạy nền;
- không tự redeem Pairing Code thật;
- giao lại để người dùng chạy ApiServer và WPF bằng Visual Studio Debug thủ công;
- không yêu cầu user retest nếu chưa chứng minh source/build path mới nhất.

## Pairing Code retest note

Pairing Code cũ có thể đã bị redeem một lần. User có thể cần tạo Pairing Code mới trên PlatformAppV0 trước retest.

Không ghi code vào report.

## Git safety

- Không `git add .` hoặc `git add -A`.
- Không reset/clean/stash/checkout/restore.
- Không source push.
- Chỉ commit `report/report008.md` vào coordination repository.

## Report 008 — bắt buộc 100% chi tiết

Tạo:

```text
report/report008.md
```

Report phải có:

1. Verdict.
2. Physical symptom reproduction.
3. Full safe comparison matrix.
4. Exact mismatched field(s).
5. Exact source location tạo mỗi conflicting value.
6. Root cause.
7. Vì sao prompt007 tests không bắt được lỗi.
8. Exact files changed.
9. Exact correction.
10. Focused tests added.
11. Build/test commands và counts.
12. Process/binary handoff status.
13. Confirmation no Pairing Code/token logged.
14. Confirmation no DB/Phase 2.
15. Exact user retest steps.
16. Coordination commit SHA trong final response.

## Verdict hợp lệ

Nếu correction hoàn tất và chờ physical retest:

```text
WPF_INSTALLATIONV0_IDENTITY_MISMATCH_CORRECTION_READY_FOR_USER_RETEST
```

Nếu full physical Phase 1 proof PASS gồm restart/resume:

```text
PHASE1_WPF_API_AUTHORIZATION_AND_MACHINE_PERSISTENCE_PASS_DATABASE_NOT_STARTED
```

Nếu chưa xác định/sửa được:

```text
BLOCKED_WPF_INSTALLATIONV0_IDENTITY_MISMATCH
```
