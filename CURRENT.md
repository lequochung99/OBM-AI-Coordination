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

The existing `prompt/promptXXX.md` and `report/reportXXX.md` convention remains canonical and must not be renumbered or rewritten. If the local prompt/report sequence is ahead of the remote coordination branch, resolve the current number from the operator/local repository rather than guessing.

## Current task

`PROMPT305B — Prepare Clean V013 WPF/API Lane` — **IMPLEMENTED_REPORT_READY**

Legacy prompt/report:

- [`prompt/prompt145.md`](prompt/prompt145.md)
- [`report/report145.md`](report/report145.md)

Lane:

- WPF DB: `obm_pos_dev_v013_pg`
- API DB: `obm_api_dev_v013_pg`

DB context:

[`context/database/V013Lane/CURRENT.md`](context/database/V013Lane/CURRENT.md)

Local artifact (SHA256 `88E644BD6BDDF228ADAFB39BC15C563F8B92A68A9DB3CFE902B36C5591B16B5C`):

`E:\Project2026\RecoveryReports\V013\Prompt305B_PrepareCleanV013Lane\V001\PROMPT305B_PREPARE_CLEAN_V013_LANE_REPORT.md`

Local WPF/API source commits (no origin): `2502a17`, `7838685` on `recovery/wpf-installation-reset-cursor-v001`.

Verdict: `V013_CLEAN_LANE_PREP_PASS`. Operator owns UI acceptance. Next: full InstallationV0 baseline seed/drain on V013 (recommended PROMPT305C).
