# Report 015 — Phase 2 Atomic Baseline Seed v001 Implementation Gate

## 1. Verdict

```text
BLOCKED_PHASE2_V001_MANIFEST_KEYS_UNRESOLVED
```

Prompt015 was stopped before source/database implementation because the required manifest gate could not be satisfied without guessing:

- exact six safe mandatory `TblParameterSetting` keys could not be proven;
- exact three placeholder-safe `TblSetupPrinter.PrinterType` rows could not be proven.

No Phase 2 source implementation was started.

## 2. Phase 1 Freeze / Prerequisite Proof

Prompt015 did not modify or read raw Phase 1 secrets.

Protected Phase 1 artifacts remain governed by the prior accepted Phase 1 closure:

```text
PHASE1_WPF_API_AUTHORIZATION_AND_MACHINE_PERSISTENCE_PASS_DATABASE_NOT_STARTED
```

No Pairing Code was requested or redeemed. No WpfJwt, DPAPI credential, checkpoint content, or LocalInstallationGuid was printed, copied, rotated, deleted, or overwritten.

## 3. Operator Decisions Applied

The following locked decisions were accepted as prompt015 inputs:

- marker table must be `dbo.TblInstallationV0Phase2SeedVersion`;
- manifest version must be `phase2-baseline-seed-v001`;
- default roles must be exactly `Owner`, `Admin`, `Sub_Manager`;
- `Sub_Manager` is the physical representation of operator-facing `SubAdmin`;
- `TblTurnSetting` is deferred;
- `TblSetting` is deferred;
- printer defaults are limited to 3 placeholder-safe rows;
- dedicated InstallationV0 deterministic outbox writer is required.

Implementation did not begin because the manifest key/type gate failed first.

## 4. Exact Six Parameter Keys / Three Printer Types

### Parameter Key Evidence

Evidence sources read:

- `E:\Project2026\RecoveryReports\InstallationV0\Phase2SeedAuditV003\safe-candidate-patterns.tsv`
- `E:\Project2026\4POS\NailSalonNet8\Services\FeatureFlags\FeatureGateNames.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\FeatureFlags\FeatureGateService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\Parameters\BusinessDayRollService.cs`
- `E:\Project2026\4POS\NailSalonNet8\SeedDb\SeedDbProvider.cs`

Proven safe direct key:

```text
Date Hien Tai
```

Proven supported feature flag names in source:

```text
EnableGiftCardMovementEngine
EnableBookingWeightInTurn
EnableTurnEngine
EnableCustomerCheckInKiosk
EnableSignalRRefreshBell
EnableBackgroundReconciliationBot
EnablePayroll
```

But live reference evidence records many keys with different prefixed names, such as feature/check-in forms, and duplicated rows by key/scope. The source scan only proved a direct read path for `Date Hien Tai` and `EnableTurnEngine`; it did not prove exactly six mandatory startup-safe keys.

Therefore the six-key list is unresolved.

### Printer Type Evidence

Evidence sources read:

- `E:\Project2026\4POS\NailSalonNet8\Enums\Types\PrintType.cs`
- `E:\Project2026\4POS\NailSalonNet8\SeedDb\SeedDbProvider.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\MainServices.cs`

Source proves five active printer types:

```text
CUSTOMER_TERMINAL
MERCHANT_TERMINAL
POS_CUSTOMER
POS_MERCHANT
POS_EMPLOYEE
```

Prompt015 requires exactly three placeholder-safe printer rows. No source or approved evidence proves which three of the five should be selected. Choosing any three would be a guess.

Therefore the three printer types are unresolved.

## 5. Manifest Files and Hash / Version

No source manifest was created.

The approved manifest version remains:

```text
phase2-baseline-seed-v001
```

Prompt015 requires exact keys/types before manifest implementation. That prerequisite failed.

## 6. Marker Schema / Migration Details

No marker schema or migration was created.

Approved but unimplemented marker target:

```text
dbo.TblInstallationV0Phase2SeedVersion
```

## 7. Exact Files Changed

Source files changed under `E:\Project2026`:

```text
none
```

Coordination repository file created:

```text
report/report015.md
```

## 8. Exact Row Groups / Counts / Stable Keys

Not implemented.

The approved target groups remain blocked at the manifest gate:

- 1 `TblTenant`;
- 1 `TblPosLocal`;
- 1 `TblSetupWeird`;
- 1 `TblSetupServicesMethod`;
- 3 `TblSetupLoginMethod`;
- 6 `TblSetupPaymentMethod`;
- 3 `TblEmployeePermission`;
- 6 `TblParameterSetting`;
- 3 `TblSetupPrinter`;
- 1 marker;
- matching outbox rows for actual inserted/updated non-marker baseline rows.

The `TblParameterSetting` and `TblSetupPrinter` exact stable keys/types were not proven.

## 9. Transaction Ownership Proof

Not implemented because source implementation did not begin.

Required future proof remains:

```text
one Npgsql/DbContext transaction owns seed rows + TblLocalOutbox rows + marker
```

## 10. Advisory-Lock Implementation

Not implemented.

Required future strategy remains transaction-scoped advisory lock derived from:

```text
InstallationV0 + Phase2 + TenantGuid + PosGuid + phase2-baseline-seed-v001
```

## 11. Outbox Writer / Deterministic Payload Design

Not implemented.

The dedicated InstallationV0 Phase 2 outbox writer remains required in the next implementation prompt after manifest keys/types are resolved.

## 12. Marker-Last / Readback-Before-Commit Proof

Not implemented.

No seed transaction was written or executed.

## 13. Rollback / Failure-Injection Proof

Not implemented.

No failure-injection tests were added or run.

## 14. Same-Version Replay Proof

Not implemented.

No replay/idempotency code or tests were added.

## 15. Partial / Conflicting / Newer-State Behavior

Not implemented.

Required future result codes remain:

```text
PHASE2_PARTIAL_BASELINE_RECOVERY_REQUIRED
PHASE2_BASELINE_CONFLICT
PHASE2_NEWER_MANIFEST_PRESENT
```

## 16. Explicit Excluded-Table Proof

No excluded table was seeded or modified.

Prompt015 did not touch:

- `TblEmployee`;
- `TblServiceCategory`;
- `TblService`;
- `TblProduct`;
- `TblCustomer*`;
- `TblGiftCard*`;
- `TblInvoice*`;
- `TblOutputInfo*`;
- booking/appointment tables;
- terminal payment/runtime data;
- queue/turn/payroll runtime history;
- historical `TblLocalOutbox` rows.

## 17. Build / Test Commands

No build or tests were run.

Reason: prompt015 instructed to stop before DB/source implementation if exact six parameter keys or three printer types could not be proven.

## 18. Disposable Physical E2E

Not run.

Reason: source implementation was blocked before any disposable E2E could be valid.

Reference database `enailsalon_phasee1_pos1_pg` was not mutated.

## 19. Runtime / Process Handoff

No WPF, ApiServer, or PlatformAppV0 process was stopped, restarted, or launched.

No WPF process was left running by this task.

## 20. Prompt015 Label Proof

Not updated.

Reason: no WPF source changes were allowed after the manifest gate failed. The active WPF label remains whatever was present before prompt015; this is intentional because prompt015 implementation did not begin.

## 21. No Reference DB Mutation / No Secret Leakage

Confirmed:

- no SQL mutation was run;
- no Phase 2 target database was created or modified;
- no reference DB row/schema/sequence/privilege was changed;
- no password, connection string, pgpass content, WpfJwt, Pairing Code, token, or protected credential was printed or committed;
- no raw customer/employee/gift-card/invoice/payment/business row was copied into this public report.

## 22. Source Git / No-Push Confirmation

No `E:\Project2026` source commit or push was performed.

No `git add .`, `git add -A`, reset, clean, stash, checkout, or restore was run against source.

## 23. Exact User Physical Test Steps

No physical WPF Phase 2 test is ready yet.

Required next step before implementation:

```text
Approve exact six TblParameterSetting keys and exact three TblSetupPrinter printer types
or provide a new prompt that changes the v001 counts.
```

After that, a future prompt can implement and test the one-transaction Phase 2 seed.

## 24. Coordination Commit SHA

Pending at report creation time. Final response will include the commit SHA after commit and push.
