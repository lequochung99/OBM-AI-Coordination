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

`PROMPT305E — V014 Manual Install Observation` — **BLOCKED**

Legacy prompt/report:

- [`prompt/prompt148.md`](prompt/prompt148.md)
- [`report/report148.md`](report/report148.md)

Verdict: `BLOCKED_V014_CRITICAL_FIX_UNCOMMITTED`

Local artifact (SHA256 `E54FFDD84E23C76CB4CE85634AD74315E4E34DC62D879B6DD557BBF2E2E7F31F`):

`E:\Project2026\RecoveryReports\V014\Prompt305E_ManualInstallObservation\V001\PROMPT305E_V014_MANUAL_INSTALL_OBSERVATION_REPORT.md`

Next: isolate/commit `EntitiesService` transaction-group + ensure-missing call sites, then retry V014 prep for operator manual install.
