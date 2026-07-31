# Prompt 001 — Khởi tạo quy trình phối hợp đơn giản

## Mục tiêu

Từ thời điểm này, sử dụng quy trình phối hợp thủ công tối giản giữa ChatGPT và Codex trong repository `lequochung99/OBM-AI-Coordination`.

Chỉ dùng hai thư mục chính:

```text
prompt/
report/
```

Quy tắc đánh số:

```text
prompt/prompt001.md  -> report/report001.md
prompt/prompt002.md  -> report/report002.md
prompt/prompt003.md  -> report/report003.md
```

Mỗi prompt và report tăng tuần tự, không ghi đè file cũ.

## Trách nhiệm

- ChatGPT tạo và commit file mới trong `prompt/`.
- Codex đọc đúng prompt có số tương ứng, thực hiện task, rồi tạo và commit report cùng số trong `report/`.
- Người dùng báo cho ChatGPT khi Codex đã hoàn thành report.
- ChatGPT đọc report, review kết quả và tạo prompt kế tiếp.

## Quy tắc report

### Lần thực hiện đầu tiên

Report có thể tập trung vào khoảng 80% thông tin quan trọng nhất, nhưng tối thiểu phải có:

- verdict;
- việc đã làm;
- file thay đổi;
- test/evidence;
- blocker;
- commit SHA nếu có;
- bước tiếp theo.

### Nếu task chạy lại mà vẫn chưa PASS

Report lần sau phải đầy đủ 100%, gồm:

- từng acceptance criterion và trạng thái;
- nguyên nhân gốc;
- evidence chi tiết;
- file/path liên quan;
- test đầy đủ;
- blocker và rủi ro;
- phần chưa xác minh;
- lý do lần trước chưa đạt;
- exact next action;
- commit SHA và trạng thái push.

## Phạm vi task này

Đây chỉ là task khởi tạo quy trình phối hợp đơn giản.

Không được:

- sửa source code OBM;
- thay đổi Git của `E:\Project2026`;
- tạo remote source;
- commit hoặc push source;
- làm PlatformAppV0 Phase 1;
- làm Phase 2;
- xóa cấu trúc coordination cũ.

Cấu trúc cũ như `prompts/`, `reports/`, `handoffs/`, `coordination/` được giữ nguyên làm lịch sử, nhưng từ task kế tiếp chỉ dùng lane mới `prompt/` và `report/`.

## Kết quả bắt buộc

Codex tạo đúng một file:

```text
report/report001.md
```

Nội dung report phải xác nhận:

1. Đã đọc `prompt/prompt001.md`.
2. Đã hiểu quy trình hai thư mục.
3. Từ prompt tiếp theo sẽ dùng đúng cặp số tương ứng.
4. Không xóa hoặc ghi đè artifact cũ.
5. Không sửa source code trong task này.
6. Ghi commit SHA của report và xác nhận đã push `origin/main`.

## Verdict mong đợi

```text
SIMPLE_PROMPT_REPORT_WORKFLOW_INITIALIZED
```
