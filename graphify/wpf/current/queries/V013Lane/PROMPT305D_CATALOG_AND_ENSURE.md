# Graphify finding — PROMPT305D catalog + ensure-missing

## Catalog sync path (FACT)

WPF `TblServiceCategory`/`TblService` → `CreateLocalOutboxSingleAsync` + `CatalogOutboxPayloadHelper` → `FlushOutboxAsync` → API `ApplyServiceCategoryTransactionEventAsync` / `ApplyServiceTransactionEventAsync` → BookingConsole `GET services-with-category` / `POST create`.

## Ensure-missing (FACT)

`PlatformOwnedIdentityEnsure.EnsureMissingAsync` used from `EntitiesService` transaction-group apply + EventLog replay heal. Helper committed; EntitiesService call sites remain mixed WT.
