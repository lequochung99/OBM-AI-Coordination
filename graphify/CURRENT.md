# Current Graphify Snapshot

Status: `NOT_REGISTERED`

No Graphify scan is registered by this structural bootstrap.

When the first public-safe scan is prepared:

1. create `graphify/manifests/V001/GRAPH_MANIFEST.json`;
2. create only the necessary sanitized summaries under `graphify/summaries/`;
3. add private raw-artifact checksums/references when applicable;
4. replace this file with a pointer to the accepted manifest;
5. update `state/CURRENT_TASK.json` only when an active task depends on that graph.

Do not point this file to unreviewed generated output.
