# Graphify finding — PROMPT305E critical fix gate

## FACT

- Committed: `PlatformOwnedIdentityEnsure.EnsureMissingAsync`
- Working tree only: `EntitiesService` call sites at apply + EventLog replay heal
- HEAD `EntitiesService` has no `ProcessTransactionGroupAsync`

## Implication

V014 clean-lane WH sync will fail without the WT API binary; cannot claim production-repeatable install until transaction-group + call sites are committed cleanly.
