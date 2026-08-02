# Report 114

Verdict: BLOCKED_MAIN_DEV_DESTINATION_ROUTING

Prompt107 artifact SHA verified: yes
Prompt108 artifact SHA verified: yes
Prompt113 artifact SHA verified: yes
Protected provisioning boundary recovered: yes
Protected credential source name only: OBM_PLATFORM_V2_P6_POS_PG_ADMIN
Prompt113 discovery gap classification: did not check accepted prompt107 admin/provisioning source
Canonical migration/provisioning role identified: yes
Grant/provisioning source-of-truth corrected: yes
Role contract applied idempotently: yes
Runtime role CREATEDB before/after: true / false
Runtime role CREATEROLE after: false
Runtime role SUPERUSER after: false
Runtime role object/database/schema owner after: no
Broad GRANT ALL used: no
Runtime SELECT on __EFMigrationsHistory granted: no; actual startup does not require it
Migration history proof: yes
API pending migrations count: 0
Runtime positive application permission proof: yes
Runtime negative DDL/admin permission proof: yes
Canonical API runtime proof: yes
ApiServer loopback readiness: yes
Admin/provisioning credential absent from normal runtime process: yes

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
Manual POS1 test ready: false

Focused test totals: 7 passed, 1 failed, 0 skipped
Architecture guard totals: not rerun after blocker; prompt110 baseline 5 passed, 0 failed, 0 skipped
WPF build totals: not run in prompt114
API build totals: not run in prompt114

WPF/API DB reset performed: no
Manual outbox/event/delivery insertion used: no
Category Weight changed: no
Booking Weight changed: no
Production/customer/reference DB mutated: no

Private artifact: yes
Aggregate SHA-256: 29013015c51314c63748cb174754a1571e3677ecd1854d4089a70ed5c1a7e22c

