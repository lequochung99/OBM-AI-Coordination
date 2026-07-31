# OBM AI Coordination

This repository stores manual AI coordination artifacts for OBM projects.

The workflow is intentionally human-controlled:

1. The user identifies the current task.
2. ChatGPT prepares a numbered, versioned prompt.
3. The user tells Codex when the next prompt is ready.
4. Codex executes the task and writes a numbered, versioned report.
5. The user tells ChatGPT when the numbered Codex report is ready.
6. ChatGPT reviews the report and evidence, then prepares the next numbered prompt.
7. GitHub stores versioned prompts, reports, handoffs, manifests, and diagrams.

Binary files should not be embedded directly in coordination artifacts. Reference them by path, artifact ID, version, commit SHA, checksum, or release URL.

Do not store passwords, API keys, tokens, private keys, real connection strings, customer data, or full OBM source code in this public repository.

