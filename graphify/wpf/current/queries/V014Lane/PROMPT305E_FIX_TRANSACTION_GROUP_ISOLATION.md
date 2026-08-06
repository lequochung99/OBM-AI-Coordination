# Graphify finding — PROMPT305E-FIX transaction-group isolation

## FACT (post-fix)

- `EntitiesService.TransactionGroup.cs` (partial class `EntitiesService`) now committed at
  `54b408b`; contains `ProcessTransactionGroupAsync` and its full private/static helper
  surface (validation, envelope checks, domain apply dispatch, replay handling, SignalR
  notify, response builders).
- `ApplyTransactionGroupDomainAsync` dispatches `TblTenant`/`TblPosLocal` through
  `PlatformOwnedIdentityEnsure.EnsureMissingAsync` (committed at `b139aba`); replay path in
  `ProcessTransactionGroupAsync` re-runs ensure-missing on `TblTenant`/`TblPosLocal` replay
  events to heal older clean-lane EventLog-without-domain-row gaps.
- `EntitiesService.cs` now `partial`, carries `_dbFactory` field/ctor param used by the TG
  file; ~285/-66 lines of unrelated dirty hunks (gift card, ServiceOrderMode, booking
  description-overwrite protection, booking assignment type, booking EventLog enrichment)
  remain intentionally uncommitted in that file — out of scope for V014 transaction-group
  behavior.

## Implication

V014 clean-lane transaction-group apply (Tenant, PosLocal, WH, CostFee, ServiceCategory,
Service, WorkAbsence, setup entities) is now backed by committed source. PROMPT305E V014
lane prep may proceed.
