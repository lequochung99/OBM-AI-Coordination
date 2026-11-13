# V014 Lane — DB_CONTEXT V002

Status: `PREP_NOT_STARTED_BLOCKER_CLEARED`
Task: `PROMPT305E-FIX — Isolate And Commit EntitiesService Transaction-Group Path`
Investigator: Cursor

## Lane targets (still not created)

| Role | Database |
|---|---|
| WPF | `obm_pos_dev_v014_pg` |
| API | `obm_api_dev_v014_pg` |

## Blocker status

`BLOCKED_V014_CRITICAL_FIX_UNCOMMITTED` (V001) — **CLEARED**.

Transaction-group process/apply path + Tenant/PosLocal ensure-missing call sites committed at
`54b408b` on `recovery/wpf-installation-reset-cursor-v001`
(`EntitiesService.TransactionGroup.cs`, `EntitiesService.cs` partial+dbFactory,
`IEntitiesService.cs`).

## DBs created this prompt

None — this prompt was source-isolation/commit only (PROMPT305E-FIX §1 out-of-scope: no V014
DB creation, no API start, no WPF config, no pairing).

## Next action

Retry `PROMPT305E — Prepare V014 Manual WPF Installation Observation Lane`: clean
`obm_api_dev_v014_pg` / `obm_pos_dev_v014_pg`, start API against V014, pairing, WPF config,
watch scripts, pause for operator.

## Pointers

- Local report: Prompt305E_FixTransactionGroupIsolation V001
- Coordination: `report/report149.md`
- Prior: `context/database/V014Lane/V001/DB_CONTEXT.md`
