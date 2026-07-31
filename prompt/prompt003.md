# Prompt 003 — Sửa runtime failure sau khi bấm Authorize Platform Administrator

## Trạng thái thực tế từ người dùng

Prompt 002 chưa đạt runtime acceptance.

Người dùng đã chạy PlatformAppV0 mới và thực hiện:

1. Google login thành công.
2. `Google login state` hiển thị `PASS`.
3. Bấm `Authorize Platform Administrator`.

Kết quả sau khi bấm:

```text
Google login state: PASS
Platform administrator authorization state: vẫn Pending
Approved identity summary: không có email hoặc tên
Create/Select Tenant and POS1: vẫn disabled
Create Pairing Code: vẫn disabled
```

Không còn đủ bằng chứng để giữ verdict `READY_FOR_USER_RUNTIME_TEST` như một closure. Đây là lần sửa tiếp theo cho cùng blocker, nên report 003 phải đầy đủ 100%.

## Mục tiêu

Tìm và sửa chính xác runtime failure khiến `/platform-admin/authorize` không hoàn tất administrator authorization hoặc không persist/hydrate approved identity sau khi người dùng bấm nút.

Task chỉ hoàn thành khi flow thật đạt:

```text
Google login state: PASS
Platform administrator authorization state: PASS
Approved identity summary: có identity đã được API phê duyệt
Create/Select Tenant and POS1: enabled
```

Sau đó kiểm tra action `Create/Select Tenant and POS1` có thể được bấm và trả về kết quả backend cụ thể.

Chưa làm WPF redeem và tuyệt đối chưa làm Phase 2.

## Tài liệu và report bắt buộc phải đọc

```text
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
contracts/v001/WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
prompt/prompt002.md
report/report002.md
```

## Source boundaries được phép

```text
E:\Project2026\PlatformAppV0
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0
E:\Project2026\1ApiServer\ApiServer01.Tests\PlatformAppV0
```

Chỉ sửa file cần thiết trong các boundary trên.

## Điều tra bắt buộc — không được đoán

Phải theo dõi đầy đủ request/runtime chain sau khi bấm nút:

```text
Home.razor button/navigation
-> GET /platform-admin/authorize
-> authenticate application cookie
-> retrieve saved id_token or approved server identity
-> call canonical PlatformAppV0 admin exchange API
-> validate API response
-> persist approved admin claims/proof into encrypted cookie
-> redirect back to UI
-> new request/circuit reads cookie claims
-> Home.razor hydrates administrator PASS + approved identity
-> Tenant/POS gate enables
```

Xác minh từng điểm sau:

1. Nút thật sự điều hướng đến `/platform-admin/authorize` hay không.
2. Endpoint có được map và được gọi trên runtime đang chạy hay không.
3. Endpoint đang authenticate đúng cookie scheme hay nhầm OIDC/external scheme.
4. Authentication result có principal hợp lệ không.
5. Saved `id_token` có tồn tại không; chỉ báo presence/absence, không log token.
6. Google principal có các claim cần thiết như subject, email, name hay không; chỉ log claim type/presence và masked identity an toàn.
7. Canonical API exchange URL/base address có đúng runtime không.
8. API exchange trả HTTP status, resultCode và approved identity gì.
9. Nếu API exchange fail, UI phải hiển thị failure cụ thể thay vì quay về `Pending` im lặng.
10. Cookie re-issue có dùng đúng authentication scheme không.
11. Approved claims có được thêm bằng canonical claim names thống nhất giữa Program.cs và Home.razor không.
12. Redirect response có Set-Cookie thành công không.
13. Sau redirect, request mới có đọc được approved claims không.
14. Blazor circuit có đang giữ AuthenticationState cũ và không refresh sau full navigation hay không.
15. Có stale binary/process, wrong launch profile, wrong port, hoặc app cũ vẫn chạy hay không.
16. Callback path `/signin-google-callback` có trùng/không khớp Google configuration không.
17. Cookie size, SameSite, Secure, Path, expiration hoặc data-protection issue có làm cookie update thất bại không.
18. API server PID/file lock có khiến PlatformAppV0/API binary không đồng bộ không.

## Safe diagnostics bắt buộc

Có thể thêm diagnostic tạm hoặc structured logging an toàn, nhưng:

- không log raw ID token;
- không log access token;
- không log cookie;
- không log client secret;
- không log authorization header;
- không ghi PII đầy đủ nếu không cần;
- chỉ ghi presence flags, claim types, masked email, status code, resultCode, correlation ID và route transition.

Diagnostic phải đủ để phân biệt ít nhất các trạng thái:

```text
AUTHORIZE_ENDPOINT_NOT_REACHED
COOKIE_AUTHENTICATION_FAILED
ID_TOKEN_NOT_PRESENT
GOOGLE_IDENTITY_CLAIMS_MISSING
ADMIN_EXCHANGE_HTTP_FAILED
ADMIN_EXCHANGE_CONTRACT_FAILED
APPROVED_IDENTITY_MISSING
COOKIE_REISSUE_FAILED
APPROVED_CLAIMS_NOT_AVAILABLE_AFTER_REDIRECT
UI_HYDRATION_FAILED
```

Không được để lỗi quay về `Pending` mà không có message cụ thể.

## Yêu cầu sửa

- Sửa root cause, không hard-code PASS.
- Không mock administrator identity trong runtime bình thường.
- Không tự lấy email từ UI rồi coi là approved identity nếu API chưa phê duyệt.
- Approved identity phải đến từ canonical PlatformAppV0 API authorization/exchange result hoặc persisted approved proof được contract cho phép.
- Dùng full-page navigation/redirect nếu cần để authentication cookie mới được đọc bởi request và Blazor circuit mới.
- Đồng bộ canonical claim names bằng constants/contract rõ ràng nếu mismatch là nguyên nhân.
- Sau reload, authorization PASS và approved identity phải còn tồn tại theo contract hiện tại.
- Sign out phải xóa state phù hợp.
- Failure phải fail closed.
- Tenant/POS không được enable nếu approved authorization chưa thật sự PASS.

## Runtime process hygiene

Trước build/test/runtime proof:

1. Xác định tất cả process PlatformAppV0 và ApiServer đang chạy, PID, path binary và listening port.
2. Không kill process không liên quan.
3. Dừng đúng process gây lock nếu cần build lại, rồi restart bằng canonical Visual Studio/launchSettings flow.
4. Chứng minh runtime đang dùng binary mới, không phải process cũ.
5. Ghi PID/port/path an toàn trong report; không ghi secret environment values.

## Build và test bắt buộc

Phải có clean evidence sau khi xử lý process lock:

```text
dotnet build E:\Project2026\PlatformAppV0\PlatformAppV0.sln

dotnet test E:\Project2026\PlatformAppV0\PlatformAppV0.sln --no-build

dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0"
```

Không chấp nhận chỉ `--no-build` cho ApiServer focused tests lần này nếu build path vẫn bị lock. Phải xử lý đúng process lock hoặc báo BLOCKED với root cause và evidence đầy đủ.

Bổ sung focused tests cho ít nhất:

- authorize endpoint authenticates correct cookie scheme;
- missing id_token fails with explicit result;
- API exchange failure is surfaced;
- approved claims are persisted using canonical names;
- UI hydration recognizes persisted claims;
- Tenant/POS gate remains disabled without approved proof;
- Tenant/POS gate enables with valid approved proof;
- reload persistence behavior.

## Physical runtime acceptance

Codex phải chuẩn bị runtime mới và, nếu không thể tự hoàn tất Google interactive login, đưa exact retest steps cho người dùng. Tuy nhiên trước khi giao lại, Codex phải dùng diagnostics/tests để chứng minh route, cookie persistence và UI hydration path không còn silent failure.

User-visible acceptance bắt buộc:

```text
A. Google login state = PASS
B. Bấm Authorize Platform Administrator
C. Authorization state = PASS
D. Approved identity có email/name hoặc approved identity summary hợp lệ
E. Create/Select Tenant and POS1 được enable
F. Bấm Tenant/POS action và nhận success hoặc explicit backend failure
```

Nếu C–E chưa có physical proof, verdict tối đa là `READY_FOR_USER_RUNTIME_RETEST`, không được PASS.

## Git safety

`E:\Project2026` là shared parent repository có dirty state và không có source remote.

Bắt buộc:

- không `git add .`;
- không `git add -A`;
- không reset/clean/stash/checkout/restore thay đổi cũ;
- không push source;
- không commit source nếu không thể isolate chính xác file do task này thay đổi;
- liệt kê exact files touched và phân biệt pre-existing changes với changes của prompt003;
- chỉ commit report vào coordination repository.

## Report 003 — bắt buộc 100% chi tiết

Tạo đúng:

```text
report/report003.md
```

Vì đây là lần sửa tiếp theo cho cùng lỗi, report không được rút gọn. Phải bao gồm đầy đủ:

1. Verdict.
2. Runtime symptom đã reproduce hay chưa.
3. Timeline/request chain từng bước.
4. Root cause chính xác.
5. Vì sao fix prompt002 chưa đạt runtime.
6. Từng acceptance criterion của prompt002 và prompt003: PASS/FAIL/BLOCKED kèm evidence.
7. Exact files changed và ownership/provenance.
8. Exact implementation correction.
9. Safe diagnostics added và kết quả.
10. Process/PID/port/binary-path evidence.
11. Build commands và kết quả đầy đủ.
12. Test commands, pass/fail/skip counts.
13. ApiServer focused test phải build được hoặc blocker 100% chi tiết.
14. Runtime evidence từng mục A–F.
15. Remaining blocker.
16. Risks và phần chưa xác minh.
17. Source Git state và xác nhận không push source.
18. Exact user retest steps.
19. Exact next action nếu vẫn chưa PASS.
20. Coordination report commit SHA và push confirmation trong final response.

## Verdict hợp lệ

Nếu đã sửa và chỉ còn chờ người dùng login/bấm lại:

```text
PLATFORMAPPV0_ADMIN_AUTHORIZATION_RUNTIME_CORRECTION_READY_FOR_USER_RETEST
```

Nếu có physical proof đầy đủ A–F:

```text
PLATFORMAPPV0_ADMIN_AUTHORIZATION_AND_TENANT_POS1_GATE_PASS
```

Nếu vẫn không xác định/sửa được root cause:

```text
BLOCKED_PLATFORMAPPV0_ADMIN_AUTHORIZATION_RUNTIME_FAILURE
```

Không được dùng verdict PASS chỉ vì unit tests PASS.
