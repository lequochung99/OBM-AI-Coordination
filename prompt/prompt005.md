# Prompt 005 — Cấu hình ApiServer Google audience và approved administrator

## Trạng thái đã xác định

Đọc đầy đủ:

```text
prompt/prompt004.md
report/report004.md
```

Root cause hiện tại đã được xác định chính xác:

```text
POST /api/platform-v0/admin/google/exchange
HTTP 401
resultCode = GOOGLE_CLIENT_ID_NOT_CONFIGURED
```

Failure condition nằm tại:

```csharp
string.IsNullOrWhiteSpace(options.Value.GoogleClientId)
```

Source code đã tách result code và có Google ID-token validator. Bước tiếp theo là sửa **effective local runtime configuration**, không tiếp tục sửa UI hydration và không hard-code giá trị vào source.

## Mục tiêu

Cấu hình an toàn cho ApiServer Phase1 runtime:

```text
Authentication:Google:ClientId
Authentication:Google:ApprovedAdminEmail
```

Hoặc approved Google subject nếu canonical local configuration đã có:

```text
Authentication:Google:ApprovedGoogleSubject
```

Sau restart, synthetic proof phải không còn trả:

```text
GOOGLE_CLIENT_ID_NOT_CONFIGURED
```

Sau đó chuẩn bị PlatformAppV0 để người dùng thực hiện physical retest:

```text
Google login = PASS
Authorize Platform Administrator
Administrator authorization = PASS
Approved identity populated
Create/Select Tenant and POS1 enabled
```

Chưa tạo Pairing Code, chưa WPF redeem và chưa Phase 2.

## Quy tắc lấy configuration

### Google ClientId

- Lấy đúng effective Google ClientId đang được PlatformAppV0 dùng để đăng nhập OIDC thành công.
- Ưu tiên đọc từ user-secrets/config canonical của project:

```text
E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\OBM.PlatformAppV0.csproj
```

- Copy cùng ClientId vào user-secrets của:

```text
E:\Project2026\1ApiServer\ApiServer01\ApiServer01.csproj
```

- Không tạo ClientId mới.
- Không thay đổi Google Cloud OAuth client trong task này.
- Không ghi full ClientId vào report public. Chỉ ghi `present`, masked suffix tối đa 6 ký tự hoặc SHA-256 rút gọn.

### Approved administrator identity

Tìm approved admin email/subject từ canonical local configuration đã tồn tại, theo thứ tự:

1. `Authentication:Google:ApprovedAdminEmail` trong PlatformAppV0 user-secrets/config.
2. `PlatformAppV0:ApprovedAdminEmail` hoặc key canonical tương đương đã được source/report xác nhận.
3. `Authentication:Google:ApprovedGoogleSubject` nếu đã tồn tại.

Không được:

- suy đoán email từ GitHub account;
- hard-code email trong source;
- tự chọn một Google account khác;
- dùng email chưa được operator/canonical config phê duyệt.

Nếu không tìm thấy approved email hoặc subject canonical, dừng với verdict:

```text
BLOCKED_APPROVED_PLATFORM_ADMIN_IDENTITY_INPUT_REQUIRED
```

Và báo chính xác key nào còn thiếu. Không in các identity khác đã tìm thấy ngoài dạng masked an toàn.

## Thao tác configuration được phép

Dùng `dotnet user-secrets` cho ApiServer project. Có thể truyền giá trị qua PowerShell variable trong memory để tránh echo ra console/report.

Target keys:

```text
Authentication:Google:ClientId
Authentication:Google:ApprovedAdminEmail
Authentication:Google:ApprovedGoogleSubject
```

Chỉ set key thứ ba nếu canonical subject đã tồn tại.

Không set Google ClientSecret trong ApiServer. API chỉ cần expected audience ClientId để validate Google ID token.

Không ghi các giá trị này vào:

- appsettings committed;
- source code;
- coordination repository;
- report ở dạng đầy đủ;
- logs/evidence public.

## Verification configuration bắt buộc

Sau khi set user-secrets:

1. Xác nhận target ApiServer user-secrets có key ClientId bằng presence flag.
2. Xác nhận có approved email hoặc subject bằng presence flag.
3. Xác nhận `IOptions<PlatformAppV0Options>` resolve effective values trong Phase1 startup path.
4. Không in raw values.
5. Restart đúng ApiServer process và PlatformAppV0 process.
6. Ghi PID, executable path và port.

## Synthetic runtime proof

Gọi với dummy token không nhạy cảm:

```text
POST http://127.0.0.1:7161/api/platform-v0/admin/google/exchange
Content-Type: application/json
{"idToken":"dummy-non-secret-token"}
```

Expected:

- không còn `GOOGLE_CLIENT_ID_NOT_CONFIGURED`;
- trả một token-validation result cụ thể như `GOOGLE_ID_TOKEN_VALIDATION_FAILED`;
- không được trả success cho dummy token.

Nếu vẫn `GOOGLE_CLIENT_ID_NOT_CONFIGURED`, điều tra user-secrets ID/startup configuration chain và sửa configuration loading; không hard-code.

## Build/test

Vì source correction prompt004 đã có, task này ưu tiên configuration/runtime. Tuy nhiên phải chạy tối thiểu:

```text
dotnet build E:\Project2026\PlatformAppV0\PlatformAppV0.sln

dotnet test E:\Project2026\PlatformAppV0\PlatformAppV0.sln --no-build

dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0"
```

Ghi pass/fail counts. Nếu không có source change mới, nói rõ.

## Physical user retest state

Sau synthetic proof, để runtime chạy tại:

```text
ApiServer: http://127.0.0.1:7161
PlatformAppV0: https://localhost:7012
```

Người dùng sẽ:

1. Sign out khỏi browser session cũ.
2. Login Google lại.
3. Bấm `Authorize Platform Administrator`.
4. Gửi URL/result và ảnh.

Expected user result:

```text
URL không còn HTTP_401_GOOGLE_CLIENT_ID_NOT_CONFIGURED
Administrator authorization = PASS
Approved identity populated
Create/Select Tenant and POS1 enabled
```

Nếu Google token audience mismatch sau khi ClientId đã load, result phải là:

```text
GOOGLE_ID_TOKEN_INVALID_AUDIENCE
```

Khi đó phải so sánh masked/hash của OIDC ClientId và API expected ClientId, không in raw values.

## Git/source safety

- Không commit/push `E:\Project2026` source.
- Không `git add .` hoặc `git add -A`.
- Không reset/clean/stash/checkout/restore.
- Không chạm WPF, Pairing Code, Phase 2 hoặc local POS database.
- User-secrets local mutation được phép trong task này.
- Chỉ commit report vào coordination repository.

## Report 005 — 100% chi tiết

Tạo:

```text
report/report005.md
```

Report phải có:

1. Verdict.
2. Config source project và target project.
3. User-secrets IDs hoặc project paths, nhưng không secret values.
4. Presence/masked/hash evidence cho ClientId.
5. Presence/masked evidence cho approved email/subject.
6. Exact keys set.
7. Effective options verification.
8. Synthetic exchange before/after status/resultCode.
9. Process PID/port/path.
10. Build/test results.
11. Source files changed, nếu có; expected là none hoặc minimal config-loader correction only.
12. Xác nhận không ghi secret vào repo/report.
13. Xác nhận source không commit/push.
14. Exact user retest steps.
15. Remaining blocker nếu chưa PASS.
16. Coordination commit SHA trong final response.

## Verdict hợp lệ

Nếu config đã load và synthetic proof PASS, chờ user retest:

```text
APISERVER_GOOGLE_ADMIN_CONFIG_READY_FOR_USER_RETEST
```

Nếu thiếu approved identity canonical:

```text
BLOCKED_APPROVED_PLATFORM_ADMIN_IDENTITY_INPUT_REQUIRED
```

Nếu config đã set nhưng runtime vẫn không load:

```text
BLOCKED_APISERVER_GOOGLE_CONFIG_NOT_EFFECTIVE
```

Không tuyên bố admin authorization PASS nếu chưa có physical Google retest.
