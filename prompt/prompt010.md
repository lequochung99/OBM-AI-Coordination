# Prompt 010 — Điều tra fresh Pairing Code vẫn trả PAIRING_CODE_EXPIRED

## Physical symptom mới

Người dùng đã chạy WPF InstallationV0 Phase 1 và nhập Pairing Code, nhưng WPF vẫn hiển thị:

```text
HTTP 400
resultCode = PAIRING_CODE_EXPIRED
```

UI vẫn đúng fail-closed:

```text
Pairing Code redeemed = Blocked
WpfJwt received = BLOCKED
Protected hello = BLOCKED
/bootstrap/me = BLOCKED
DPAPI/checkpoint = not started
Local database touched = false
```

Người dùng cho biết đã thử lại nhưng vẫn giống cũ. Không được kết luận đơn giản rằng user nhập code cũ. Phải xác định bằng evidence liệu WPF gửi code cũ, code mới, request cũ, hay ApiServer đánh giá expiration sai.

## Mục tiêu

Xác định và sửa chính xác nguyên nhân một Pairing Code vừa được tạo/nhập lại vẫn nhận:

```text
HTTP 400 / PAIRING_CODE_EXPIRED
```

Sau correction, một code mới hợp lệ phải redeem được và tiếp tục:

```text
WpfJwt issued
protected hello PASS
/bootstrap/me PASS
DPAPI/checkpoint PASS
```

Chưa làm Phase 2.

## Bắt buộc đọc

```text
prompt/prompt008.md
report/report008.md
prompt/prompt009.md
report/report009.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

## Source boundaries được phép

```text
E:\Project2026\PlatformAppV0
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0
E:\Project2026\1ApiServer\ApiServer01.Tests\PlatformAppV0
E:\Project2026\4POS\NailSalonNet8\InstallationV0
E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0
```

## WPF visible prompt label — bắt buộc từ prompt010 trở đi

Mỗi prompt có sửa WPF/InstallationV0 phải cập nhật một label nhìn thấy rõ trên UI để operator biết chắc đang mở đúng binary mới nhất.

Đối với task này, giá trị chính xác là:

```text
prompt010
```

Yêu cầu implementation:

1. Tạo một constant duy nhất trong InstallationV0, ví dụ:

```csharp
public const string CoordinationPromptLabel = "prompt010";
```

2. Hiển thị chính xác `prompt010` ở ít nhất hai nơi:

```text
Window title: OBM InstallationV0 Phase 1 - prompt010
Visible UI label gần tiêu đề: Build label: prompt010
```

3. Label phải lấy từ constant/code của binary đang chạy; không đọc từ file coordination GitHub tại runtime.
4. Không dùng timestamp thay cho prompt label.
5. Không được để label cũ như `prompt009` sau khi build prompt010.
6. Thêm focused test xác nhận source/UI chứa chính xác label hiện tại và không còn label prompt trước trong active WPF title/header path.
7. Report phải chụp/ghi evidence exact title/header text.

Quy tắc lâu dài cho các prompt sau:

```text
Nếu prompt011 sửa WPF -> label = prompt011
Nếu prompt012 sửa WPF -> label = prompt012
...
```

Nếu một prompt không sửa WPF thì không bắt buộc đổi WPF label.

Operator acceptance:

```text
Nếu UI không hiển thị đúng prompt number hiện tại, không được yêu cầu user test chức năng.
```

## Điều tra bắt buộc

Trace full lifecycle của một fresh Pairing Code:

```text
PlatformAppV0 Create Pairing Code click
-> API create endpoint
-> persisted pairing authorization record
-> CreatedAtUtc / ExpiresAtUtc / status
-> one-time plaintext returned once
-> user enters code into WPF
-> WPF click handler reads current textbox/password-box value
-> normalization
-> Phase1InstallationService receives current value
-> POST redeem request
-> API lookup by normalized protected code
-> record found
-> server now UTC comparison
-> active/expired/used decision
```

Phải xác minh từng mục:

1. Pairing Code lifetime configured bao lâu.
2. `CreatedAtUtc`, `ExpiresAtUtc`, và server `UtcNow` đều dùng UTC hay có local/UTC mismatch.
3. Có sub-second/tick precision issue ở expiration không.
4. API create endpoint có persist đúng `ExpiresAtUtc` vừa trả UI không.
5. PlatformAppV0 có đang hiển thị code/expiry của record mới nhất không.
6. WPF click handler có đọc giá trị hiện tại từ control không, hay dùng cached field/old command parameter.
7. Sau một failed attempt, WPF nhập code mới có thực sự thay thế giá trị cũ không.
8. WPF có trim/remove spaces/hyphens đúng contract không.
9. `EnsurePendingAsync` có giữ request GUID/local installation đúng nhưng không được giữ Pairing Code cũ.
10. HTTP body hiện tại có chứa code mới vừa nhập hay request object cũ.
11. ApiServer có đang chạy cùng state store mà PlatformAppV0 dùng để tạo code không.
12. Có nhiều ApiServer instance/process hoặc nhiều state file/ProductRoot khác nhau không.
13. Pairing record lookup có tìm nhầm record cũ cùng normalized code/hash không.
14. Expired branch exact condition là gì (`now >= expires`, `now > expires`, status flag, cancellation, used state).
15. Pairing code generation có khả năng collision/reuse không.
16. PlatformAppV0 có thể tạo code mới nhưng API create response lấy record cũ do idempotency key reuse không.
17. Browser/runtime instance có stale binary/config không.

## Safe correlation evidence

Không được log hoặc report raw Pairing Code.

Để correlate flow, dùng các safe identifiers:

```text
PairingAuthorizationGuid
TenantGuid
PosStationId
CreatedAtUtc
ExpiresAtUtc
ServerUtc
status
clientRequestGuid
localInstallationGuid
correlationId
```

Không ghi hash của short Pairing Code vào public report vì có thể brute-force.

Nếu API expired problem response chưa trả đủ safe correlation, có thể bổ sung:

```text
pairingAuthorizationGuid
createdAtUtc
expiresAtUtc
serverUtc
correlationId
```

Chỉ trả các field không phải secret.

## WPF diagnostics an toàn

WPF UI/result model phải có thể hiển thị:

```text
PAIRING_CODE_INPUT_PRESENT = true|false
PAIRING_CODE_INPUT_LENGTH = <length only>
PAIRING_CODE_REQUEST_SENT = true|false
redeemClientRequestGuid
HTTP status
resultCode
correlationId
pairingAuthorizationGuid (nếu API trả)
createdAtUtc / expiresAtUtc / serverUtc (nếu API trả)
```

Không hiển thị code plaintext.

Sau failed attempt, khi người dùng nhập code khác:

- click phải đọc lại control value;
- không reuse cached plaintext;
- request DTO phải được tạo mới với current input;
- pending checkpoint vẫn giữ same idempotency GUID theo retry contract nhưng không chứa Pairing Code.

## Yêu cầu sửa

- Sửa root cause tối thiểu.
- Không kéo dài expiry tùy tiện để che lỗi.
- Không bypass expiration validation.
- Không cho expired/used code redeem.
- Không log Pairing Code.
- Không lưu Pairing Code vào checkpoint.
- Không reset `LocalInstallationGuid` hoặc tạo attempt trùng không cần thiết.
- Giữ protected hello, `/bootstrap/me`, expiration precision correction, DPAPI và restart/resume flow.
- Giữ visible WPF label chính xác là `prompt010` trong title và UI.

## Tests bắt buộc

Bổ sung/duy trì tests cho:

- fresh code created then immediately redeemed = PASS;
- exact boundary before expiry = PASS;
- exact boundary at/after expiry = EXPIRED theo canonical rule;
- UTC/local timezone mismatch regression;
- create response timestamps equal persisted record timestamps;
- idempotent create does not accidentally return expired prior record unless contract explicitly requires;
- WPF current input replaces previous failed code;
- WPF does not cache Pairing Code across failed attempts;
- WPF request DTO contains current input (assert internally, never print value);
- expired response preserves safe correlation fields;
- no DPAPI/checkpoint on expired response;
- successful fresh redeem continues to protected hello and `/bootstrap/me`;
- WPF title/header show exact label `prompt010`;
- active WPF title/header path does not show stale `prompt009`.

Build/test:

```text
dotnet build E:\Project2026\PlatformAppV0\PlatformAppV0.sln

dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0"

dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0"
```

## Runtime handoff

Cuối task:

1. Stop đúng stale ApiServer/PlatformAppV0/WPF processes.
2. Build/test latest source.
3. Restart ApiServer port 7161 và PlatformAppV0 port 7012 từ latest Debug output.
4. Prove PID, start time, binary LastWriteTime, path, port owner.
5. Không để WPF chạy nền.
6. Không tự redeem real Pairing Code.
7. Giao user chạy WPF Visual Studio Debug thủ công.
8. Không yêu cầu user functional retest nếu WPF UI chưa hiển thị chính xác `Build label: prompt010`.

## Physical retest steps phải chuẩn bị

1. User mở PlatformAppV0.
2. Create a fresh Pairing Code.
3. Ghi lại expiry trên UI, không gửi code vào report/chat.
4. Mở WPF bằng Visual Studio Debug.
5. Xác nhận trước tiên:

```text
Window title contains: prompt010
Visible UI contains: Build label: prompt010
```

6. Nếu label không đúng, dừng test vì đang chạy stale binary.
7. Nhập code mới ngay.
8. Confirm fresh code no longer returns `PAIRING_CODE_EXPIRED`.
9. Confirm protected hello marker PASS.
10. Confirm `/bootstrap/me`, DPAPI, checkpoint PASS.
11. Restart WPF cùng ProductRoot và confirm resume without second redeem.

## Git safety

- Không `git add .` hoặc `git add -A`.
- Không reset/clean/stash/checkout/restore.
- Không source commit/push.
- Chỉ commit `report/report010.md` vào coordination repository.
- Không ghi Pairing Code/token/secret vào report.

## Report 010 — bắt buộc 100% chi tiết

Tạo:

```text
report/report010.md
```

Report phải có:

1. Verdict.
2. Exact physical symptom.
3. Full fresh-code lifecycle timeline.
4. Safe timestamp/correlation matrix.
5. Exact expired branch/file/method/condition.
6. Whether WPF sent current or cached code.
7. Whether PlatformAppV0 and ApiServer shared same state/runtime.
8. Root cause.
9. Why previous tests missed it.
10. Exact files changed.
11. Exact correction.
12. Tests added and counts.
13. Runtime instance evidence.
14. Exact WPF prompt-label constant, window title và visible UI text.
15. Confirmation stale prompt label is absent from active title/header path.
16. Confirmation no raw code/token logged.
17. Confirmation no DB/Phase 2.
18. Exact user retest steps.
19. Coordination commit SHA.

## Verdict hợp lệ

Nếu correction hoàn tất và chờ user retest:

```text
FRESH_PAIRING_CODE_EXPIRATION_CORRECTION_READY_FOR_USER_RETEST
```

Nếu physical Phase 1 full PASS:

```text
PHASE1_WPF_API_AUTHORIZATION_AND_MACHINE_PERSISTENCE_PASS_DATABASE_NOT_STARTED
```

Nếu chưa xác định được:

```text
BLOCKED_FRESH_PAIRING_CODE_STILL_EXPIRED
```
