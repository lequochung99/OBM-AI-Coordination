# report149 — PROMPT305E-FIX Isolate And Commit EntitiesService Transaction-Group Path

## Verdict

`TRANSACTION_GROUP_SOURCE_ISOLATION_COMMIT_PASS`

## Summary

Extracted `ProcessTransactionGroupAsync` (+ validation, domain apply dispatch, replay
handling, SignalR notify, response builders) from the dirty `EntitiesService.cs` into a new
partial-class file `EntitiesService.TransactionGroup.cs`. The block was fully self-contained
in the working tree (no cross-file dependencies beyond shared instance fields), so no
behavior was lost. `EntitiesService.cs` itself only needed a minimal edit (`partial` keyword
+ `_dbFactory` field/ctor param); all other unrelated dirty hunks in that file (gift-card
refactor, `ServiceOrderMode` normalize, `WebServiceDescriptionOverwriteProtection`,
`WebBookingAssignmentType`, booking EventLog enrichment) were left uncommitted/out of scope.

Tenant/PosLocal ensure-missing wiring (via the already-committed `PlatformOwnedIdentityEnsure`
helper from `b139aba`) is now committed as part of `ApplyTransactionGroupDomainAsync` and the
replay-heal path. Added CostFee/WorkAbsence transaction-group apply test coverage (previously
missing) and repointed two source-text architecture tests to the new file location. All 10
required tests from §4 pass. Build: 0 errors. Focused filter: 33 passed / 2 pre-existing
unrelated failures (WPF source assertion; missing migration-DB env var). Full test project:
166 passed / 8 pre-existing unrelated failures (lane-name/env/stale-contract, confirmed
unchanged by the extraction).

An incident occurred and was fully recovered during this work: a first extraction attempt
normalized line endings across the whole file (via `Get-Content`/`Set-Content`), producing an
oversized diff; while correcting it, `git checkout --` briefly discarded the dirty
`EntitiesService.cs` working tree. It was restored byte-for-byte via a pre-captured diff and
`git apply --ignore-whitespace`, then re-extracted safely using raw string slicing. No data
was permanently lost; full detail in the local report.

## Unblock cleared

`BLOCKED_V014_CRITICAL_FIX_UNCOMMITTED` (report148) is cleared. Transaction-group process path
and Tenant/PosLocal ensure-missing call sites are committed at `54b408b` on
`recovery/wpf-installation-reset-cursor-v001`.

## Local artifact

Path: `E:\Project2026\RecoveryReports\V014\Prompt305E_FixTransactionGroupIsolation\V001\PROMPT305E_FIX_TRANSACTION_GROUP_ISOLATION_REPORT.md`
SHA256: `82C15F7602E449488FC5F140841A64EF2A91C0D5604A96E5132DC27B611DA02A`

## Source

Branch `recovery/wpf-installation-reset-cursor-v001`, commit `54b408b`
("Isolate API transaction-group sync apply path"). Local-only / no origin — not pushed.

## Next

Retry `PROMPT305E — Prepare V014 Manual WPF Installation Observation Lane` (clean V014 DB/API
prep, pairing, WPF config, operator pause).
