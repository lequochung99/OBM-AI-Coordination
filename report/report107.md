# Report 107

Verdict: OBM_MAIN_WPF_DEV_RESET_MIGRATION_READY_FOR_API_RESET

V001 aggregate SHA verified: yes

Exact failed maintenance method/command: guarded local Npgsql maintenance runner

Exact failed maintenance stage: maintenance connection open/auth/database selection

Protected credential source checked: OBM_PLATFORM_V2_P6_POS_PG_ADMIN

Sanitized exception type: Npgsql.NpgsqlException

PostgreSQL SQLSTATE: NOT_AVAILABLE

Physical target DB connection proof: yes

No-running-WPF-process classified stop-ready: yes

Maintenance database name sanitized: postgres

Maintenance database differs from target: yes

Maintenance connection proof: yes

Target session count before reset: 17

Target session count after close: 0

WPF Development reset executed: yes

WPF Development DB recreated: yes

UTF8 proof: yes

WPF migration chain applied from zero: yes

WPF pending migrations count: 0

Physical grouped schema proof: yes

Exact failed test result: pass

Focused test totals: 8 passed, 0 failed, 0 skipped

WPF build totals: 0 errors, 0 warnings

API DB mutated: no

Sync flow code changed: no

Production/customer/reference DB mutated: no

Private artifact: yes

Aggregate SHA-256: 47f68c634a5984611f3cb8b39ba3999f6005a558ad1e0d64bf998f7f4c2a0c58

