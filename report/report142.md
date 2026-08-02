# report142 - Active LocalPosStartupService bootstrap gate repair

## Verdict

`BLOCKED_PHYSICAL_WPF_LANE_NOT_RUN_SOURCE_GATE_REPAIRED_AND_TESTED`

Source correction was implemented and verified by focused tests/builds. Physical WPF approval was not claimed because no operator-visible WPF session was driven in this prompt.

## Final Canonical Startup Authority

The active authority is now:

```text
App.xaml.cs
-> LocalPosStartupService.AssessAsync
-> BootstrapCompletionVerifier.VerifyAsync
-> RuntimeModeGate
-> StartNormalApplicationAsync
-> ShowMainWindowForActivatedRuntimeAsync
```

`BootstrapCompletionVerifier` is the single shared semantic definition for local ApplicationReady/Activated eligibility. It lives in `4POS/NailSalonNet8/InstallationV0/Phase2/BootstrapCompletionVerifier.cs`, which is referenced by the WPF app and InstallationV0.

## Before C# Call Chain

| Order | File | Method | Condition | Next call |
|---:|---|---|---|---|
| 1 | `4POS/NailSalonNet8/App.xaml.cs` | constructor | calls `LocalPosStartupService.AssessAsync` | runtime mode gate |
| 2 | `4POS/NailSalonNet8/Services/Startup/LocalPosStartupService.cs` | `AssessAsync` | `RuntimeState='Activated'` + Tenant/POS match | returned `Ready` |
| 3 | `4POS/NailSalonNet8/App.xaml.cs` | `StartNormalApplicationAsync` | `CanStartOperationalWorkers` | `ShowMainWindowForActivatedRuntimeAsync` |
| 4 | `4POS/NailSalonNet8/Services/Startup/RuntimeProfileStartupAssessmentService.cs` | `VerifyBootstrapBaselineAsync` | marker/baseline predicate existed here only | not on active startup route |

## After C# Call Chain

| Order | File | Method | Condition | Next call |
|---:|---|---|---|---|
| 1 | `4POS/NailSalonNet8/App.xaml.cs` | constructor | calls `LocalPosStartupService.AssessAsync` | runtime mode gate |
| 2 | `4POS/NailSalonNet8/Services/Startup/LocalPosStartupService.cs` | `AssessAsync` | Activated row + Tenant/POS match | shared completion verifier |
| 3 | `4POS/NailSalonNet8/InstallationV0/Phase2/BootstrapCompletionVerifier.cs` | `VerifyAsync` | marker v004 + approved baseline proof | `Ready` or `InstallationIncomplete` |
| 4 | `4POS/NailSalonNet8/Services/Startup/RuntimeProfileStartupAssessmentService.cs` | `VerifyInstalledLocalRuntimeAsync` | delegates to same verifier | no duplicate predicate |
| 5 | `4POS/NailSalonNet8/InstallationV0/Phase2/Phase2StartupHydrationService.cs` | `HydrateAsync` | delegates to same verifier before `V003Complete` | Open button agrees with startup |

## Files/Classes/Methods Changed

| File | Class/method | Change |
|---|---|---|
| `4POS/NailSalonNet8/InstallationV0/Phase2/BootstrapCompletionVerifier.cs` | `BootstrapCompletionVerifier.VerifyAsync` | New shared read-only verifier |
| `4POS/NailSalonNet8/Services/Startup/LocalPosStartupService.cs` | `AssessAsync` | Calls verifier before returning `Ready` |
| `4POS/NailSalonNet8/Services/Startup/RuntimeProfileStartupAssessmentService.cs` | `VerifyInstalledLocalRuntimeAsync` | Removed independent marker/baseline predicate and delegates to verifier |
| `4POS/NailSalonNet8/InstallationV0/Phase2/Phase2StartupHydrationService.cs` | `HydrateAsync` | Uses shared verifier before reporting complete |
| `4POS/NailSalonNet8.Tests/Startup/LocalPosStartupResultTests.cs` | focused tests/fake DB | Added missing marker/table, incomplete baseline, invalid PIN, no-mutation coverage |
| `4POS/NailSalonNet8.Tests/Startup/RuntimeProfileStartupAssessmentServiceTests.cs` | fake DB | Updated fake to match shared verifier |

## Readiness Predicate After Correction

`LocalPosStartupService` still requires:

```text
PostgreSQL provider
dbo schema
runtime lifecycle tables
exactly one Activated runtime profile
complete runtime identity
matching Tenant row
matching POS row
```

Before returning `Ready`, it now also requires:

```text
Phase2TrialCompletionMarker table exists
matching marker count = 1
marker Version = Phase2TrialConstants.Version
marker Status = Completed
TenantGuid/PosGuid match the activated runtime identity
TblEmployeePermission table exists and has tenant rows
TblEmployee table exists and has tenant employee rows with valid permission parents
employee operational PIN policy has zero invalid/duplicate rows
```

Failure maps to:

```text
LocalPosStartupDecision.InstallationIncomplete
CanEnterNormalApplication = false
Result codes:
- LOCAL_BOOTSTRAP_COMPLETION_PROOF_MISSING
- LOCAL_BOOTSTRAP_BASELINE_INCOMPLETE
```

## Semantic Divergence Removed

The previous independent predicate in `RuntimeProfileStartupAssessmentService` was removed. That service now delegates to `BootstrapCompletionVerifier.VerifyAsync`.

`Phase2StartupHydrationService` also delegates to the same verifier before returning `V003Complete`, so the InstallationV0 Open button and normal WPF startup no longer use separate completion meanings.

## Test/Build Totals

| Command | Result |
|---|---|
| `dotnet test ... --filter "FullyQualifiedName~Startup"` | Passed: 67, Failed: 0, Skipped: 0 |
| `dotnet test ... --filter "FullyQualifiedName~InstallationV0"` | Passed: 69, Failed: 0, Skipped: 0 |
| `dotnet build ...\InstallationV0.csproj` | 0 warnings, 0 errors |
| `dotnet build ...\NailSalonNet8.csproj` | 0 warnings, 0 errors |

Initial sandboxed test attempt failed before build because MSBuild could not read the Windows SDK cache under the user profile; rerun outside sandbox passed.

## Count-Only DB Evidence

No DB inspection or mutation was performed in this prompt. Existing DB was not reset, migrated, seeded, or patched.

## Destructive Operation Counts

| Operation | Count |
|---|---:|
| DB drop/create/reset/copy | 0 |
| Manual marker/runtime/baseline insert/update | 0 |
| Migration/seed execution | 0 |
| Source rollback/revert | 0 |
| Secret/config/ProductRoot edits | 0 |

## Physical Result

Physical WPF lane was not run, so PASS verdict is not claimed.

Required remaining physical proof:

```text
launch latest WPF
verify Activated + incomplete baseline blocks MainWindow
verify InstallationV0/repair UI opens
run Install/Resume once
verify marker + baseline + Activated
verify installation outbox rows remain zero
verify MainWindow responsive with API offline
restart twice and verify direct MainWindow without InstallationV0 flash
```

## Private Artifact

No separate private evidence artifact was created for prompt142. Evidence is the local source diff plus command output in this Codex session.

## Coordination Commit SHA

`RETURNED_IN_CODEX_FINAL_RESPONSE`
