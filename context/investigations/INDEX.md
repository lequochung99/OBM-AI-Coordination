# Investigation Index

Register every reusable investigation here. Do not rely on chat history as the only record.

| ID | Module | Status | Source commits | Summary | Superseded by |
|---|---|---|---|---|---|
| DBCTX-TblSetupCostAndFeeMerchant-V001 | TblSetupCostAndFeeMerchant | VERIFIED | WPF `d7bd0177` | Fee spelling canonical; seeded ClientFeeInPercent=2 via BaselineSeeder + InitialSeedBootstrapOutbox; Free is mapper-name-only | — |

## Folder convention

```text
context/investigations/INVESTIGATION001/
  FINDINGS.md
  EVIDENCE.md
  AFFECTED_SYMBOLS.md
  CORRECTIONS.md
```

## Required fields

- investigation ID, date, task ID, investigator;
- exact question investigated;
- source repositories and commit SHAs;
- files and symbols inspected;
- facts, inferences, unknowns, and conflicts;
- call chain and data flow;
- conclusion and implementation implications;
- validity and invalidation conditions;
- follow-up task references;
- correction or superseding investigation when later evidence disagrees.
