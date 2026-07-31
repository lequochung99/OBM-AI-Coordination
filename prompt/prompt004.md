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
- blocker hiện tại nằm trong contract/request/authentication giữa PlatformAppV0 và PlatformAppV0 API;
- lỗi không còn được xem là silent UI hydration nếu không có evidence mới;
- Tenant/POS và Pairing Code chưa được bắt đầu.

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

Chỉ sửa file cần thiết trong các boundary trên.

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
-> Platform administrator bootstrap/allowlist lookup
-> API resultCode
```

Phải xác minh bằng evidence cụ thể:

1. Exact API endpoint URL được gọi.
2. Exact method và request contract field names.
3. Google `id_token` có được gửi theo body/header đúng contract hay không; chỉ ghi presence, độ dài an toàn hoặc hash rút gọn, không ghi raw token.
4. API endpoint có `[Authorize]`, fallback policy hoặc middleware nào chặn request trước controller hay không.
5. `PLATFORM_ADMIN_NOT_AUTHENTICATED` được phát sinh tại middleware, controller, endpoint mapper hay service nào.
6. API đang mong đợi Google ID token exchange anonymous, Platform cookie, bearer token, provider headers hay contract khác.
7. Token validation issuer/audience/client-id có khớp OIDC client của PlatformAppV0 hay không.
8. API runtime có đúng config/client ID/provider metadata cần thiết hay không.
9. Google token có `sub`, `email`, `email_verified`, issuer và audience hợp lệ hay không; chỉ báo claim presence và identity đã mask.
10. Platform administrator bootstrap/allowlist có chứa đúng authenticated Google subject/email hay không.
11. Có mismatch claim name, endpoint version, route prefix, HTTP method hoặc API base URL không.
12. Có stale API binary/config/runtime process không.
13. API error body có bị PlatformAppV0 map sai thành `PLATFORM_ADMIN_NOT_AUTHENTICATED` không.
14. Nếu exchange endpoint phải anonymous vì Google ID token chính là credential, chứng minh bằng threat model/tests và cấu hình rõ; không hạ security tùy tiện.
15. Nếu endpoint phải dùng server-to-server authentication khác, implement đúng canonical contract và tests.

## Yêu cầu bằng chứng C# đầy đủ để ChatGPT review

Repository coordination hiện là Public, vì vậy **không được chép toàn bộ ApiServer01 hoặc toàn bộ source OBM**. Tuy nhiên `report/report004.md` bắt buộc phải chứa **toàn bộ code C# liên quan trực tiếp đến auth exchange path**, đã loại bỏ secret, để ChatGPT có thể đọc và xác định lỗi.

Với mỗi file/method liên quan, report phải ghi:

```text
Full local path
Project-relative path
Class/type name
Method/endpoint name
Approximate line range
Role in request chain
```

Sau đó chèn fenced code block `csharp` chứa code đầy đủ của method/class liên quan, không chỉ vài dòng rời rạc.

Tối thiểu phải bao gồm đầy đủ các phần sau nếu chúng tồn tại:

1. PlatformAppV0 `/platform-admin/authorize` endpoint hoặc handler.
2. Code tạo request DTO và gọi HttpClient/API exchange.
3. Request/response/problem DTO/record/class của admin exchange.
4. API controller/minimal endpoint nhận admin exchange.
5. `[Authorize]`, `[AllowAnonymous]`, policy hoặc endpoint metadata gắn trên exchange endpoint.
6. API authentication/authorization registration trong `Program.cs` hoặc extension method liên quan.
7. Google ID-token validator/service method.
8. Platform administrator bootstrap/allowlist lookup/service method.
9. Exact branch trả HTTP 401 và `PLATFORM_ADMIN_NOT_AUTHENTICATED`.
10. Result-code/problem response mapper liên quan.
11. DI registrations của các service trên.
12. Focused tests chứng minh request contract, middleware behavior, invalid/valid token và approved identity.

Nếu một file rất lớn, không chép cả file. Chép đầy đủ:

- `using` cần thiết để hiểu code;
- class declaration và constructor/dependencies;
- toàn bộ method liên quan;
- private helper mà method gọi trực tiếp;
- constants/claim names/result codes mà flow dùng.

Không được chép:

- raw token;
- client secret;
- cookie;
- authorization header value;
- connection string;
- environment secret;
- customer data;
- unrelated business source.

Mọi value nhạy cảm phải thay bằng:

```text
<REDACTED>
<CONFIGURED_CLIENT_ID>
<SECRET_FROM_LOCAL_STORE>
```

Report phải có thêm hai phần:

### C# before/after correction

- Trích đầy đủ method trước sửa nếu có thể lấy từ pre-edit snapshot/diff.
- Trích đầy đủ method sau sửa.
- Giải thích từng branch thay đổi và branch nào đã tạo 401.

### Exact failure location

Ghi theo định dạng:

```text
Failure file: <path>
Failure type/method: <type.method>
Failure line/range: <line-range>
Failure condition: <boolean/claim/policy condition>
Returned HTTP status: 401
Returned resultCode: PLATFORM_ADMIN_NOT_AUTHENTICATED
Why the condition was true: <evidence>
```

Nếu Codex không thể cung cấp code đầy đủ an toàn cho một mục, phải ghi rõ lý do và cung cấp path + line range + sanitized pseudocode. Không được bỏ qua im lặng.

## Phân biệt failure bắt buộc

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
- 401/403 phải giữ nguyên result code cụ thể.
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
- auth middleware does not incorrectly block intended exchange contract;
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

Nếu Codex không thể thực hiện interactive Google login, chuẩn bị cho user retest. Trước khi giao lại phải xác định chính xác nơi phát sinh 401 và có test/integration evidence cho correction.

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
2. Reproduction exact URL/result code.
3. Exact request/response timeline.
4. Exact location phát sinh `PLATFORM_ADMIN_NOT_AUTHENTICATED`.
5. Root cause.
6. Vì sao prompt003 chưa giải quyết API 401.
7. Exact endpoint/method/request contract.
8. Authentication/authorization scheme analysis.
9. Google token validation analysis an toàn.
10. Platform admin bootstrap/allowlist analysis.
11. Toàn bộ relevant C# evidence theo yêu cầu ở trên.
12. C# before/after correction.
13. Exact files changed.
14. Exact correction.
15. Diagnostics/result codes.
16. Build commands/results.
17. Test commands/counts.
18. Process/PID/port/binary evidence.
19. Acceptance matrix A–G.
20. Remaining blockers/risks/unverified.
21. Source Git state/no push confirmation.
22. Exact user retest steps.
23. Coordination commit SHA trong final response.

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
