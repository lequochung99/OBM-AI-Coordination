# prompt149 — PROMPT305E-FIX Isolate And Commit EntitiesService Transaction-Group Path

Date: 2026-08-06
Executor: Cursor

Goal: Unblock V014 manual install prep (`BLOCKED_V014_CRITICAL_FIX_UNCOMMITTED` from
report148) by making the API transaction-group process/apply path + Tenant/PosLocal
ensure-missing call sites committed, reviewable, and repeatable — source isolation/commit
gate only, no V014 DB/API/pairing/WPF prep in this prompt.

Local report:
`E:\Project2026\RecoveryReports\V014\Prompt305E_FixTransactionGroupIsolation\V001\PROMPT305E_FIX_TRANSACTION_GROUP_ISOLATION_REPORT.md`
