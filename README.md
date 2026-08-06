# OBM AI Coordination

This repository is the shared coordination layer for the OBM operator, ChatGPT, Cursor, Codex, and approved supporting tools such as Graphify.

It stores coordination artifacts, not the full OBM product source.

## Start here

Every AI agent must read in this order:

1. [`AGENTS.md`](AGENTS.md)
2. [`CURRENT.md`](CURRENT.md)
3. [`constitution/CURRENT.md`](constitution/CURRENT.md)
4. [`state/CURRENT_TASK.json`](state/CURRENT_TASK.json)
5. Only the active task and the context files referenced by that task

The complete structure is documented in [`docs/REPOSITORY_STRUCTURE.md`](docs/REPOSITORY_STRUCTURE.md).

## Operating workflow

1. The operator identifies the desired outcome.
2. ChatGPT classifies risk and prepares one complete numbered prompt/task.
3. Cursor is the primary current AI Engineer: targeted investigation, implementation, focused validation, report, and GitHub commit.
4. Codex later rejoins the same workflow for hard/critical implementation or independent review.
5. The operator performs UI and physical runtime acceptance.
6. Reusable discoveries update module context, investigation records, Graphify deltas, or decision records so later tasks do not repeat the same investigation.

The existing numbered history remains canonical:

- `prompt/promptXXX.md`
- `report/reportXXX.md`

## Graphify

Graphify may be refreshed frequently. Commit only reviewed public-safe manifests, summaries, and deltas. Because this repository is public, sensitive raw graph output must remain private and be referenced by version, path, artifact ID, commit SHA, or checksum.

See [`graphify/README.md`](graphify/README.md).

## Security boundary

Do not store passwords, API keys, tokens, private keys, real connection strings, customer data, employee personal data, production dumps, private logs, or full OBM source code in this public repository.

Binary and private evidence should be referenced by repository path, artifact ID, version, commit SHA, checksum, private storage location, or release URL rather than embedded directly.
