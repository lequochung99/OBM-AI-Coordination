# Prompt 006 — Hoàn tất ApiServer Google ClientId và approved administrator local configuration

## Operator decision

Người dùng đã xác nhận một Google email cụ thể làm approved Platform Administrator.

Vì repository `OBM-AI-Coordination` là Public, raw email **không được ghi vào prompt, report hoặc GitHub**.

Trước khi chạy prompt này, operator phải lưu approved email trực tiếp vào ApiServer user-secrets bằng key:

```text
Authentication:Google:ApprovedAdminEmail
```

Codex chỉ được đọc presence, normalized comparison result, masked suffix và hash prefix. Không được in raw email.

## Mục tiêu

Hoàn tất local-only ApiServer configuration để admin exchange có thể xác minh Google ID token và approved administrator identity:

1. xác nhận approved email đã có trong ApiServer user-secrets;
2. đọc Google ClientId từ PlatformAppV0 user-secrets;
3. ghi cùng ClientId vào ApiServer user-secrets;
4. không copy ClientSecret vào ApiServer;
5. restart ApiServer và PlatformAppV0 bằng canonical runtime;
6. chứng minh `GOOGLE_CLIENT_ID_NOT_CONFIGURED` đã biến mất;
7. chuẩn bị user physical retest cho administrator authorization và Tenant/POS gate.

Chưa làm Pairing Code, WPF redeem hoặc Phase 2.

## Bắt buộc đọc

```text
prompt/prompt004.md
report/report004.md
prompt/prompt005.md
report/report005.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

## Source và target projects

Source Google OIDC configuration:

```text
E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\OBM.PlatformAppV0.csproj
```

Target ApiServer configuration:

```text
E:\Project2026\1ApiServer\ApiServer01\ApiServer01.csproj
```

## Precondition fail-closed

Trước mọi mutation, kiểm tra target key:

```text
Authentication:Google:ApprovedAdminEmail
```

Nếu key không tồn tại hoặc rỗng:

```text
BLOCKED_APPROVED_PLATFORM_ADMIN_EMAIL_NOT_PRESEEDED
```

Không được đoán email, lấy từ GitHub profile, report public, browser screenshot hoặc source history.

Nếu key tồn tại:

- normalize để so sánh nội bộ;
- chỉ report `present=true`, masked suffix tối đa 4 ký tự và SHA-256 prefix;
- không in raw value.

## Local user-secrets mutation được phép

Chỉ được thay đổi các ApiServer user-secret keys sau:

```text
Authentication:Google:ClientId
Authentication:Google:ApprovedAdminEmail
```

`ApprovedAdminEmail` đã được operator pre-seed; không thay đổi sang identity khác.

Có thể set compatibility key nếu code hiện tại cần, nhưng phải chứng minh lý do:

```text
PlatformAppV0:GoogleClientId
PlatformAppV0:ApprovedAdminEmail
```

Không được set:

```text
Authentication:Google:ClientSecret
```

ApiServer chỉ cần expected audience ClientId và approved identity để validate Google ID token.

## Exact configuration workflow

1. Đọc source `Authentication:Google:ClientId` từ PlatformAppV0 user-secrets.
2. Xác nhận source ClientId present; không in raw value.
3. Đọc target approved email presence.
4. Ghi source ClientId vào target ApiServer user-secrets key `Authentication:Google:ClientId`.
5. Không ghi raw ClientId/email vào report/log.
6. Re-read target user-secrets và so sánh:
   - source ClientId hash == target ClientId hash;
   - approved email vẫn present và không đổi;
   - ClientSecret không được thêm.
7. Stop đúng stale ApiServer/PlatformAppV0 processes.
8. Build/test.
9. Restart canonical runtimes.
10. Run synthetic exchange with dummy non-secret token.

## Synthetic acceptance

Trước config:

```text
HTTP 401 / GOOGLE_CLIENT_ID_NOT_CONFIGURED
```

Sau config, dummy token **không được** trả `GOOGLE_CLIENT_ID_NOT_CONFIGURED`.

Kết quả chấp nhận có thể là:

```text
GOOGLE_ID_TOKEN_VALIDATION_FAILED
GOOGLE_ID_TOKEN_INVALID_ISSUER
GOOGLE_ID_TOKEN_INVALID_AUDIENCE
GOOGLE_ID_TOKEN_EXPIRED
```

vì dummy token không hợp lệ.

Điều này chứng minh ApiServer đã load ClientId và đi vào token validation.

## Build/test bắt buộc

```text
dotnet build E:\Project2026\PlatformAppV0\PlatformAppV0.sln

dotnet test E:\Project2026\PlatformAppV0\PlatformAppV0.sln --no-build

dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0"
```

ApiServer focused tests phải chạy build-enabled.

Bổ sung/duy trì tests cho:

- effective Google ClientId loaded from target user-secrets/config path;
- missing ClientId returns explicit error;
- configured ClientId reaches validator;
- missing approved identity fails explicit;
- approved email normalization/match;
- mismatched approved email fails explicit;
- successful validated identity returns Platform admin JWT;
- no ClientSecret required by ApiServer exchange.

## Runtime process evidence

Report PID, binary path và ports an toàn:

```text
ApiServer: 7161
PlatformAppV0: 7012
```

Probes:

```text
GET http://127.0.0.1:7161/api/platform-v0/readiness
GET https://localhost:7012/
POST synthetic admin exchange with dummy token
```

Không log raw token/config values.

## User physical retest

Sau khi Codex chuẩn bị runtime:

1. Open `https://localhost:7012`.
2. Sign out để loại stale cookie.
3. Login lại bằng Google account đã được operator phê duyệt.
4. Confirm Google login = PASS.
5. Click `Authorize Platform Administrator`.
6. URL không được chứa:

```text
HTTP_401_GOOGLE_CLIENT_ID_NOT_CONFIGURED
```

7. Expected:

```text
Platform administrator authorization = PASS
Approved identity summary = populated
Create/Select Tenant and POS1 = enabled
```

8. Click `Create/Select Tenant and POS1`.
9. Ghi success hoặc explicit backend resultCode.
10. Không tạo Pairing Code trong task này.

## Failure handling

Nếu real Google exchange fail, giữ exact safe resultCode:

```text
GOOGLE_ID_TOKEN_INVALID_AUDIENCE
GOOGLE_EMAIL_NOT_VERIFIED
PLATFORM_ADMIN_BOOTSTRAP_IDENTITY_NOT_FOUND
PLATFORM_ADMIN_IDENTITY_MISMATCH
```

Không map trở lại generic `PLATFORM_ADMIN_NOT_AUTHENTICATED`.

## Git safety

- User-secrets local mutation được phép theo prompt này.
- Không commit user-secrets.
- Không ghi secret vào source/appsettings/report.
- Không `git add .` hoặc `git add -A`.
- Không reset/clean/stash/checkout/restore.
- Không source commit/push.
- Chỉ commit `report/report006.md` vào coordination repository.

## Report 006 — bắt buộc 100% chi tiết

Tạo:

```text
report/report006.md
```

Report phải bao gồm:

1. Verdict.
2. Approved email presence evidence đã mask/hash.
3. Source/target ClientId presence + hash-match evidence.
4. Exact user-secret keys changed, không có raw values.
5. Confirmation ClientSecret không được set.
6. Effective options verification.
7. Synthetic exchange before/after.
8. Exact resultCode sau config.
9. Build/test commands và counts.
10. PID/port/binary evidence.
11. Runtime readiness probes.
12. User acceptance matrix.
13. Remaining blocker/risks.
14. Source Git/no push confirmation.
15. Exact user retest steps.
16. Coordination commit SHA trong final response.

## Verdict hợp lệ

Nếu local config hoàn tất và synthetic proof PASS, chờ user Google retest:

```text
APISERVER_GOOGLE_ADMIN_CONFIG_READY_FOR_USER_RETEST
```

Nếu physical flow đến Tenant/POS gate PASS:

```text
PLATFORMAPPV0_ADMIN_AUTHORIZATION_AND_TENANT_POS1_GATE_PASS
```

Nếu approved email chưa được pre-seed:

```text
BLOCKED_APPROVED_PLATFORM_ADMIN_EMAIL_NOT_PRESEEDED
```

Nếu ClientId không thể copy/load:

```text
BLOCKED_APISERVER_GOOGLE_CLIENT_ID_CONFIGURATION
```
