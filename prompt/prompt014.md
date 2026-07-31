# Prompt 014 — Hoàn tất Phase 2 live DB audit bằng fixed local PGPASS path

## Trạng thái

Đọc đầy đủ:

```text
prompt/prompt012.md
report/report012.md
prompt/prompt013.md
report/report013.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Prompt013 bị block chỉ vì Codex process không inherit user-level environment variable `PGPASSFILE`.

Operator đã chứng minh bên ngoài Codex rằng local read-only access hoạt động:

```text
transaction_read_only = on
current_database = enailsalon_phasee1_pos1_pg
current_user = hung
```

Prompt014 phải tránh phụ thuộc vào process inheritance. Nó được phép tự đặt process-local `PGPASSFILE` bằng fixed local path đã được operator tạo, nhưng tuyệt đối không được đọc nội dung file.

Prompt014 vẫn là read-only audit/design task. Không implement Phase 2 seed.

## Fixed local credential path

Trong PowerShell process do Codex chạy, dùng chính xác:

```powershell
$pgpassPath = Join-Path $env:LOCALAPPDATA "OBM\Phase2SeedAudit\pgpass.conf"
```

Bắt buộc:

```powershell
$pgpassExists = Test-Path -LiteralPath $pgpassPath -PathType Leaf
```

Nếu file không tồn tại:

```text
BLOCKED_PHASE2_READONLY_DATABASE_ACCESS
```

Nếu tồn tại, set **process-local only**:

```powershell
$env:PGPASSFILE = $pgpassPath
$env:PGOPTIONS = "-c default_transaction_read_only=on"
```

Không được:

- `Get-Content`, `type`, `cat`, `more`, `gc` hoặc mở file;
- đọc/hash/copy nội dung file;
- print expanded full path trong public report;
- print password/connection string;
- commit file;
- thay đổi ACL;
- đổi password;
- fallback sang `postgres` hoặc admin role.

Public report chỉ ghi:

```text
Fixed local pgpass path resolved = true|false
PGPASSFILE process-local set = true|false
Credential file content read = false
```

## PostgreSQL executable resolution

Ưu tiên executable trong PATH:

```powershell
$psql = (Get-Command psql -ErrorAction SilentlyContinue).Source
```

Nếu không có, dùng:

```text
C:\Program Files\PostgreSQL\18\bin\psql.exe
```

Nếu cả hai không tồn tại:

```text
BLOCKED_PHASE2_READONLY_DATABASE_ACCESS
```

Không cài package hoặc sửa PATH hệ thống.

## Connection boundary

Chỉ kết nối:

```text
Host: 127.0.0.1
Port: 5432
Database: enailsalon_phasee1_pos1_pg
User: hung
```

Mọi invocation phải dùng:

```text
-X
-v ON_ERROR_STOP=1
```

Mọi SQL batch phải chạy trong:

```sql
BEGIN TRANSACTION READ ONLY;
SET LOCAL statement_timeout = '30s';
SHOW transaction_read_only;
SELECT current_database(), current_user;
-- SELECT-only audit
ROLLBACK;
```

Bắt buộc chứng minh trước audit chính:

```text
transaction_read_only = on
current_database = enailsalon_phasee1_pos1_pg
current_user = hung
```

Nếu proof không khớp, dừng ngay.

## Cấm mutation tuyệt đối

Không chạy hoặc gọi trực tiếp/gián tiếp:

```text
INSERT UPDATE DELETE TRUNCATE ALTER CREATE DROP
GRANT REVOKE
nextval setval
CREATE TEMP TABLE
COPY TO PROGRAM
VACUUM ANALYZE
functions/procedures có side effect
migration EnsureCreated seed bootstrap mutation
```

Chỉ dùng SELECT đối với `pg_catalog`, `information_schema`, và candidate tables.

Không thay đổi reference database, sequence, privilege, schema hoặc row.

## Phase 1 freeze

Chỉ kiểm tra presence, không đọc raw secrets:

```text
E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
InstallationV0\Checkpoints\api-authorized.json
InstallationV0\Secrets\bootstrap-credential.dpapi
```

Không stop/restart WPF, ApiServer hoặc PlatformAppV0.
Không thay đổi ProductRoot/checkpoint/credential.

## Local evidence V003

Tạo mới, không overwrite V001/V002:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2SeedAuditV003\
```

Allowed files:

```text
READONLY-PROOF.txt
schema-metadata.tsv
candidate-table-counts.tsv
constraints.tsv
fk-dependency.tsv
safe-candidate-patterns.tsv
source-live-comparison.md
AUDIT-MANIFEST.json
SHA256SUMS.txt
```

Không commit local evidence lên public GitHub.

Evidence files không được chứa:

- password/connection string/pgpass content;
- raw customer, employee, invoice, payment, booking, gift-card, output rows;
- printer IP/name/driver cụ thể;
- PIN/token/secret/key/credential;
- Pairing Code/WpfJwt;
- salon-private values không cần thiết.

Tạo SHA-256 cho mọi evidence file, ngoại trừ `SHA256SUMS.txt` chính nó.

## Live schema audit bắt buộc

Xác định schema vật lý (`public`, `dbo`, hoặc khác) và exact table names.

Audit đầy đủ candidate set:

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
TblTurnSetting
```

Cho mỗi candidate, thu thập:

```text
schema/table
exact row count
PK
FK and referenced table/columns
unique constraints/indexes
column names/types
nullable/non-null
server defaults
identity/sequence behavior
check constraints
Tenant/POS/machine scoping columns
```

Đối với excluded groups, chỉ existence/count/classification; không đọc business rows:

```text
TblEmployee*
TblServiceCategory
TblService
TblProduct
TblCustomer*
TblGiftCard*
TblInvoice*
TblOutputInfo*
booking/appointment
terminal payment
queue/turn/payroll runtime
event/delivery operational
```

## Safe row-pattern audit

Chỉ candidate A/B/C.

Cho phép thu thập/report:

```text
safe stable key names
boolean/numeric defaults
non-sensitive enum/code labels
row count
scope category: global/tenant/POS/machine
required versus historical/salon-specific classification
```

Không export raw rows.

### Settings/parameters

- liệt kê safe key names;
- phân loại mandatory, optional, historical, salon-specific, secret/private;
- chỉ report safe canonical values;
- không report terminal/payment credentials hoặc private URLs.

### Roles/permissions

- xác định table shape/stable key;
- xác định khả năng seed `Owner`, `Admin`, `SubAdmin` không cần employee row;
- không đọc/report employee/PIN.

### Printer defaults

- xác định required row shape;
- không copy printer name, IP, driver hoặc machine-specific value;
- phân loại placeholder-safe versus bind-later fields.

### Tenant/POS

- xác định exact FK/required fields;
- source identity phải là Phase 1 checkpoint/API identity, không copy tenant/POS identity từ reference DB.

### Outbox

- xác định exact columns, constraints and stable idempotency possibilities;
- chỉ inspect safe schema/payload metadata;
- không dump private payloads.

## Source-to-live synthesis

Không cần lặp lại toàn bộ source scan. Dùng report012/013 làm source inventory rồi đối chiếu live DB:

```text
Seed concern
Live schema/pattern
Legacy WPF seed method
Hard-coded coverage
Gap/conflict
Reuse/refactor/exclude
Canonical Phase 2 owner
```

Bắt buộc kết luận:

```text
SeedParameterSetingAsync coverage
SeedEmployeePermissionAsync compatibility
SeedSetupPrinterAsync machine-specific risk
SeedPaymentMethodsAsync necessity
SeedSetupLoginMethodAsync necessity
TblSystemBaselineVersion suitability
TblTenant/TblPosLocal requirement
CreateLocalOutboxSingleAsync compatibility
```

## Final A–F classification

Mỗi relevant table phải chốt một nhóm:

```text
A Mandatory baseline seed
B Required lookup/reference seed
C Conditional plan/machine seed
D User-created/imported later
E Runtime/transactional excluded
F Deferred with exact unresolved decision
```

Mỗi A/B/C phải có:

```text
stable row key
value/identity source
expected row count/range
FK insert order
idempotency rule
outbox rule
verification rule
```

## Final one-transaction design

Chốt design implementation-ready:

```text
Revalidate Phase 1 prerequisite
Verify target DB schema/eligibility
Acquire pg_advisory_xact_lock
BEGIN one transaction
Validate phase2-baseline-seed-v001 state
Insert/upsert approved baseline rows in exact FK order
Insert deterministic TblLocalOutbox rows in same transaction
Write completion marker last
Readback stable keys/counts/invariants
COMMIT
```

Failure:

```text
ROLLBACK baseline rows
ROLLBACK outbox rows
ROLLBACK marker
Phase 1 unchanged
Phase 2 not complete
```

Chốt exact behavior cho:

- empty target DB;
- same version complete;
- partial rows/no marker;
- conflicting rows;
- newer version marker;
- retry after rollback;
- second-instance concurrency;
- schema/migration prerequisite boundary.

## Final manifest proposal

Hoàn tất sanitized proposal:

```text
phase2-baseline-seed-v001
```

Không tạo implementation manifest trong WPF source.

Proposal phải có exact A/B/C table groups, stable keys, expected counts, FK order, value sources, outbox and verification rules, explicit exclusions.

## Prompt label/runtime rule

Prompt014 không sửa WPF:

```text
WPF label remains prompt011
```

Không build/test source.
Không stop/restart runtime.
Không source changes.

## Report 014 — 100% evidence

Tạo:

```text
report/report014.md
```

Bắt buộc gồm:

1. Verdict.
2. Fixed local pgpass resolution proof, no path/content disclosure.
3. Read-only transaction/current user/current DB proof.
4. Live schema/table/count inventory.
5. Candidate PK/FK/unique/default/non-null metadata.
6. Safe row-pattern findings.
7. Final A–F classification.
8. Source-to-live comparison.
9. Coverage-gap/conflict conclusions.
10. Final canonical baseline list.
11. Explicit exclusions.
12. Exact FK insert order.
13. Final one-transaction design.
14. Idempotency/version/rollback/retry/concurrency policy.
15. Final TblLocalOutbox policy.
16. Final phase2-baseline-seed-v001 proposal.
17. Mermaid flow/DAG.
18. Operator decisions still required.
19. Exact implementation scope for prompt015.
20. Phase 1 freeze proof.
21. No mutation/source/runtime change proof.
22. Local evidence V003 filenames and SHA-256.
23. Coordination commit SHA in final response.

Public report phải sanitized.

## Verdicts

PASS:

```text
PHASE2_BASELINE_SEED_AUDIT_READY_FOR_IMPLEMENTATION_PROMPT
```

Access blocked:

```text
BLOCKED_PHASE2_READONLY_DATABASE_ACCESS
```

Design unresolved:

```text
BLOCKED_PHASE2_BASELINE_SEED_DESIGN_UNRESOLVED
```

Không implement Phase 2 trong prompt014.
