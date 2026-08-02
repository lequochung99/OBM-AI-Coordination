# report138 - InstallationV0 post-redeem command enablement

## Verdict

`WPF_V005_POST_REDEEM_INSTALL_RESUME_ENABLEMENT_FIXED`

Full physical verdict is not claimed. The remaining V005 physical path must still be run by the operator through ApplicationReady, MainWindow 60-second proof, and restart proofs.

## Root Cause Classification

`UI_PREDICATE_REQUIRED_PHASE2_HYDRATION_BEFORE_DB_INSTALL`

After successful Pairing Code redeem and protected checkpoint persistence, the WPF window attempted Phase 2 hydration against the local target database. When the target database was absent, empty, or not yet baselined, the hydration blocked path hard-disabled `Install/Resume Local Database`. That made the UI require local runtime readiness before allowing the action that creates/resumes that readiness.

## Active Owner Reused

The fix reuses the existing Phase 1 resume/readiness owner after successful redeem:

- redeem succeeds;
- protected checkpoint is persisted by the existing Phase 1 service;
- the window immediately calls the existing resume assessment;
- command state is recomputed from the refreshed checkpoint, current local DB inputs, hydration status, and operation-busy flags.

## Files Changed

- `4POS/NailSalonNet8/InstallationV0/Application/InstallationV0CommandRules.cs`
- `4POS/NailSalonNet8/InstallationV0/Application/InstallationV0BuildInfo.cs`
- `4POS/NailSalonNet8/InstallationV0/Presentation/InstallationV0Window.cs`
- `4POS/NailSalonNet8.Tests/InstallationV0/InstallationV0Phase1Tests.cs`

## Predicate Before/After

Before:

- `Install/Resume Local Database` could be disabled by Phase 2 hydration failure even when the protected checkpoint existed and local DB inputs were valid.
- `Open OBM-POS` required Phase 2 complete state.

After:

- `Install/Resume Local Database` is enabled when the protected checkpoint exists, local DB host/port/credentials are present in the current UI attempt, the database name passes safety validation, the target is not ApplicationReady/Activated, the target is safely resumable, and no install/open operation is running.
- Pairing Code textbox contents are not part of the post-redeem install predicate.
- Hydration failure for a not-yet-baselined DB no longer disables install when the checkpoint and DB inputs are valid.
- `Open OBM-POS` remains disabled until ApplicationReady/Activated.

## Build And Tests

- `dotnet build` for `InstallationV0`: passed, 0 warnings, 0 errors.
- Focused `InstallationV0` tests: passed, 69 passed, 0 failed, 0 skipped.

## Destructive Operation Counts

Changed source files contain:

- `DROP DATABASE`: 0
- `DROP SCHEMA`: 0
- `TRUNCATE`: 0
- `EnsureDeleted`: 0

No database reset, recreation, manual table creation, manual ApplicationReady insertion, or physical DB mutation was performed by Codex for this prompt.

## Physical Result

Not performed in this Codex runner. Operator should physically verify:

1. Launch WPF build with prompt138 label.
2. Redeem a valid Pairing Code or load an existing protected checkpoint.
3. Confirm `Install/Resume Local Database` enables immediately after successful reassessment.
4. Confirm `Open OBM-POS` remains disabled before ApplicationReady/Activated.
5. Click `Install/Resume Local Database` once and resume the existing empty local DB without drop/recreate.
6. Continue to ApplicationReady, MainWindow, and restart proofs before claiming full physical pass.

## Local Evidence

Local evidence artifact version: `WpfV005PostRedeemEnablementPrompt138V001`

Manifest SHA-256: `07B03134E1F099014F272D2884550285081B3A215222BCBAE7557D32E43EC7A7`

## Security

No Pairing Code, JWT/token, Google identity, raw tenant/POS/attempt/local-installation identity values, PostgreSQL passwords, or full connection strings are included in this report.
