# Report 043 - Verified Runtime Handoff Provenance

## Verdict

OBM_POS_VERIFIED_RUNTIME_HANDOFF_PROVENANCE_READY_FOR_USER_RETEST

## Scope

Prompt: `prompt/prompt043.md`

Source roots inspected and changed:

- `E:\Project2026\4POS\NailSalonNet8`
- `E:\Project2026\4POS\NailSalonNet8.Tests`

Coordination artifact created:

- `report/report043.md`

No source commit was made in the OBM source tree. Only this coordination report is committed to `lequochung99/OBM-AI-Coordination`.

## Prompt042 Failure Classification

The operator retest from prompt042 proved:

- `ResultCode=DevelopmentDatabaseRejected`
- `StageId=DevelopmentStartupGuard`
- `EffectiveProductRootSource=LaunchProfileEnvironment`
- `RootPresent=True`
- `RootApproved=True`
- `CachedAssessmentReused=False`

The ProductRoot handoff was fixed. The remaining rejected predicate was the development database predicate, not the ProductRoot predicate.

## Root Cause

The post-configuration development startup guard was reading the database name from the current/default connection string path instead of preferring the runtime bootstrap metadata from the approved ProductRoot.

That made the guard reject startup with `DevelopmentDatabaseRejected` even though:

- the V0 ProductRoot was present;
- the V0 ProductRoot was approved;
- runtime bootstrap under that ProductRoot identified the approved V0 database.

The prompt043 fix makes the guard prefer the ProductRoot runtime bootstrap database when the effective ProductRoot is approved, then fall back to parsing the connection string only when bootstrap metadata is unavailable.

## Changed Source Files

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\DevelopmentProfileLaunchPolicy.cs`
- `E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\LaunchProvenanceContext.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\R4DevelopmentProfileLauncherTests.cs`

## Implementation Summary

Added explicit launch provenance modes:

- `InstallationDiagnostics`
- `RuntimeDevelopmentProfile`
- `VerifiedInstallationHandoff`
- `MissingOrRejected`

Added a process-local verified handoff state through `LaunchProvenanceContext`.

Implemented same-process handoff behavior after successful installation readiness proof:

- mark provenance as `VerifiedInstallationHandoff`;
- set effective ProductRoot source to `InstallationHandoff`;
- preserve process `SPACEPOS_PRODUCT_ROOT`;
- clear process `SPACEPOS_INSTALLATION_MODULE`;
- clear the latched startup assessment;
- perform a fresh guard/readiness assessment.

Kept diagnostics mode fail-closed for runtime handoff:

- `Open OBM-POS` is blocked unless the phase2/v002 completed proof is present;
- missing completed proof records `INSTALLATION_V0_COMPLETED_PROOF_MISSING`;
- diagnostics mode is not silently promoted to runtime mode.

Kept direct Visual Studio runtime development behavior:

- a normal runtime development profile can launch only with the approved V0 ProductRoot and approved V0 database;
- ProgramData fallback remains disabled in development/debug paths;
- forbidden legacy/Royal/Production paths remain rejected.

## Evidence References

Key source evidence:

- `LaunchProvenanceContext`: `E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\LaunchProvenanceContext.cs`
- Guard DB predicate: `E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\DevelopmentProfileLaunchPolicy.cs:150`
- Guard provenance predicate: `E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\DevelopmentProfileLaunchPolicy.cs:153`
- Guard rejected provenance: `E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\DevelopmentProfileLaunchPolicy.cs:160`
- Runtime bootstrap DB resolver: `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs:747`
- Verified handoff diagnostic: `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs:1268`
- Handoff provenance mark: `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs:1302`
- Fresh readiness guard rerun: `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs:1347`
- Missing completed proof guard: `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs:187`
- Build label: `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs:5`

## Fail-Closed Result Codes

Implemented or preserved safe result codes:

- `DEVELOPMENT_PRODUCT_ROOT_MISSING`
- `DEVELOPMENT_PRODUCT_ROOT_NOT_APPROVED`
- `DEVELOPMENT_PRODUCT_ROOT_HANDOFF_PROFILE_MISMATCH`
- `DEVELOPMENT_DATABASE_NOT_APPROVED`
- `DEVELOPMENT_LAUNCH_PROVENANCE_NOT_APPROVED`
- `INSTALLATION_V0_COMPLETED_PROOF_MISSING`
- `VERIFIED_INSTALLATION_HANDOFF_APPROVED`

## Build Verification

Command:

```powershell
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj
```

Result:

- PASS
- 0 warnings
- 0 errors

Command:

```powershell
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj
```

Result:

- PASS
- 0 warnings
- 0 errors

Built binary evidence:

- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\bin\net8.0-windows\InstallationV0.dll`
  - LastWriteTimeUtc: `2026-08-01T13:40:17Z`
  - SHA-256: `DF4DD72C134C9459AC2F88D107D1D6C35F0D7BD9DCEAE8082127218D1EE342E4`
- `E:\Project2026\4POS\NailSalonNet8\bin\net8.0-windows\NailSalonNet8.dll`
  - LastWriteTimeUtc: `2026-08-01T13:40:36Z`
  - SHA-256: `BE6A6E60B42A2CDC85F153B0AAB9E00090FA42111BAF721C404E23193A09EB89`

## Test Verification

Command:

```powershell
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap|FullyQualifiedName~DevelopmentProfile"
```

Result:

- PASS
- 107 passed
- 0 failed
- 0 skipped

Notes:

- Existing warning-only nullable/analyzer messages remain.
- One stale test dependency on deleted `Start-DevelopmentProfile.ps1` was corrected to validate the current canonical Visual Studio launch profile and active development policy instead.

## Read-Only Database Verification

Database checked:

- `obm_pos_dev_v0_pg`

Verification mode:

- PostgreSQL transaction opened as `READ ONLY`
- Query completed
- Transaction rolled back

Observed values:

- `TblEmployeePermission = 7`
- `TblEmployee = 20`
- `TblLocalOutbox = 62`
- `Phase2_v002_marker = 1`
- `RuntimeState = Activated`

No database mutation was performed.

## Security

- No passwords were printed.
- No connection string was printed.
- No token, secret, Google credential, cookie, or protected local secret was printed.
- No User or Machine environment variable write was added.
- Only process-scoped runtime transition is used for the same-process handoff.

## Retest Instructions

Operator retest route A:

1. Start WPF from Visual Studio using the canonical Installation V0 development profile.
2. Confirm the screen shows build label `prompt043`.
3. Click `Open OBM-POS`.
4. Expected: no `DevelopmentDatabaseRejected`; runtime handoff should evaluate as `VerifiedInstallationHandoff`.

Operator retest route B:

1. Start WPF from Visual Studio using the canonical runtime development profile.
2. Keep the approved V0 ProductRoot.
3. Expected: startup guard accepts only approved ProductRoot and approved V0 database.

## Final Classification

OBM_POS_VERIFIED_RUNTIME_HANDOFF_PROVENANCE_READY_FOR_USER_RETEST

