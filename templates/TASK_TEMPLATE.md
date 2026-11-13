# PROMPTxxx Task Packet — <Title>

## Classification

| Field | Value |
|---|---|
| Task type | INVESTIGATION_ONLY / LOCAL_IMPLEMENTATION / CROSS_COMPONENT / CRITICAL |
| Risk | L0 / L1 / L2 / L3 |
| Primary executor | Cursor / Codex / approved alternative |
| Reviewer | None / ChatGPT / Cursor / Codex |
| UI owner | Operator |
| Source worktree/branch | `<path and branch>` |

## Goal

<One clear business or engineering outcome.>

## Business acceptance

1. <Observable required behavior>
2. <Negative or regression condition>
3. <Operator acceptance condition>

## Reusable context — mandatory first reads

- Constitution: `constitution/CURRENT.md`
- Module context: `<path or NONE>`
- Investigations: `<paths or NONE>`
- Graphify manifest: `<path or graphify/CURRENT.md>`
- Prior reports: `<paths or NONE>`

Do not repeat an investigation documented above unless exact invalidation evidence is recorded.

## Known facts

- `FACT:` <verified fact with source reference>

## Unknowns permitted for targeted investigation

- `UNKNOWN:` <specific unanswered question>

## In scope

- <files/modules/behaviors allowed>

## Protected / out of scope

- <areas that must not change>

## Investigation budget

- Start from referenced graph/context.
- Inspect only the direct call chain and discovered concrete dependencies.
- Record every reusable finding in the designated context artifact.
- Stop after three meaningful failed attempts, or earlier when the same blocker repeats.

## Implementation requirements

1. <requirement>
2. <requirement>

## Validation

### Focused automated checks

- <build/test/query/probe>

### Critical checks when applicable

- <transaction, migration, sync, security, production evidence>

## Manual UI/runtime check

```text
MANUAL_UI_CHECK_REQUIRED | NOT_REQUIRED
Screen/runtime:
Steps:
Expected result:
Control case:
```

## Required outputs

- Numbered prompt: `prompt/promptxxx.md`
- Task status: `tasks/active/PROMPTxxx/STATUS.json`
- Report: `report/reportxxx.md`
- Reusable context updates: `<paths or NONE with justification>`
- Graph delta: `<path or NONE with justification>`

## Allowed verdicts

- `<EXACT_PASS_VERDICT>`
- `BLOCKED_<EXACT_REASON>`
- `OPERATOR_UI_CHECK_PENDING`
