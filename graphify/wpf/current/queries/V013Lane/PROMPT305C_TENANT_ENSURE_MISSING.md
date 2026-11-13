# Graphify finding — PROMPT305C Tenant ensure-missing (API)

Date: 2026-08-06  
Confidence: FACT (runtime + source read)

## Confirmed edge

`SyncTransactionGroup` → `EntitiesService.ApplyTransactionGroupDomainAsync`  
→ previously **skipped** `TblTenant` / `TblPosLocal` domain apply (EventLog only)  
→ `ApplyWebEmployeeWorkingHourTransactionEventAsync` insert failed `FK_TblWebEmployeeWorkingHour_Tenant` on clean V013.

## Fix (working tree)

`EnsureMissingPlatformOwnedIdentityAsync` on first apply + on EventLog idempotent replay heal.

Do not treat Graphify absence of this edge as deletion proof; DI/runtime confirmed.
