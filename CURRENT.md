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

`PROMPT305C — Full InstallationV0 Baseline Seed / Outbox Drain` — **IMPLEMENTED_REPORT_READY**

Legacy prompt/report:

- [`prompt/prompt146.md`](prompt/prompt146.md)
- [`report/report146.md`](report/report146.md)

Lane:

- WPF DB: `obm_pos_dev_v013_pg`
- API DB: `obm_api_dev_v013_pg`

DB context:

[`context/database/V013Lane/CURRENT.md`](context/database/V013Lane/CURRENT.md)

Local artifact (SHA256 `A4ACEC00403702254DA9999AA1DD8E090D6BAC8744B87F57515B958BD5C7DDAF`):

`E:\Project2026\RecoveryReports\V013\Prompt305C_FullInstallationBaselineOutboxDrain\V001\PROMPT305C_FULL_INSTALLATION_BASELINE_OUTBOX_DRAIN_REPORT.md`

Source: local-only @ `7838685` on `recovery/wpf-installation-reset-cursor-v001` (API Tenant ensure-missing in WT, not committed).

Verdict: `V013_FULL_BASELINE_SEED_OUTBOX_DRAIN_PASS`. Operator owns UI acceptance. Next: V013 catalog/booking probe (recommended PROMPT305D).
