# OBM AI Shared Operating Rules

This file is the first mandatory read for ChatGPT, Cursor, Codex, and any temporary investigation agent.

## Canonical read order

1. `AGENTS.md`
2. `CURRENT.md`
3. `constitution/CURRENT.md`
4. `state/CURRENT_TASK.json`
5. The active task referenced by `state/CURRENT_TASK.json`
6. Only the module context, investigation, graph manifest, and reports referenced by that task

Do not read the entire repository unless the active task proves that broader inspection is necessary.

## Roles

- **ChatGPT:** task design, architecture, risk classification, report review, contradiction detection, and context curation.
- **Cursor:** primary code investigator and implementer; focused build/test; GitHub commit and report production.
- **Codex:** hard/critical implementation or independent review when quota is available.
- **Operator:** UI and physical runtime acceptance.
- **Other models:** optional, investigation-only, and only when they provide clear net value.

## One prompt equals one complete task

A task normally includes:

1. reuse existing context;
2. investigate only missing facts;
3. record every reusable discovery;
4. implement within scope;
5. run focused validation;
6. write the report and evidence references;
7. provide a manual UI/runtime checklist for the operator when needed.

Split a task only for a proven external blocker, an architecture decision requiring operator approval, or an unexpected critical-scope expansion.

## Investigation rule

Before investigating, search the active task references, module context, investigation index, and current Graphify manifest.

Do not repeat a documented investigation. If prior context is stale, record the exact invalidation evidence before reinvestigating.

Every new reusable discovery must update one of:

- `context/modules/<Module>/CURRENT.md`;
- `context/investigations/INVESTIGATIONxxx/`;
- `graphify/deltas/`;
- `decisions/`.

## Graphify rule

Graphify is a navigation aid, not the runtime source of truth.

- Update Graphify after a completed task or meaningful commit when source changed.
- Prefer incremental updates.
- Use the graph to locate the direct call chain.
- Read the actual affected source before changing code.
- Investigate only the conflicting or missing branch when graph and source disagree.
- Record graph source commit SHAs and changed modules.

## Validation policy

- The operator owns visual/UI acceptance.
- AI must provide a concise `MANUAL_UI_CHECK_REQUIRED` checklist when applicable.
- Use focused build/tests by default.
- Full cross-system verification is reserved for critical work such as payment, checkout, migration, installation, security, production data, and canonical sync/outbox behavior.

## Git and artifact policy

- Preserve existing numbered `prompt/` and `report/` artifacts.
- Never overwrite or delete prior versions automatically.
- Use versioned folders and explicit `CURRENT.md` pointers.
- Commit only task-related files.
- Prefer a branch and draft PR for coordination-structure changes.

## Public repository security boundary

This repository is currently public.

Never commit:

- passwords, API keys, JWTs, private keys, real connection strings, or secret environment files;
- customer or employee personal data;
- production database dumps or logs containing private data;
- full proprietary OBM source code;
- raw Graphify output when it embeds source text, secrets, private paths, or sensitive schema/data.

Public-safe Graphify summaries may contain sanitized module names, symbol references, call-chain summaries, source commit SHAs, checksums, and artifact references. Store sensitive raw graph artifacts outside this public repository and reference them by path, version, checksum, or private artifact URL.
