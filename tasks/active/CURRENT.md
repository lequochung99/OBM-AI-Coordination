# Active Task

Status: `NONE`

The structural bootstrap does not activate a numbered task.

When a task is ready, replace this content with:

```text
Active task: PROMPTxxx
Task packet: tasks/active/PROMPTxxx/TASK.md
Prompt: prompt/promptxxx.md
Status: READY | IN_PROGRESS | IMPLEMENTED | OPERATOR_CHECK | BLOCKED
Executor: Cursor | Codex | other approved executor
Reviewer: none | ChatGPT | Cursor | Codex
```

Update `state/CURRENT_TASK.json` in the same coordination change.
