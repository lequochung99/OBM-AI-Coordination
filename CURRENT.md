# OBM AI Coordination — Current Entry Point

This file is the stable human-readable entry point. Machine-readable task state lives in `state/CURRENT_TASK.json`.

## Current operating model

```text
ChatGPT -> classify and define one complete task
Cursor  -> investigate targeted scope, implement, validate, commit, report
Codex   -> hard/critical implementation or independent review when available
Operator -> UI and physical runtime acceptance
Graphify -> current code-navigation map
```

## Required pointers

- Constitution: [`constitution/CURRENT.md`](constitution/CURRENT.md)
- Active task state: [`state/CURRENT_TASK.json`](state/CURRENT_TASK.json)
- Active task pointer: [`tasks/active/CURRENT.md`](tasks/active/CURRENT.md)
- Database context: [`context/database/README.md`](context/database/README.md)
- Graphify pointer: [`graphify/CURRENT.md`](graphify/CURRENT.md)

## Current task

`PROMPT305E-FIX — Isolate And Commit EntitiesService Transaction-Group Path` — **PASS**

Legacy prompt/report:

- [`prompt/prompt149.md`](prompt/prompt149.md)
- [`report/report149.md`](report/report149.md)

Verdict: `TRANSACTION_GROUP_SOURCE_ISOLATION_COMMIT_PASS`

Local artifact (SHA256 `82C15F7602E449488FC5F140841A64EF2A91C0D5604A96E5132DC27B611DA02A`):

`E:\Project2026\RecoveryReports\V014\Prompt305E_FixTransactionGroupIsolation\V001\PROMPT305E_FIX_TRANSACTION_GROUP_ISOLATION_REPORT.md`

Source commit: `54b408b` on `recovery/wpf-installation-reset-cursor-v001` (local-only / no origin).

`BLOCKED_V014_CRITICAL_FIX_UNCOMMITTED` (report148) is cleared. Next: retry
`PROMPT305E — Prepare V014 Manual WPF Installation Observation Lane` (clean V014 DB/API prep,
pairing, WPF config, watch scripts, operator pause).
