# Report 112

Verdict: BLOCKED_MAIN_DEV_API_STARTUP

Prompt111 artifact SHA verified: yes
Exact startup script/profile: 1ApiServer/ApiServer01/start-api-local.ps1 / LocalDevelopment
Exact credential/configuration failure classification: protected credential present but carried noncanonical DB name; startup script dropped it and reached hidden prompt
Protected PostgreSQL runtime credential source name only: ConnectionStrings__PostgreSqlConnection from central env import
Hidden password prompt reachable after task: no for automation; only explicit operator -InteractiveCredentialPrompt

Canonical API DB runtime proof: yes
API startup non-interactive: yes
ApiServer loopback readiness: yes
API pending migrations count: not proven; SQLSTATE 42501 reading EF history table

Production sync auth physically passed: no
Development prerequisites created: no
Production Price Rule Save used: no
Local domain + outbox atomic commit: no
Local group expected/actual count: expected 2 for clean first-save case / actual 0
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

Focused test totals: 3 passed, 1 failed, 0 skipped
Architecture guard totals: not rerun after blocker; prompt110 baseline 5 passed, 0 failed, 0 skipped
WPF build totals: passed, 172 warnings, 0 errors
API build totals: passed, 0 warnings, 0 errors

WPF/API DB reset performed: no
Manual outbox/event/delivery insertion used: no
Production/customer/reference DB mutated: no

Private artifact: yes
Aggregate SHA-256: 6cf0b248fe2888e5605b54c22bcb09d8a3d0bc3acf4e2efa913cf7cd83e3a7e6


