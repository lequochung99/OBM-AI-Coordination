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

`TASK-SETUP-COST-FEE-MERCHANT-SEED-V001` — **IMPLEMENTED_REPORT_READY**

Task packet:

`tasks/active/TASK-SETUP-COST-FEE-MERCHANT-SEED-V001/TASK.md`

Report:

`report/report144.md`

DB context:

`context/database/TblSetupCostAndFeeMerchant/V001/DB_CONTEXT.md`

Verdict: `WPF_SETUP_COST_FEE_MERCHANT_CANONICAL_INITIAL_SEED_PASS`. ChatGPT review next; operator owns UI acceptance.

