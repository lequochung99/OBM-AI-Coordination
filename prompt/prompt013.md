# Prompt 013 — Hoàn tất Phase 2 live read-only baseline seed audit

## Trạng thái trước prompt013

Đọc đầy đủ:

```text
prompt/prompt012.md
report/report012.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Prompt012 đã hoàn tất phần source/document audit nhưng bị block tại live PostgreSQL inspection:

```text
BLOCKED_PHASE2_BASELINE_SEED_AUDIT
```

Lý do duy nhất còn lại:

```text
Không có credential path local được operator chuẩn bị để role read-only kết nối database enailsalon_phasee1_pos1_pg.
```

Prompt013 chỉ hoàn tất phần DB audit còn thiếu và chốt thiết kế. Chưa implement Phase 2 seed.

## Operator-provided credential boundary

Operator sẽ tự tạo một temporary local `PGPASSFILE` và restart Codex để process mới nhận environment variable:

```text
PGPASSFILE=<local secret file path>
```

Codex tuyệt đối không được:

- hỏi operator cung cấp password trong chat;
- mở hoặc đọc nội dung file `PGPASSFILE`;
- in path đầy đủ của secret file vào public report nếu path chứa username/private folder detail;
- in password, connection string, token hoặc secret;
- copy credential file;
- commit credential file;
- sửa password PostgreSQL;
- thử fallback sang role `postgres`.

Codex chỉ được kiểm tra:

```text
PGPASSFILE environment variable present = true|false
Referenced file exists = true|false
File content = NEVER READ / NEVER PRINT
```

Database access phải dùng existing runtime role:

```text
hung
```

Known safety expectation: role `hung` chỉ dùng cho WPF runtime/read-only inspection; không thay đổi password, ownership hoặc privileges.

Nếu `PGPASSFILE` không tồn tại, role `hung` không connect được, hoặc role thiếu SELECT cần thiết:

```text
BLOCKED_PHASE2_READONLY_DATABASE_ACCESS
```

Không fallback sang administrator credentials và không mutate privileges.

## Database target và hard read-only guard

Reference database:

```text
enailsalon_phasee1_pos1_pg
```

Connection target:

```text
Host: 127.0.0.1
Port: 5432
User: hung
Database: enailsalon_phasee1_pos1_pg
```

Mọi `psql` invocation phải dùng:

```text
-X
-v ON_ERROR_STOP=1
PGOPTIONS=-c default_transaction_read_only=on
```

Mọi SQL evidence batch phải bắt đầu/kết thúc:

```sql
BEGIN TRANSACTION READ ONLY;
SET LOCAL statement_timeout = '30s';
SHOW transaction_read_only;
SELECT current_database(), current_user;
-- read-only metadata/query work
ROLLBACK;
```

Trước query chính, chứng minh:

```text
transaction_read_only = on
current_database = enailsalon_phasee1_pos1_pg
current_user = hung
```

Cấm tuyệt đối:

```text
INSERT UPDATE DELETE TRUNCATE ALTER CREATE DROP
GRANT REVOKE
COPY ... TO PROGRAM
CREATE TEMP TABLE
setval / nextval
functions/procedures có side effect
VACUUM / ANALYZE mutation-oriented commands
migration / seed / EnsureCreated
```

Không thay đổi reference DB dù chỉ một row hoặc sequence value.

## Phase 1 freeze

Giữ nguyên và chỉ kiểm tra presence:

```text
E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
ApiAuthorized checkpoint
DPAPI-protected WpfJwt
LocalInstallationGuid
Tenant/POS/InstallationAttempt identity
```

Không mở raw protected token, không rotate, không delete, không overwrite.

## Local-only evidence folder

Tạo version mới, không overwrite artifact khác:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2SeedAuditV002\
```

Cho phép lưu local-only:

```text
schema-metadata.tsv
candidate-table-counts.tsv
constraints.tsv
fk-dependency.tsv
safe-candidate-patterns.tsv
seed-module-crosscheck.md
AUDIT-MANIFEST.json
SHA256SUMS.txt
```

Không commit folder này lên public coordination repository.

Mọi file local evidence phải:

- UTF-8;
- không chứa password/connection string;
- không chứa raw customer/employee/invoice/gift-card/booking/payment/output rows;
- không chứa terminal credential/private config value;
- không chứa Pairing Code/WpfJwt;
- có SHA-256 trong `SHA256SUMS.txt`.

## Live schema inventory bắt buộc

Audit all schemas relevant to WPF mappings, especially `public` and/or `dbo` as physically present.

Đối với candidate table, thu thập:

```text
schema
physical table name
estimated and exact row count where safe
primary key columns
foreign keys and referenced tables
unique constraints/indexes
all columns and PostgreSQL types
nullable/non-null
server default expressions
identity/sequence behavior
check constraints
TenantGuid/PosGuid/PosStationId/local-machine scoping columns
```

Candidate table set từ report012 cần xác nhận live:

```text
TblSystemBaselineVersion
TblSetting
TblParameterSetting
TblSetupWeird
TblSetupServicesMethod
TblSetupLoginMethod
TblSetupPaymentMethod
TblSetupPrinter
TblEmployeePermission
TblTenant
TblPosLocal
TblLocalOutbox
TblTurnSetting (conditional investigation only)
```

Cũng xác nhận existence/count/classification, không đọc business rows, cho excluded groups:

```text
TblEmployee*
TblServiceCategory
TblService
TblProduct
TblCustomer*
TblGiftCard*
TblInvoice*
TblOutputInfo*
booking/appointment tables
terminal payment tables
queue/turn/payroll runtime tables
event/delivery operational tables
```

## Safe candidate row-pattern audit

Chỉ với candidate A/B/C, xác định canonical pattern cần thiết để seed.

Cho phép public report ghi:

- stable key names không nhạy cảm;
- boolean/numeric defaults;
- safe enum/code labels;
- row counts;
- global/tenant/POS/machine scope;
- value source category.

Không được public raw salon-specific values.

Local-only evidence vẫn phải redact mọi column/value có tên hoặc ý nghĩa tương tự:

```text
Password
Pin
Secret
Token
Key
Credential
ConnectionString
Terminal
Merchant
ApiKey
Private
Customer
Phone
Email cá nhân
GiftCard number/balance
```

Đối với printer rows:

- xác định row shape và required columns;
- không copy physical printer name/IP/driver từ reference DB;
- phân loại field nào placeholder-safe và field nào machine binding later.

Đối với roles/permissions:

- xác định physical table shape và stable key;
- xác định liệu `Owner`, `Admin`, `SubAdmin` có thể biểu diễn không cần seed employee;
- không copy employee identifiers/PIN.

Đối với parameters/settings:

- lập list safe key names;
- phân loại mandatory vs historical/salon-specific;
- không lấy value nhạy cảm.

## Source-to-live comparison bắt buộc

Dùng report012 module inventory rồi đối chiếu live schema:

```text
Seed concern
Live table/constraint pattern
Legacy WPF seed method
Rows currently hard-coded
Duplicate/omission/conflict
Canonical Phase 2 owner
Reuse/refactor/exclude decision
```

Đặc biệt xác minh:

```text
SeedParameterSetingAsync coverage gap
SeedEmployeePermissionAsync role compatibility
SeedSetupPrinterAsync machine-specific risk
SeedPaymentMethodsAsync / SeedSetupLoginMethodAsync required rows
TblSystemBaselineVersion suitability
TblTenant / TblPosLocal Phase 1 identity mapping
CreateLocalOutboxSingleAsync payload compatibility
```

## Final candidate classification A–F

Không để provisional sau prompt013. Mỗi table phải được chốt:

```text
A Mandatory baseline seed
B Required lookup/reference seed
C Conditional plan/machine seed
D User-created/imported later
E Runtime/transactional explicitly excluded
F Deferred with exact blocker/operator decision
```

Mỗi A/B/C phải có:

```text
stable row key
identity/value source
expected row count/range
FK order
idempotency rule
outbox policy
verification rule
```

## One-transaction design phải được chốt

Chốt transaction boundary cho implementation prompt sau:

```text
Phase 1 prerequisite revalidation
Target DB eligibility/schema prerequisite
Acquire PostgreSQL advisory transaction lock
BEGIN one transaction
Validate seed manifest/version
Insert/upsert canonical rows in exact FK order
Insert deterministic TblLocalOutbox rows in same DbContext transaction
Write Phase 2 completion marker last
Read back stable keys/counts/invariants
COMMIT
```

Failure:

```text
ROLLBACK all baseline rows
ROLLBACK all outbox rows
ROLLBACK marker
Phase 1 checkpoint unchanged
Phase 2 remains NotStarted/FailedRetryable
```

Chốt:

- exact table insertion order;
- schema/migration prerequisite boundary;
- advisory lock key strategy;
- behavior for empty DB;
- same version already complete;
- partial state;
- conflicting state;
- newer version present;
- retry after rollback;
- outbox deterministic idempotency key;
- seed completion marker table/version.

## Seed manifest proposal v001

Hoàn tất proposal:

```text
phase2-baseline-seed-v001
```

Manifest proposal phải chứa exact A/B/C tables, stable row keys, expected counts, FK order, value source, outbox and verification policy.

Chưa tạo implementation manifest trong WPF source.

## Prompt label/runtime rule

Prompt013 không sửa WPF:

```text
WPF build label remains prompt011
```

Không stop/restart ApiServer, PlatformAppV0 hoặc WPF.
Không build/test solution nếu không sửa source.

## Report 013 — 100% evidence

Tạo đúng:

```text
report/report013.md
```

Report phải có:

1. Verdict.
2. PGPASSFILE precondition evidence without path/content disclosure.
3. Read-only transaction proof.
4. Role `hung` privilege/read-only proof.
5. Live schema/table/row-count inventory.
6. PK/FK/unique/default/non-null metadata for candidates.
7. Final A–F classification.
8. Safe candidate row-pattern findings.
9. Source-to-live module comparison.
10. Duplicate/conflict/coverage-gap conclusions.
11. Final canonical baseline table list.
12. Explicit exclusions.
13. Exact FK insert order.
14. Final one-transaction design.
15. Idempotency/version/rollback/retry/concurrency policy.
16. Final `TblLocalOutbox` policy.
17. `phase2-baseline-seed-v001` manifest proposal.
18. Mermaid flow/DAG.
19. Remaining operator decisions, if any.
20. Exact implementation scope for prompt014.
21. Phase 1 freeze proof.
22. Confirmation no DB/source/runtime mutation.
23. Local evidence folder/file hashes.
24. Coordination commit SHA in final response.

Public report must remain sanitized.

## Acceptance verdicts

PASS:

```text
PHASE2_BASELINE_SEED_AUDIT_READY_FOR_IMPLEMENTATION_PROMPT
```

Blocked access:

```text
BLOCKED_PHASE2_READONLY_DATABASE_ACCESS
```

Blocked unresolved design:

```text
BLOCKED_PHASE2_BASELINE_SEED_DESIGN_UNRESOLVED
```

Do not implement Phase 2 in prompt013.
