# Reusable Technical Context

This directory stores curated knowledge that should reduce repeated repository investigation.

## Structure

```text
context/
  modules/            Stable per-module maps and contracts
  investigations/     Versioned read-only findings and corrections
  project/            Cross-repository indexes and architecture summaries
```

## Content quality

Every factual statement should identify its evidence through source repository, commit SHA, file path, symbol, test, database proof, or numbered report.

Use these labels explicitly:

- `FACT` — verified directly;
- `INFERENCE` — reasoned but not directly proven;
- `UNKNOWN` — requires targeted inspection or runtime evidence;
- `CONFLICT` — sources disagree;
- `SUPERSEDED` — replaced by a newer version without deleting history.

Do not copy large source files into this repository. Reference source locations and commits instead.
