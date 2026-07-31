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
5. build và restart ApiServer/PlatformAppV0 bằng canonical runtime;
6. bắt buộc chứng minh instance mới nhất đang chạy;
7. chứng minh `GOOGLE_CLIENT_ID_NOT_CONFIGURED` đã biến mất;
8. chuẩn bị user physical retest cho administrator authorization và Tenant/POS gate.

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
7. Xác định mọi process cũ đang giữ ApiServer/PlatformAppV0 binary hoặc port.
8. Stop đúng stale processes.
9. Build/test hoàn tất.
10. Ghi build completion UTC và fingerprint binary mới.
11. Start canonical ApiServer và PlatformAppV0 từ output vừa build.
12. Chứng minh listener duy nhất và process mới nhất đang phục vụ request.
13. Run synthetic exchange với dummy non-secret token.

## Bắt buộc chứng minh instance mới nhất đang chạy

Không được chỉ ghi `runtime restarted`. Report phải có bằng chứng vật lý cho **cả ApiServer và PlatformAppV0**.

### A. Trước build

Liệt kê an toàn:

```text
process name
PID
start time UTC/local
executable path
command line hoặc launch profile name nếu an toàn
listening port
```

Cho các port:

```text
7161 — ApiServer
7012 — PlatformAppV0
```

### B. Sau build

Ghi cho từng binary vừa build:

```text
absolute binary path
LastWriteTimeUtc
file length
SHA-256 hoặc SHA-256 prefix tối thiểu 16 hex
build completion UTC
```

Binary canonical:

```text
E:\Project2026\1ApiServer\ApiServer01\bin\Debug\net8.0\ApiServer01.exe
E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\bin\Debug\net8.0\OBM.PlatformAppV0.exe
```

### C. Sau restart

Với process thực sự listen port, report phải chứng minh:

```text
PID mới
process StartTimeUtc
executable path khớp canonical binary path
process StartTimeUtc >= build completion UTC
binary LastWriteTimeUtc <= process StartTimeUtc
port owner PID khớp PID được báo cáo
chỉ một listener hợp lệ trên mỗi port
```

Nếu wrapper `dotnet.exe` được dùng, phải truy ngược command line/module path để chứng minh nó đang chạy đúng DLL vừa build.

### D. Runtime build marker

Nếu app hiện chưa có cách chứng minh version/fingerprint, được phép thêm một marker an toàn, không secret, ví dụ response header hoặc readiness field:

```text
buildStartedAtUtc
assemblyInformationalVersion
source/build fingerprint
processStartedAtUtc
```

Marker phải lấy từ assembly/runtime hiện tại, không hard-code một giá trị PASS.

Ưu tiên:

- ApiServer readiness trả build/process marker;
- PlatformAppV0 diagnostic/readiness endpoint hoặc response header trả marker tương ứng.

### E. Restart fail-closed

Nếu không chứng minh được process đang chạy là binary mới nhất:

```text
BLOCKED_LATEST_RUNTIME_INSTANCE_NOT_PROVEN
```

Không được giao user retest và không được dùng verdict READY.

### F. Không chấp nhận

Các bằng chứng sau **không đủ một mình**:

- HTTP 200;
- port đang listen;
- process tồn tại;
- build PASS;
- nói rằng đã restart;
- PID khác PID cũ nhưng không đối chiếu binary timestamp/hash;
- chạy từ một output folder khác canonical.

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
- no ClientSecret required by ApiServer exchange;
- runtime marker không rỗng và phản ánh assembly/process hiện tại nếu marker được thêm.

## Runtime process evidence

Report PID, binary path, timestamps, fingerprints và ports an toàn:

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

Phải đối chiếu runtime marker với binary/process evidence. Không log raw token/config values.

## User physical retest

Chỉ giao user retest sau khi `LATEST_RUNTIME_INSTANCE_PROVEN = true` cho cả hai process.

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
10. Pre-build process inventory.
11. Build completion UTC.
12. Binary paths, LastWriteTimeUtc, size và hash fingerprint.
13. Post-restart PID, StartTimeUtc, executable/module path và port-owner proof.
14. Proof chỉ có một valid listener trên 7161 và 7012.
15. Runtime marker/fingerprint comparison.
16. Explicit flags:

```text
APISERVER_LATEST_INSTANCE_PROVEN = true|false
PLATFORMAPPV0_LATEST_INSTANCE_PROVEN = true|false
LATEST_RUNTIME_INSTANCE_PROVEN = true|false
```

17. Runtime readiness probes.
18. User acceptance matrix.
19. Remaining blocker/risks.
20. Source Git/no push confirmation.
21. Exact user retest steps.
22. Coordination commit SHA trong final response.

## Verdict hợp lệ

Nếu local config hoàn tất, synthetic proof PASS và latest runtime instance được chứng minh:

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

Nếu không chứng minh được instance mới nhất:

```text
BLOCKED_LATEST_RUNTIME_INSTANCE_NOT_PROVEN
```
