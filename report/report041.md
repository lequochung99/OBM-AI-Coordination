# REPORT-041 - Preserve InstallationV0 ProductRoot Handoff

## Verdict

OBM_POS_PRODUCTROOT_HANDOFF_READY_FOR_USER_RETEST

## Prompt

- Source prompt: `prompt/prompt041.md`
- Coordination repository: `lequochung99/OBM-AI-Coordination`
- Execution date: 2026-08-01

## Scope

Prompt041 investigated and corrected the InstallationV0 diagnostics `Open OBM-POS` handoff so the exact verified V0 ProductRoot is preserved when transitioning from the diagnostics window into normal OBM-POS runtime startup.

Canonical ProductRoot:

`E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot`

Canonical database:

`obm_pos_dev_v0_pg`

## Root Cause

The physical diagnostics page showed the correct V0 ProductRoot, but the same-process main runtime handoff did not pass that verified root into the main startup readiness path. The Development guard still had stale approved Development constants, so the normal startup readiness check could classify the database/root lane as rejected even after Phase 2 v002 completion was visible in the diagnostics UI.

## Implemented Correction

- `InstallationV0Module` now exposes an explicit ProductRoot-aware Open POS callback.
- `InstallationV0Window` passes the service-resolved ProductRoot into the Open OBM-POS request.
- `App.xaml.cs` validates the handoff ProductRoot before main startup readiness retry.
- The verified ProductRoot is applied through a process-local in-memory override only.
- Missing ProductRoot, forbidden roots, ProgramData fallback, and handoff/profile mismatch are rejected.
- Development-approved constants now point to the V0 isolated lane.
- Prompt build label now reports `prompt041`.
- The stale Phase 1 message was replaced with:

`Phase 1 resume verified. Pairing Code is not required. Phase 2 database status is shown above.`

## Files Changed In Source

Source changes were made under the WPF source tree only and were not committed or pushed by this task:

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Infrastructure\Phase1InstallationService.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0Module.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\DevelopmentProfileLaunchPolicy.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

## Build And Test Evidence

- `dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj`: PASS
- `dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj`: PASS, 0 errors
- `dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap"`: PASS
- Focused test count: 94 passed, 0 failed, 0 skipped

Existing project warnings remain outside the prompt041 functional change.

## Read-Only Database Verification

Executed a PostgreSQL read-only transaction against `obm_pos_dev_v0_pg` and rolled it back.

Observed canonical counts:

- `TblEmployeePermission`: 7
- `TblEmployee`: 20
- `TblLocalOutbox`: 62
- `Phase2TrialCompletionMarker` v002 completed marker: 1
- Runtime state: `Activated`

No Phase 2 rerun, database creation, migration, seed, provisioning, or runtime mutation was performed.

## ProductRoot Contract

Canonical precedence after this correction:

1. Explicit verified ProductRoot handed from InstallationV0 diagnostics UI to normal startup.
2. Standard Development launch profile `SPACEPOS_PRODUCT_ROOT`.
3. No Development fallback to ProgramData or stale source folders.

ProgramData and stale Development roots remain rejected for Development runtime startup.

## Retest Instructions

Route A, Visual Studio Runtime Development profile:

- Start the standard OBM-POS Runtime Development profile.
- Verify it opens MainWindow directly.
- Verify no `DevelopmentDatabaseRejected` guard is shown.
- Verify no setup wizard opens.

Route B, InstallationV0 diagnostics:

- Start the InstallationV0 diagnostics profile with the canonical V0 ProductRoot.
- Verify diagnostics shows Phase 2 v002 complete.
- Click `Open OBM-POS`.
- Expected result: exactly one MainWindow opens and the diagnostics window closes.
- Expected blocked result if any: specific ProductRoot mismatch reason, not generic fallback behavior.

## Safety Confirmation

- No secrets, passwords, tokens, or connection strings were printed.
- No WPF physical launch was performed in this report task.
- No local POS database mutation was performed.
- No Platform mutation was performed.
- No source commit or source push was performed.
- Only this coordination report is intended to be committed and pushed.

## Final Classification

OBM_POS_PRODUCTROOT_HANDOFF_READY_FOR_USER_RETEST
