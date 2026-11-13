# reportxxx — <Title>

## Verdict

`<EXACT_VERDICT>`

## Task and source state

| Field | Value |
|---|---|
| Prompt | `prompt/promptxxx.md` |
| Executor | <agent/model> |
| Source repositories | <names> |
| Starting commit(s) | <SHAs> |
| Ending commit(s) | <SHAs> |
| Branch | <branch> |
| Database/runtime lane | <safe identifier> |

## Reused context

- <module context>
- <investigation IDs>
- <Graphify manifest/delta>
- Repeated investigation: `NONE` or exact justified scope

## Root cause / finding

<Concise causal explanation. Separate facts from inference.>

## Changes

| File or component | Change | Reason |
|---|---|---|
| — | — | — |

## Protected invariants

- <invariant and evidence that it was preserved>

## Validation

| Check | Result | Evidence reference |
|---|---|---|
| Focused build | PASS/FAIL | <log/path/count> |
| Focused tests | PASS/FAIL | <count/path> |
| Runtime/API/DB | PASS/FAIL/NOT_REQUIRED | <safe summary> |

Do not paste large logs. Reference private evidence by path, artifact ID, SHA, or checksum.

## Manual operator verification

```text
MANUAL_UI_CHECK_REQUIRED | NOT_REQUIRED | COMPLETED_BY_OPERATOR
Screen/runtime:
Steps:
Expected result:
Control case:
Observed result:
```

AI must not claim visual or physical PASS before operator confirmation.

## Reusable knowledge produced

- Module context updated: <path or NONE>
- Investigation created/updated: <path or NONE>
- Graph delta created: <path or NONE>
- Decision record created: <path or NONE>

## Known limitations and follow-up

- <remaining issue or NONE>
