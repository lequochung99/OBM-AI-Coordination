# Graphify Coordination Area

Graphify provides a frequently refreshed navigation map for ChatGPT, Cursor, and Codex.

## Structure

```text
graphify/
  CURRENT.md                    Pointer to the active public-safe manifest
  manifests/                    Versioned scan manifests
  summaries/                    Sanitized architecture and module summaries
  deltas/                       Meaningful graph changes between source commits
  private-artifact-references/  Checksums and locations of sensitive raw output
```

## Refresh policy

Graphify may run after each completed task, meaningful commit, or several hours of active code changes. Prefer incremental refresh when available.

A graph refresh does not automatically require a full AI resummary. Update only the affected module summary or delta unless architecture boundaries changed materially.

## Public repository safety

Before committing any generated file, verify that it contains no:

- source bodies or large code excerpts;
- credentials, tokens, connection strings, or environment values;
- customer/employee data;
- database row contents;
- private hostnames, sensitive absolute paths, or production topology;
- proprietary implementation detail inappropriate for a public repository.

Raw `graph.json`, HTML viewers, caches, embeddings, or generated databases must remain private when they contain such material. Commit only a manifest plus a checksum/path reference in `private-artifact-references/`.

## Consumption rule

1. Read `graphify/CURRENT.md`.
2. Confirm source commit SHAs.
3. Use summaries to find the direct call chain.
4. Open the actual affected source files.
5. Perform targeted investigation only when source and graph disagree.
6. Record the conflict and correction in a delta or investigation artifact.
