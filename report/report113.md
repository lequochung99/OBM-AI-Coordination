# Report 113

Verdict: BLOCKED_MAIN_DEV_API_ROLE_CONTRACT

Prompt112 artifact SHA verified: yes
Exact 42501 failed command/method: dotnet ef migrations list / DesignTimeExternalDbContextFactory.CreateDbContext
Exact inaccessible object/schema: public.__EFMigrationsHistory
Failure classification A-G: C plus role-contract defect
Runtime role is superuser/owner: no superuser, not object owner; CREATEDB is incorrectly enabled
Canonical migration/provisioning role identified: no
Grant/provisioning source-of-truth corrected: no
Broad GRANT ALL used: no
Runtime SELECT migration history required by actual startup: no, only proof command observed
Migration history proof: no
API pending migrations count: not proven
Runtime positive application permission proof: no
Runtime negative DDL/admin permission proof: no; runtime role has CREATEDB
Canonical API DB runtime proof: yes
API startup non-interactive: yes
ApiServer loopback readiness: yes

Production sync auth physically passed: no
Development prerequisites created: no
Production Price Rule Save used: no
Local domain + outbox atomic commit: no
Local group expected/actual count: expected 2 / actual 0
Local group sequence/order proof: no
Existing periodic WPF worker used: no
Production worker cycle count: 0
Grouped HTTP request count: 0
Canonical API ingest action used: no
API transaction begin count: 0
API durable commit count: 0
TblEventLog group row count: 0
TblEventDelivery destination row count: 0
TblEventDelivery source row count: 0
Source exclusion proof: no
SignalR attempted after commit: no
SignalR publish succeeded: no
Destination notification observed: not-applicable
WPF group rows marked Sent count: 0
All-or-none local completion proof: no
Parallel production path invoked count: 0
Happy-path marker present: no

Focused test totals: 3 passed, 2 failed, 0 skipped
Architecture guard totals: not rerun after blocker; prompt110 baseline 5 passed, 0 failed, 0 skipped
WPF build totals: not run in prompt113
API build totals: not run in prompt113

WPF/API DB reset performed: no
Manual outbox/event/delivery insertion used: no
Production/customer/reference DB mutated: no

Private artifact: yes
Aggregate SHA-256: b7460bc0e6a6fda241537af758524ea17a9daf4d37942828bac92aca04fb11cd

