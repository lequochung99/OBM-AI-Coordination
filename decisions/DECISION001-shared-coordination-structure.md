# DECISION001 — Shared GitHub Coordination Structure

Status: `PROPOSED`

Date: 2026-08-06

## Context

OBM work is coordinated among the operator, ChatGPT, Cursor, and Codex. Repeated repository orientation, undocumented investigation, duplicated execution, and long prompt/report boilerplate consume quota and increase error risk.

Graphify can refresh code relationships cheaply, but its output must be tied to source commits and separated from AI interpretation. The coordination repository is public, so sensitive raw artifacts require a strict boundary.

## Decision

Use `lequochung99/OBM-AI-Coordination` as the shared coordination repository with these canonical layers:

1. `AGENTS.md` and Constitution — stable operating rules.
2. `state/` and `tasks/` — current machine/human task state.
3. Existing `prompt/` and `report/` — immutable numbered execution history.
4. `context/modules/` — durable module knowledge.
5. `context/investigations/` — reusable findings and corrections.
6. `graphify/` — public-safe manifests, summaries, deltas, and private artifact references.
7. `decisions/` — cross-task architecture/workflow decisions.
8. `templates/` — interoperable artifact formats.

The operator owns UI and physical acceptance. One primary executor owns implementation. Reviewers inspect risk-relevant diffs and direct call chains rather than duplicating the executor's entire investigation.

## Consequences

### Positive

- New agents have a deterministic read order.
- Graphify and investigation knowledge can be reused without scanning the whole codebase.
- Cursor and Codex can exchange task state through GitHub.
- Historical numbered prompts/reports remain intact.
- Public-repository security is explicit.

### Costs

- Active task pointers must be maintained accurately.
- Reusable findings require disciplined updates.
- Raw Graphify output may need private storage or sanitization.

## Acceptance

Change status to `ACCEPTED` only after operator review and merge of the proposed structure.
