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
- Agent status: [`state/AGENT_STATUS.md`](state/AGENT_STATUS.md)
- Module context: [`context/modules/README.md`](context/modules/README.md)
- Investigation index: [`context/investigations/INDEX.md`](context/investigations/INDEX.md)
- Graphify pointer: [`graphify/CURRENT.md`](graphify/CURRENT.md)
- Architecture decisions: [`decisions/INDEX.md`](decisions/INDEX.md)
- Templates: [`templates/README.md`](templates/README.md)

## Legacy numbered workflow

The existing `prompt/promptXXX.md` and `report/reportXXX.md` convention remains canonical and must not be renumbered or rewritten. New task packets may reference those numbered artifacts rather than duplicating their content.

## Current task

No active task is registered by this structural bootstrap. The next coordinator must update `state/CURRENT_TASK.json` atomically when activating a task.
