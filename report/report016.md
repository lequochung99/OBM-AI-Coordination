# REPORT-0016 - Phase 2 V001 Manifest Key Resolution Audit

## Verdict

BLOCKED_PHASE2_V001_MANIFEST_KEYS_UNRESOLVED

## Prompt

- Prompt path: `prompt/prompt016.md`
- Prompt commit: `a7af8b842391d3e3955db408d89048f537ecb3b4`
- Mode: read-only manifest-resolution audit

## Scope Compliance

- No source files modified.
- No WPF files modified.
- No database schema, seed, migration, baseline, or runtime data modified.
- No build or test was run.
- No process was stopped or restarted.
- No local POS PostgreSQL database was created, connected for write, or mutated.
- No secrets, passwords, tokens, connection strings, or protected credentials were printed or copied.

## Prompt015 Blocker Recheck

Prompt015 left Phase 2 V001 blocked because the proposed manifest referenced six `TblParameterSetting` rows and three placeholder-safe `TblSetupPrinter` rows without proving the exact mandatory keys from canonical evidence.

This audit confirms the blocker remains unresolved. The available static evidence proves some read paths and legacy seed behavior, but it does not prove the exact six parameter keys or the exact three printer rows required for a safe implementation manifest.

## Evidence Priority Review

### Canonical accepted baseline and provisioning artifacts

Reviewed available canonical/recovery artifacts, including:

- `E:\Project2026\RecoveryReports\InstallationV0\Phase2SeedAuditV003`
- `report/report014.md`
- `report/report015.md`

The Prompt014 V003 evidence hashes matched the recorded manifest/report hashes. Its safe candidate output was useful as historical read-only evidence, but it showed reference data with `TblParameterSetting` count 110 and `TblSetupPrinter` count 5, not the proposed Phase 2 V001 manifest counts of 6 and 3.

### Read-only development DB check

The fixed local pgpass path existed, but read-only access to `obm_pos_dev_v0_pg` could not be established under the allowed credential path.

Safe failure observed:

```text
psql: error: connection to server at "127.0.0.1", port 5432 failed: fe_sendauth: no password supplied
```

The audit did not fall back to an admin user, did not read or print the password file, and did not change credentials.

The intended local evidence directory exists but no read-only proof files were produced:

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2ManifestKeysV001
```

### WPF source read paths and enums

Source files reviewed statically:

- `E:\Project2026\4POS\NailSalonNet8\Services\Parameters\BusinessDayRollService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\FeatureFlags\FeatureGateNames.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\FeatureFlags\FeatureGateService.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\QueueTurnRecalculationGate.cs`
- `E:\Project2026\4POS\NailSalonNet8\Enums\Types\PrintType.cs`
- `E:\Project2026\4POS\NailSalonNet8\Services\MainServices.cs`
- `E:\Project2026\4POS\NailSalonNet8\SeedDb\SeedDbProvider.cs`

## Parameter Key Findings

### Proven from current WPF source

`Date Hien Tai`

- Proven by `BusinessDayRollService.cs`.
- Legacy seed support exists in `SeedDbProvider.cs`.
- This is the strongest exact `TblParameterSetting.PropertyName` candidate.

`EnableTurnEngine`

- Present in `FeatureGateNames.cs`.
- Read through `FeatureGateService.cs` by exact `TblParameterSetting.PropertyName`.
- Used by `QueueTurnRecalculationGate.cs`.

### Feature gate names present in source

The current WPF source defines these feature-gate names:

```text
EnableGiftCardMovementEngine
EnableBookingWeightInTurn
EnableTurnEngine
EnableCustomerCheckInKiosk
EnableSignalRRefreshBell
EnableBackgroundReconciliationBot
EnablePayroll
```

However, the audit did not find static source proof that these are the same six mandatory seed rows intended by Prompt015.

### Reference-data keys not proven as current WPF read paths

Prompt014 reference evidence contained keys such as:

```text
CustomerCheckIn:Gateway:Prefix
CustomerCheckIn:Gateway:PublicUrl
CustomerCheckIn:Mode
CustomerCheckIn:TicketPrefix
FeatureFlags:EnableBookingIntegration
FeatureFlags:EnableCustomerCheckInModule
FeatureFlags:EnableCustomerSelfCheckInKiosk
FeatureFlags:EnablePromotionEngine
FeatureFlags:EnableTurnEngine
```

Current scoped WPF source searches did not prove these colon-prefixed names as active read paths. They therefore cannot safely be promoted into a canonical Phase 2 V001 manifest without further evidence or explicit operator approval.

## Parameter Decision

Exact six mandatory `TblParameterSetting` keys are not proven.

The audit does not recommend preserving the proposed count of six by compatibility assumption. It also does not recommend changing the count without operator approval. The next implementation prompt must either provide verified DB read evidence or explicitly approve the exact key list.

## Printer Findings

### Current enum values

`PrintType.cs` defines five values:

```text
CUSTOMER_TERMINAL
MERCHANT_TERMINAL
POS_CUSTOMER
POS_MERCHANT
POS_EMPLOYEE
```

### Runtime read paths

`MainServices.cs` reads all five printer types.

### Legacy seed support

`SeedDbProvider.cs` seeds all five printer types with machine/image-path-oriented data. This is legacy-supporting evidence only and is not proof of a three-row placeholder-safe canonical manifest.

## Printer Decision

Exact three placeholder-safe `TblSetupPrinter.PrinterType` rows are not proven.

Available source evidence points to five supported printer types, while Prompt015 requested three placeholder-safe rows. The audit found no canonical evidence proving which two of the five should be omitted.

## Proposed Stable Keys if Approved Later

These are design observations only, not an implementation decision:

- `TblParameterSetting`: `TenantGuid + PropertyName`, with an additional scope discriminator if duplicated values must be represented.
- `TblSetupPrinter`: `TenantGuid + PrinterType`.

## Runtime Impact Assessment

The unresolved manifest affects:

- new local database creation;
- Phase 2 seed selection;
- system setting availability;
- printer default availability;
- normal WPF startup behavior where source reads seeded defaults.

The audit did not find enough evidence to assert that omitting the unresolved rows is safe.

## Recommended Prompt017 Scope

Prompt017 should choose one of these small bounded paths:

1. Fix the read-only DB credential handoff for `obm_pos_dev_v0_pg` using the approved pgpass owner, then rerun only the safe SELECT audit.
2. Operator explicitly approves the exact six `TblParameterSetting.PropertyName` values and exact three `TblSetupPrinter.PrinterType` values.
3. Operator approves changing the manifest counts from six/three to the source-supported counts after a focused design review.

No Phase 2 implementation should proceed until one of those is completed.

## Final Classification

BLOCKED_PHASE2_V001_MANIFEST_KEYS_UNRESOLVED
