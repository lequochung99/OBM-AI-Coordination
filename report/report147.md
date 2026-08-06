# report147 — PROMPT305D V013 Catalog Sync + Booking Create

## Verdict

`V013_CATALOG_SYNC_BOOKING_CREATE_PASS`

## Summary

V013 catalog seeded (17 categories / 152 services) from read-only same-tenant V012 copy, staged via production outbox helpers, drained Sent=2. BookingConsole create returned 200 with EventLog Seq/Exp=1 and assignment type 2. Tenant ensure-missing extracted to `PlatformOwnedIdentityEnsure` + 6 tests (committed `b139aba`); `EntitiesService` call sites remain mixed WT. V012 untouched.

## Counts

| Layer | Value |
|---|---|
| WPF outbox after drain | 276 Sent=2 (17 cat + 152 svc + 107 baseline) |
| API categories/services | 17 / 152 |
| EventLog cat/svc | 17 / 152 |
| SOM Asc/Desc | 13 / 4 (WPF=API) |
| BookingGuid | `f53cd35c-d7bc-4b2e-a94b-ba3bc7c1ad6b` |

## Source / coordination

- Source commit: `b139aba` (local-only)
- Local report SHA256: `5D24DB9B4538983CF2654DE94228EE36F2EC629B597631DD348D842C60E3D876`

## Next

PROMPT305E — isolate remaining EntitiesService transaction-group commit; day-schedule 400.
