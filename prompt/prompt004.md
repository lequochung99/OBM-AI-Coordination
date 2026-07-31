# Prompt 004 — Sửa API 401 sau Platform administrator authorization exchange

## Runtime evidence từ người dùng

Sau khi chạy prompt003, người dùng đăng nhập Google và bấm `Authorize Platform Administrator`.

URL sau redirect:

```text
https://localhost:7012/?admin_authorize_result=HTTP_401_PLATFORM_ADMIN_NOT_AUTHENTICATED
```

UI đồng thời hiển thị:

```text
Google login state: PASS
Platform administrator authorization state: Pending
Approved identity summary: Pending administrator authorization
Create/Select Tenant and POS1: disabled
Create Pairing Code: disabled
```

## Kết luận boundary đã biết

Result code có prefix `HTTP_401_`, vì vậy:

- `/platform-admin/authorize` đã được gọi;
- PlatformAppV0 đã đi đến bước gọi API exchange;
- API đã trả HTTP 401 với result code `PLATFORM_ADMIN_NOT_AUTHENTICATED`;
- blocker hiện tại nằm trong contract/request/authentication giữa PlatformAppV0 và PlatformAppV0 API, không còn là silent UI hydration;
- Tenant/POS và Pairing Code chưa được bắt đầu.

Không được tiếp tục sửa UI hydration như root cause chính nếu không có evidence mới.

## Mục tiêu

Xác định và sửa chính xác vì sao canonical PlatformAppV0 API administrator exchange trả:

```text
HTTP 401
PLATFORM_ADMIN_NOT_AUTHENTICATED
```

Flow hợp lệ phải đạt:

```text
Google login state = PASS
Authorize Platform Administrator
API exchange = HTTP 200 success contract
Platform administrator authorization state = PASS
Approved identity summary = populated
Create/Select Tenant and POS1 = enabled
```

Sau đó chỉ kiểm tra Tenant/POS action trả success hoặc explicit backend result. Chưa làm Pairing Code, WPF redeem hoặc Phase 2.

## Tài liệu/report bắt buộc đọc

```text
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
contracts/v001/WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
prompt/prompt002.md
report/report002.md
prompt/prompt003.md
report/report003.md
```

## Source boundaries được phép

```text
E:\Project2026\PlatformAppV0
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0
E:\Project2026\1ApiServer\ApiServer01.Tests\PlatformAppV0
```

## Điều tra bắt buộc

Trace exact request từ PlatformAppV0 đến API:

```text
/platform-admin/authorize
-> resolved API base URL
-> exact HTTP method/path
-> request headers/body contract
-> API endpoint routing
-> API authentication/authorization scheme
-> Google ID-token validation
-> Platform administrator allowlist/bootstrap lookup
-> API resultCode
```

Phải xác minh bằng evidence cụ thể:

1. Exact API endpoint URL được gọi.
2. Exact method và request contract field names.
3. Google `id_token` có được gửi theo body/header đúng contract hay không; chỉ ghi presence, length/range hoặc hash an toàn, không ghi raw token.
4. API endpoint có `[Authorize]` hoặc policy nào khiến request bị chặn trước controller không.
5. `PLATFORM_ADMIN_NOT_AUTHENTICATED` được phát sinh tại middleware, controller hay service nào.
6. API đang mong đợi Google ID token exchange anonymous, Platform cookie, bearer token, provider headers hay một contract khác.
7. Token validation issuer/audience/client-id có khớp OIDC client đang dùng tại PlatformAppV0 hay không.
8. API runtime có đúng config/client ID/provider metadata cần thiết hay không.
9. Google token có `sub`, `email`, `email_verified`, issuer và audience hợp lệ hay không; chỉ báo claim presence/masked identity.
10. Platform administrator bootstrap/allowlist có chứa đúng authenticated Google subject/email hay không.
11. Có mismatch claim name, endpoint version, route prefix hoặc API base URL không.
12. Có stale API binary/config/runtime process không.
13. API error body có bị PlatformAppV0 map sai thành `PLATFORM_ADMIN_NOT_AUTHENTICATED` không.
14. Nếu API exchange endpoint phải anonymous vì Google ID token chính là credential, chứng minh và cấu hình rõ; không được hạ security tùy tiện.
15. Nếu endpoint phải dùng một server-to-server auth mechanism khác, implement đúng canonical contract và tests.

## Phân biệt các failure bắt buộc

Diagnostics an toàn phải phân biệt tối thiểu:

```text
ADMIN_EXCHANGE_ROUTE_NOT_FOUND
ADMIN_EXCHANGE_BLOCKED_BY_AUTH_MIDDLEWARE
GOOGLE_ID_TOKEN_NOT_SENT
GOOGLE_ID_TOKEN_INVALID_ISSUER
GOOGLE_ID_TOKEN_INVALID_AUDIENCE
GOOGLE_ID_TOKEN_EXPIRED
GOOGLE_EMAIL_NOT_VERIFIED
PLATFORM_ADMIN_BOOTSTRAP_IDENTITY_NOT_FOUND
PLATFORM_ADMIN_IDENTITY_MISMATCH
ADMIN_EXCHANGE_CONTRACT_MISMATCH
ADMIN_EXCHANGE_SUCCESS
```

Không log raw token, cookie, authorization header, client secret hoặc full sensitive claims.

## Yêu cầu sửa

- Sửa root cause tối thiểu ở đúng boundary.
- Không hard-code PASS.
- Không mock approved identity trong runtime.
- Không coi Google login local là Platform administrator approval nếu API chưa xác minh.
- API success phải trả approved identity contract rõ ràng.
- PlatformAppV0 chỉ persist approved claims sau API HTTP 200 và success contract hợp lệ.
- 401/403 phải hiển thị result code cụ thể.
- Không enable Tenant/POS nếu API authorization chưa PASS.
- Không tạo Pairing Code trong task này.

## Build/test bắt buộc

Phải dừng đúng stale process nếu cần và chạy build-enabled:

```text
dotnet build E:\Project2026\PlatformAppV0\PlatformAppV0.sln

dotnet test E:\Project2026\PlatformAppV0\PlatformAppV0.sln --no-build

dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0"
```

Focused tests phải bao gồm:

- valid Google ID-token exchange returns HTTP 200 approved identity;
- missing token fails explicit;
- invalid issuer fails explicit;
- invalid audience fails explicit;
- expired token fails explicit;
- unapproved administrator identity fails explicit;
- approved identity succeeds;
- auth middleware does not incorrectly block the intended exchange contract;
- PlatformAppV0 serializes request exactly as API expects;
- 401 error mapping preserves API result code;
- success response enables approved state only after validation.

Không được dùng chỉ `--no-build` cho ApiServer proof.

## Runtime evidence bắt buộc

Restart canonical runtimes và ghi PID/port/binary path an toàn.

Tối thiểu prove:

```text
GET API readiness = 200 PLATFORM_V0_PHASE1_READY
GET PlatformAppV0 root = 200
Admin exchange synthetic/focused integration proof = expected status/result
```

Nếu Codex không thể thực hiện interactive Google login, chuẩn bị cho user retest. Nhưng trước khi giao lại phải xác định chính xác nơi phát sinh 401 và có test/integration evidence cho correction.

## Physical user acceptance

Sau correction, user phải thấy:

```text
A. Google login = PASS
B. Click Authorize Platform Administrator
C. URL không còn admin_authorize_result=HTTP_401_PLATFORM_ADMIN_NOT_AUTHENTICATED
D. Administrator authorization = PASS
E. Approved identity populated
F. Create/Select Tenant and POS1 enabled
G. Tenant/POS click returns success hoặc explicit backend failure
```

Nếu D–F chưa có physical proof, verdict tối đa là READY_FOR_USER_RETEST.

## Git safety

`E:\Project2026` vẫn là shared dirty parent repo không có source remote.

- Không `git add .` hoặc `git add -A`.
- Không reset/clean/stash/checkout/restore.
- Không push source.
- Không commit source nếu không isolate được exact task files.
- Liệt kê exact files touched và pre-existing provenance.
- Chỉ commit report vào coordination repository.

## Report 004 — bắt buộc 100% chi tiết

Tạo:

```text
report/report004.md
```

Report phải có:

1. Verdict.
2. Reproduction của exact URL/result code.
3. Exact request/response timeline.
4. Exact location phát sinh `PLATFORM_ADMIN_NOT_AUTHENTICATED`.
5. Root cause.
6. Vì sao prompt003 chưa giải quyết API 401.
7. Exact endpoint/method/request contract.
8. Authentication/authorization scheme analysis.
9. Google token validation analysis an toàn.
10. Platform admin bootstrap/allowlist analysis.
11. Exact files changed.
12. Exact correction.
13. Diagnostics/result codes.
14. Build commands/results.
15. Test commands/counts.
16. Process/PID/port/binary evidence.
17. Acceptance matrix A–G.
18. Remaining blockers/risks/unverified.
19. Source Git state/no push confirmation.
20. Exact user retest steps.
21. Coordination commit SHA trong final response.

## Verdict hợp lệ

Nếu correction hoàn tất và chờ interactive retest:

```text
PLATFORMAPPV0_API_ADMIN_EXCHANGE_401_CORRECTION_READY_FOR_USER_RETEST
```

Nếu physical flow A–G PASS:

```text
PLATFORMAPPV0_ADMIN_AUTHORIZATION_AND_TENANT_POS1_GATE_PASS
```

Nếu chưa sửa được:

```text
BLOCKED_PLATFORMAPPV0_API_ADMIN_EXCHANGE_401
```

Không được tuyên bố PASS chỉ vì unit tests PASS.
