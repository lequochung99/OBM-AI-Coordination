# Complete Task Packets

The numbered prompt remains the operator-facing execution instruction. A task packet provides compact machine-readable coordination around that prompt.

## Folder convention

```text
tasks/
  active/
    CURRENT.md
    PROMPTxxx/
      TASK.md
      STATUS.json
      CONTEXT_REFERENCES.md
  archive/
    PROMPTxxx/
      ...final task state...
```

Only one task should normally be active for a source workstream. Multiple independent workstreams must state their repository and branch boundaries explicitly.

## Task lifecycle

```text
DRAFT -> READY -> IN_PROGRESS -> IMPLEMENTED -> OPERATOR_CHECK -> COMPLETE
                           \-> BLOCKED
```

## Activation

Activating a task requires one coordinated change:

1. create `tasks/active/PROMPTxxx/`;
2. point `tasks/active/CURRENT.md` to it;
3. update `state/CURRENT_TASK.json`;
4. ensure `prompt/promptxxx.md` exists;
5. identify one executor and, when needed, one reviewer.

## Closure

After report review and operator acceptance:

1. set final status and verdict;
2. update reusable module/investigation/graph context;
3. move or reproduce the final task packet under `tasks/archive/PROMPTxxx/` without deleting numbered prompt/report history;
4. set `state/CURRENT_TASK.json` to `IDLE` or activate the next task.
