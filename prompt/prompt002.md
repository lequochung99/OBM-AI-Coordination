# Prompt 002 — Sửa PlatformAppV0 administrator authorization để mở khóa Tenant/POS1

## Mục tiêu

Chỉ tập trung sửa blocker đầu tiên của Phase 1 trong `PlatformAppV0`:

```text
Google ID token unavailable from authenticated session
```

Sau khi sửa, người dùng phải có thể:

1. đăng nhập Google thành công;
2. bấm `Authorize Platform Administrator` và nhận trạng thái PASS;
3. thấy approved administrator identity được điền đúng;
4. thấy `Create/Select Tenant and POS1` được enable;
5. bấm được `Create/Select Tenant and POS1` và nhận trạng thái thành công hoặc một lỗi backend cụ thể, không còn bị chặn bởi administrator authorization.

Task này chưa yêu cầu hoàn tất Pairing Code nếu prerequisite Tenant/POS1 chưa xong. Không được làm WPF redeem hoặc Phase 2.

## Tài liệu canonical

Đọc và tuân thủ:

```text
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Bản đã publish để tham chiếu:

```text
contracts/v001/WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Chỉ thực hiện phần PlatformAppV0 Phase 1 liên quan đến administrator authorization và prerequisite Tenant/POS1.

## Source boundaries được phép

```text
E:\Project2026\PlatformAppV0
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0
E:\Project2026\1ApiServer\ApiServer01.Tests\PlatformAppV0
```

Chỉ sửa file thật sự cần thiết trong các boundary trên.

## Tình trạng hiện tại cần reproduce

UI hiện cho thấy:

```text
Google login state: PASS
Platform administrator authorization: FAILED
Failure: Google ID token unavailable from authenticated session
Approved identity: Pending administrator authorization
Create/Select Tenant and POS1: disabled
Create Pairing Code: disabled
```

## Yêu cầu điều tra

1. Reproduce lỗi bằng startup/debug flow chuẩn của Visual Studio.
2. Xác định chính xác nơi Google authentication thành công nhưng ID token không còn khả dụng khi action `Authorize Platform Administrator` chạy.
3. Kiểm tra đúng authentication scheme, cookie scheme, OIDC token persistence, callback, server-side auth properties, token retrieval path và Blazor Server circuit/session boundary.
4. Không được chỉ che lỗi bằng UI hoặc hard-code identity/token.
5. Không được dùng mock identity trong runtime bình thường.
6. Không được log raw Google ID token, access token, cookie hoặc secret.
7. Nếu thiết kế hiện tại không cần raw ID token sau callback, chỉ được thay contract sau khi chứng minh backend authorization vẫn xác minh được Google identity một cách an toàn và deterministic. Không được hạ thấp security để làm nút PASS.

## Yêu cầu implementation

- Sửa root cause tối thiểu.
- Giữ Google login hiện tại hoạt động.
- `Authorize Platform Administrator` phải thực hiện transition thật, không chỉ đổi màu UI.
- Approved identity phải lấy từ authenticated server-side identity đã được xác minh.
- Refresh/reload không được làm mất authorization state đã persist theo contract hiện có.
- Failure phải fail closed và hiển thị message cụ thể.
- Không tạo Pairing Code trước khi administrator authorization và Tenant/POS1 PASS.
- Không chạm local POS database.

## Git safety

`E:\Project2026` là một shared parent Git repository, không có source remote và đang có nhiều thay đổi cũ.

Bắt buộc:

- không dùng `git add .`;
- không dùng `git add -A`;
- không reset, clean, stash, checkout hoặc ghi đè thay đổi cũ;
- chỉ sửa/stage path thuộc task này nếu cần local commit;
- không push source vì source repo chưa có remote;
- report phải liệt kê chính xác file Codex sửa;
- mọi thay đổi ngoài task phải được giữ nguyên.

Task có thể kết thúc với source changes chưa commit nếu việc tạo local commit sẽ trộn thay đổi cũ không rõ ownership. Khi đó report phải nói rõ.

## Build và test bắt buộc

Chạy bằng quy trình Visual Studio/.NET canonical, tối thiểu:

1. build solution/project PlatformAppV0;
2. chạy focused tests liên quan authentication/authorization;
3. chạy regression tests phù hợp trong `OBM.PlatformAppV0.Tests` và `ApiServer01.Tests/PlatformAppV0`;
4. ghi exact command, pass/fail count và lỗi còn lại.

Không được tuyên bố PASS chỉ dựa trên unit test nếu chưa có runtime evidence.

## Runtime evidence bắt buộc

Codex phải tự chạy hoặc chuẩn bị app ở trạng thái để người dùng test và báo rõ từng bước:

```text
Google login PASS
Authorize Platform Administrator PASS
Approved identity populated
Create/Select Tenant and POS1 enabled
Create/Select Tenant and POS1 action result
Create Pairing Code state after Tenant/POS1 result
```

Không chụp/log token hoặc secret.

## Acceptance criteria

Task chỉ PASS khi:

- lỗi `Google ID token unavailable from authenticated session` không còn xuất hiện trong flow hợp lệ;
- Google login vẫn PASS;
- `Authorize Platform Administrator` chạy thật và PASS;
- approved administrator identity hiển thị đúng;
- `Create/Select Tenant and POS1` được enable;
- action Tenant/POS1 không còn bị chặn bởi administrator authorization;
- tests/build liên quan PASS hoặc mọi failure không liên quan được phân loại rõ;
- không làm Phase 2;
- không chạm local POS database;
- không lộ secret;
- không phá hoặc ghi đè thay đổi cũ trong `E:\Project2026`.

Nếu runtime physical proof chưa thực hiện được, verdict không được là PASS hoàn toàn. Dùng `READY_FOR_USER_RUNTIME_TEST` hoặc `BLOCKED` phù hợp.

## Report bắt buộc

Tạo đúng:

```text
report/report002.md
```

Report lần đầu có thể tập trung khoảng 80%, nhưng tối thiểu phải có:

- verdict;
- root cause;
- files changed;
- exact fix;
- build/test results;
- runtime evidence;
- remaining blocker;
- source Git state;
- source local commit SHA nếu có;
- confirmation source was not pushed;
- exact user retest steps;
- next recommended task.

Nếu phải chạy lại mà vẫn chưa PASS, lần report tiếp theo phải đầy đủ 100% theo quy tắc đã thống nhất.

## Verdict mong đợi

Nếu Codex đã sửa xong và chỉ còn chờ người dùng bấm thử:

```text
PLATFORMAPPV0_ADMIN_AUTHORIZATION_FIX_READY_FOR_USER_RUNTIME_TEST
```

Nếu Codex có physical runtime proof đầy đủ:

```text
PLATFORMAPPV0_ADMIN_AUTHORIZATION_AND_TENANT_POS1_GATE_PASS
```
