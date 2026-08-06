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
- Agent status: [`state/AGENT_STATUS.md`](state/AGENT_STATUS.md)
- Module context: [`context/modules/README.md`](context/modules/README.md)
- Database context: [`context/database/README.md`](context/database/README.md)
- Investigation index: [`context/investigations/INDEX.md`](context/investigations/INDEX.md)
- Graphify pointer: [`graphify/CURRENT.md`](graphify/CURRENT.md)
- Architecture decisions: [`decisions/INDEX.md`](decisions/INDEX.md)
- Templates: [`templates/README.md`](templates/README.md)

## Legacy numbered workflow

The existing `prompt/promptXXX.md` and `report/reportXXX.md` convention remains canonical and must not be renumbered or rewritten.

## Current task

`PROMPT305D — V013 Catalog Sync + Booking Create` — **IMPLEMENTED_REPORT_READY**

Legacy prompt/report:

- [`prompt/prompt147.md`](prompt/prompt147.md)
- [`report/report147.md`](report/report147.md)

Lane:

- WPF DB: `obm_pos_dev_v013_pg`
- API DB: `obm_api_dev_v013_pg`

DB context:

[`context/database/V013Lane/CURRENT.md`](context/database/V013Lane/CURRENT.md)

Local artifact (SHA256 `5D24DB9B4538983CF2654DE94228EE36F2EC629B597631DD348D842C60E3D876`):

`E:\Project2026\RecoveryReports\V013\Prompt305D_CatalogSyncBookingCreate\V001\PROMPT305D_CATALOG_SYNC_BOOKING_CREATE_REPORT.md`

Source local commit: `b139aba` on `recovery/wpf-installation-reset-cursor-v001` (no origin). EntitiesService call sites still mixed WT.

Verdict: `V013_CATALOG_SYNC_BOOKING_CREATE_PASS`. Next: isolate remaining EntitiesService transaction-group commit / day-schedule 400 (PROMPT305E).
