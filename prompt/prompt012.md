# Prompt 012 — Phase 2 read-only audit cho baseline seed một transaction

## Quyết định của operator

Phase 1 đã được physical test và đóng với trạng thái:

```text
PHASE1_WPF_API_AUTHORIZATION_AND_MACHINE_PERSISTENCE_PASS_DATABASE_NOT_STARTED
```

Phase 2 chưa được phép seed ngay. Trước khi implementation, phải điều tra lại một lần đầy đủ để:

1. tham khảo cấu trúc và dữ liệu thật trong database nguồn;
2. tìm toàn bộ seed/default/bootstrap module đang tồn tại trong WPF;
3. loại bỏ trùng lặp và xác định một canonical baseline seed plan;
4. thiết kế toàn bộ baseline seed chạy trong **một PostgreSQL transaction duy nhất**;
5. giữ Phase 1 credential/checkpoint nguyên vẹn.

Database tham khảo do operator chỉ định:

```text
enailsalon_phasee1_pos1_pg
```

Database này chứa dữ liệu thật. Đây là **reference database read-only**, không phải target để seed.

## Mục tiêu của prompt012

Tạo một audit/design report đủ chi tiết để prompt kế tiếp có thể implement Phase 2 an toàn.

Task này chỉ điều tra và thiết kế. Không được:

- tạo database mới;
- sửa database tham khảo;
- chạy migration;
- seed bất kỳ database nào;
- sửa source WPF/API;
- thay đổi Phase 1 ProductRoot/checkpoint/DPAPI credential;
- bắt đầu Phase 2 runtime.

## Phase 1 freeze boundary

Phải giữ nguyên:

```text
ProductRoot:
E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot

Phase 1 ApiAuthorized checkpoint
DPAPI-protected WpfJwt
LocalInstallationGuid
Tenant/POS/InstallationAttempt identity
```

Không xóa, rotate, overwrite hoặc di chuyển các artifact trên.

Trước audit, chỉ kiểm tra presence và trạng thái an toàn; không in raw token, credential hoặc Pairing Code.

## Source boundaries cần audit read-only

```text
E:\Project2026\4POS\NailSalonNet8
E:\Project2026\4POS\NailSalonNet8.Tests
E:\Project2026\CanonicalInstallationDocs
E:\Project2026\RecoveryReports
```

Tập trung vào:

- seed modules;
- default-data initializers;
- bootstrap/provisioning code;
- settings/parameters/printer/role initialization;
- database creation/migration/initialization call chain;
- `TblLocalOutbox` emission patterns;
- seed completion/version markers hiện có;
- tests liên quan.

Không audit sâu các runtime transaction modules không thuộc baseline seed.

## Reference database read-only rules

Database:

```text
enailsalon_phasee1_pos1_pg
```

Bắt buộc:

1. Chỉ dùng read-only connection/session.
2. Nếu dùng SQL transaction, dùng read-only transaction và kết thúc bằng rollback/no mutation.
3. Không `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`, `ALTER`, `CREATE`, `DROP`, `GRANT`, `REVOKE`, `VACUUM FULL`, sequence mutation hoặc function có side effect.
4. Không in connection string, password hoặc credential.
5. Không export raw customer/business data.
6. Không đưa dữ liệu thật vào public GitHub report.

Cho phép report:

- table/schema names;
- row counts;
- FK/unique/index/default metadata;
- safe configuration key names;
- sanitized value categories;
- non-PII canonical defaults.

Không được đưa vào report:

- customer names/phones/emails;
- employee personal data;
- invoice/output/payment rows;
- gift-card numbers/balances;
- booking data;
- terminal credentials;
- salon-private values không cần thiết;
- raw row dumps.

Nếu cần detailed row-level evidence để quyết định seed, giữ local-only trong folder versioned:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2SeedAuditV001\
```

Public report chỉ ghi path, file hash và sanitized conclusion; không commit local detailed evidence.

## Known operator seed policy cần kiểm chứng

Các định hướng đã được operator xác nhận, nhưng Codex phải đối chiếu với schema/code trước khi chốt:

### Mandatory baseline candidates

- application settings;
- system parameters;
- printer defaults;
- default roles:
  - Owner
  - Admin
  - SubAdmin

### Không seed trong initial baseline

- employees/staff;
- services/service categories;
- customers;
- gift cards;
- bookings/appointments;
- invoices;
- `TblOutputInfo` và related runtime output tables;
- terminal payment/runtime transaction tables;
- turn/runtime state tables;
- event delivery/log operational data.

Các bảng trên chỉ cần schema/count/classification đủ để xác nhận loại trừ; không cần điều tra sâu dữ liệu thật.

Mục tiêu baseline dự kiến dưới khoảng 30 tables, nhưng không ép số lượng. Chỉ chọn table thật sự cần để app khởi động và mở UI an toàn.

## Audit database schema bắt buộc

Đối với mọi candidate baseline table, report phải có:

```text
Table
Purpose
Current reference row count
Primary key
Foreign keys
Unique constraints
Required/non-null columns
Server defaults
Identity/sequence behavior
Tenant/Pos scoping columns
Current code entity/model
Seed source module(s)
Candidate decision
Reason
```

Phân loại mỗi table vào đúng một nhóm:

```text
A. Mandatory baseline seed
B. Required lookup/reference seed
C. Conditional plan-specific seed
D. User-created/imported later
E. Runtime/transactional — explicitly excluded
F. Unknown/defer pending operator decision
```

## Audit dữ liệu tham khảo bắt buộc

Với candidate A/B/C:

- xác định safe canonical row patterns;
- phân biệt giá trị global, tenant-scoped, POS-scoped và machine-local;
- xác định cột nào phải generate mới;
- xác định cột nào có thể lấy từ Phase 1 identity;
- xác định cột nào là environment-specific và không được copy từ reference DB;
- xác định cột nào là secret/private và không được seed;
- xác định row nào chỉ là dữ liệu lịch sử của salon và không phải default.

Không copy nguyên dữ liệu thật sang manifest chỉ vì reference DB đang có row.

## Audit WPF seed modules bắt buộc

Tìm toàn bộ code path có ý nghĩa seed/default/bootstrap, bao gồm các từ khóa tương đương:

```text
Seed
Seeder
Initialize
Initializer
Bootstrap
Provision
Default
EnsureCreated
CreateDefaults
SetupDefaults
SystemBaseline
PrinterDefaults
Parameters
Settings
Roles
LocalOutbox
```

Với mỗi module/method, report:

```text
Full path
Class/type
Method
Current caller/call chain
Tables touched
Transaction behavior
Idempotency behavior
Outbox behavior
Duplicate/overlap with other modules
Safe to reuse / must refactor / obsolete / unknown
```

Phải lập duplicate/conflict matrix:

```text
Seed concern | Module A | Module B | Same tables/rows? | Conflict risk | Canonical owner đề nghị
```

## Source-of-truth comparison matrix

Đối với mỗi baseline concern, so sánh:

```text
Concern
Reference DB pattern
Current WPF seed code
Current hard-coded/default value
Conflict/difference
Proposed canonical source
Operator decision needed?
```

Ví dụ concern:

- settings;
- parameters;
- printer defaults;
- roles;
- plan metadata;
- machine/POS identity references;
- outbox events;
- seed version marker.

## One-transaction Phase 2 design bắt buộc

Report phải đề xuất **một transaction boundary duy nhất** cho baseline seed.

Transaction dự kiến phải bao gồm, nếu schema/code xác nhận cần:

```text
BEGIN
  verify Phase 1 prerequisite identity
  verify target DB eligibility/empty-or-compatible state
  insert canonical baseline rows in FK-safe order
  insert required TblLocalOutbox rows in same transaction
  write seed version/completion marker in same transaction
  validate expected row counts/invariants
COMMIT
```

Bất kỳ lỗi nào:

```text
ROLLBACK toàn bộ
không để partial seed
không đổi Phase 1 checkpoint
không đánh dấu Phase 2 complete
```

Report phải xác định rõ:

1. Schema/migrations là prerequisite bên ngoài seed transaction hay nằm trong boundary nào.
2. Exact insert order theo FK dependency.
3. Row-level idempotency keys.
4. Seed manifest/version key.
5. Existing-state behavior:
   - empty DB;
   - same seed version already complete;
   - partial rows;
   - conflicting rows;
   - newer seed version.
6. Rollback behavior.
7. Retry behavior.
8. Concurrency/second-instance lock strategy.
9. Which rows require `TblLocalOutbox` and why.
10. Whether outbox payload/map code already exists and can be reused.
11. How to prove no operational/runtime tables were seeded.

Không được đề xuất `git add .`, destructive reset hoặc manual partial inserts.

## Dependency graph bắt buộc

Report phải có Mermaid diagram thể hiện:

```text
Phase 1 authorized checkpoint
-> Phase 2 preflight
-> target DB eligibility
-> single transaction
-> ordered baseline seed groups
-> TblLocalOutbox
-> seed marker
-> commit/rollback
```

Đồng thời có table-level dependency DAG hoặc ordered list đủ chính xác để implementation.

## Seed manifest proposal

Report phải đề xuất manifest version đầu tiên, ví dụ:

```text
phase2-baseline-seed-v001
```

Nhưng chưa tạo/commit implementation manifest vào source.

Manifest proposal phải có:

```text
version
candidate tables
row groups
identity source
value source
FK order
outbox policy
idempotency key
expected row count/range
verification rule
excluded tables
```

Mọi artifact mới phải versioned; không overwrite artifact cũ.

## Safety / privacy

Coordination repository là Public.

Không commit:

- raw DB exports;
- source code dumps;
- customer/employee/business data;
- connection strings;
- passwords;
- tokens;
- Pairing Codes;
- private keys;
- terminal credentials.

Relevant code evidence trong report chỉ dùng path/class/method và sanitized pseudocode hoặc đoạn ngắn cần thiết.

## Prompt label rule

Prompt012 là read-only audit và **không được sửa WPF**, vì vậy:

```text
WPF build label vẫn giữ prompt011
```

Không đổi label sang prompt012.

Nếu Codex thấy cần sửa WPF để audit, phải dừng và báo blocker; không tự sửa.

## Runtime/process rule

Không stop/restart ApiServer, PlatformAppV0 hoặc WPF do operator đang có thể dùng runtime hiện tại.

Không cần build/test toàn solution vì task không sửa source.

Chỉ thực hiện read-only inspection. Nếu một command có nguy cơ lock/mutate, không chạy.

## Report 012 — bắt buộc 100% chi tiết

Tạo đúng:

```text
report/report012.md
```

Đây là database/high-risk design audit nên report đầu tiên phải 100% evidence, không dùng mức 80%.

Report tối thiểu gồm:

1. Verdict.
2. Phase 1 freeze verification.
3. Reference database read-only proof.
4. Schema/table inventory.
5. Candidate table classification A–F.
6. Safe reference row-pattern findings.
7. Complete WPF seed module inventory.
8. Seed module call-chain map.
9. Duplicate/conflict matrix.
10. Source-of-truth comparison matrix.
11. Proposed canonical baseline table list.
12. Explicit excluded table list.
13. FK/dependency order.
14. One-transaction design.
15. Idempotency/version/rollback/concurrency design.
16. `TblLocalOutbox` policy.
17. Seed manifest v001 proposal.
18. Mermaid transaction flow.
19. Operator decisions still required.
20. Exact implementation scope for prompt013.
21. Confirmation no DB/source/runtime mutation.
22. Local-only evidence paths/hashes if created.
23. Coordination commit SHA in final response.

## Acceptance criteria

Prompt012 PASS khi:

- reference DB was inspected read-only;
- no real/private rows were leaked;
- all existing WPF seed modules were inventoried;
- duplicate seed responsibilities were identified;
- baseline candidate tables and exclusions are explicit;
- one-transaction boundary is concrete;
- FK order, idempotency, rollback, retry, outbox and marker policies are concrete;
- Phase 1 artifacts remain untouched;
- no source/DB/runtime mutation occurred;
- report is detailed enough to write prompt013 implementation without guessing.

## Verdict hợp lệ

Nếu audit hoàn tất:

```text
PHASE2_BASELINE_SEED_AUDIT_READY_FOR_IMPLEMENTATION_PROMPT
```

Nếu thiếu schema/access/evidence:

```text
BLOCKED_PHASE2_BASELINE_SEED_AUDIT
```
