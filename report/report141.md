# report141 - WPF startup route audit after prompt140

## 1. Verdict

`BLOCKED_PROMPT141_LIVE_RUNNING_BINARY_AND_BRANCH_OBSERVATION_NOT_AVAILABLE_CSHARP_BYPASS_PROVEN`

The live physical process was not available for read-only inspection during this prompt:

```powershell
Get-Process NailSalonNet8 -ErrorAction SilentlyContinue
Get-Process InstallationV0 -ErrorAction SilentlyContinue
```

Both returned no process. Therefore PID, exact loaded executable, visible window build label, loaded assembly label, and actual debugger branch values could not be captured.

Static C# control-flow evidence is conclusive for the source tree: the prompt140 baseline/marker guard was added to `RuntimeProfileStartupAssessmentService`, but the active App startup route uses `LocalPosStartupService`. `LocalPosStartupService` can return `Ready` from `RuntimeState='Activated'` plus Tenant/POS identity and does not execute the prompt140 `Phase2TrialCompletionMarker` / baseline-count guard.

Primary classification: `C5_BASELINE_VALIDATION_NOT_EXECUTED`.

Contributing classification: `C2_NORMAL_STARTUP_BYPASSES_PROMPT140_ASSESSMENT`.

## 2. Running Binary Proof

Read-only process proof:

| Item | Result |
|---|---|
| `NailSalonNet8` process | Not running during inspection |
| `InstallationV0` process | Not running during inspection |
| PID | Not available |
| Loaded executable path | Not available from process inspection |
| Visible build label | Not available because no live UI was present |
| Running compiled `InstallationV0BuildInfo` value | Not available because no live process was present |

Build output files found under the source tree:

| File | Timestamp UTC | File version | Product version |
|---|---:|---|---|
| `E:\Project2026\4POS\NailSalonNet8\bin\Debug\net8.0-windows\NailSalonNet8.exe` | `2026-08-02 22:47:20Z` | `1.0.1.0` | `1.0.1+b50a167be3cf4c8cef39f665aa5f841bc157bd5d` |
| `E:\Project2026\4POS\NailSalonNet8\bin\Debug\net8.0-windows\NailSalonNet8.dll` | `2026-08-02 22:47:20Z` | `1.0.1.0` | `1.0.1+b50a167be3cf4c8cef39f665aa5f841bc157bd5d` |
| `E:\Project2026\4POS\NailSalonNet8\bin\Debug\net8.0-windows\InstallationV0.dll` | `2026-08-02 22:47:10Z` | `1.0.0.0` | `1.0.0+b50a167be3cf4c8cef39f665aa5f841bc157bd5d` |

Source label:

```csharp
// 4POS/NailSalonNet8/InstallationV0/Application/InstallationV0BuildInfo.cs
namespace OBM.InstallationV0.Application;

public static class InstallationV0BuildInfo
{
    public const string CoordinationPromptLabel = "prompt140";
}
```

Binary string scan of `InstallationV0.dll` did not find `prompt140`, `prompt139`, or `prompt141`; the live compiled label remains unproven without a running process/window.

## 3. Full Startup Call-Chain Table

| Order | File | Class | Method | Input | Condition evaluated | Returned decision | Next call |
|---:|---|---|---|---|---|---|---|
| 1 | `4POS/NailSalonNet8/App.xaml.cs` | `App` | constructor | WPF startup | `_installationV0Requested = InstallationV0Module.IsRequestedFromEnvironment()` | Registers InstallationV0 open callback | Builds configuration |
| 2 | `4POS/NailSalonNet8/App.xaml.cs` | `App` | constructor | local config + connection string | `LocalPosStartupService.AssessAsync(...)` | `LocalPosStartupResult` | Enters normal/bootstrap gate |
| 3 | `4POS/NailSalonNet8/Services/Startup/LocalPosStartupService.cs` | `LocalPosStartupService` | `AssessAsync` | local PostgreSQL connection | DB/schema/runtime/identity checks only | `Ready` if checks pass | `RuntimeModeGate.EnterNormalMode` |
| 4 | `4POS/NailSalonNet8/App.xaml.cs` | `App` | `OnStartup` | startup args | no InstallationV0 callback here unless gate blocks | calls normal startup | `StartNormalApplicationAsync(e.Args)` |
| 5 | `4POS/NailSalonNet8/App.xaml.cs` | `App` | `StartNormalApplicationAsync` | `_runtimeModeGate` | if `CanStartOperationalWorkers` false then InstallationV0; otherwise continue | proceeds to MainWindow | `ShowMainWindowForActivatedRuntimeAsync` |
| 6 | `4POS/NailSalonNet8/App.xaml.cs` | `App` | `ShowMainWindowForActivatedRuntimeAsync` | DI services | existing visible MainWindow or construct new | `MainWindowShownResult` | `MainWindow.Show()` |
| 7 | `4POS/NailSalonNet8/InstallationV0/Presentation/InstallationV0Window.cs` | `InstallationV0Window` | `OpenPosAsync` | button click | requires `_phase2Startup?.IsV003Complete == true` | blocks or calls module | `InstallationV0Module.RequestOpenObmPosAsync` |
| 8 | `4POS/NailSalonNet8/App.xaml.cs` | `App` | `OpenInstalledPosFromInstallationV0Async` | verified product root | retries startup assessment using `ILocalPosStartupService` | if `CanEnterNormalApplication`, opens normal path | `StartNormalApplicationAsync(..., forceNormalApplication: true)` |

## 4. Route 1 Versus Route 2

### Route 1 - InstallationV0 Open button

`InstallationV0Window.OpenPosAsync` first checks the UI hydration result:

```csharp
if (_phase2Startup?.IsV003Complete != true)
{
    var blocked = MainWindowOpenResult.Blocked(
        "INSTALLATION_V0_COMPLETED_PROOF_MISSING",
        "InstallationV0OpenObmPosHandoff",
        "Local Phase 2 completion proof is missing.",
        _phase2Startup?.RuntimeState ?? "Unknown");
    InstallationV0Module.RecordOpenObmPosResult(InstallationV0Module.FormatRouteResult(blocked));
    SetOpenPosFailed(blocked);
    return;
}

routeResult = await InstallationV0Module.RequestOpenObmPosAsync(_service.ProductRoot);
```

However, after this UI-specific check passes, the App callback reruns the active normal assessment:

```csharp
var assessment = await RetryStartupAssessmentAsync();
if (!assessment.CanEnterNormalApplication)
{
    return RouteFromAssessment(assessment, mainWindowConstructed: false, mainWindowShown: false, mainWindowVisible: false);
}

var routeResult = await StartNormalApplicationAsync(Array.Empty<string>(), forceNormalApplication: true);
```

`RetryStartupAssessmentAsync` resolves `ILocalPosStartupService`, not `RuntimeProfileStartupAssessmentService`.

### Route 2 - Normal startup/restart

Normal startup performs the same active local assessment in the App constructor:

```csharp
_startupAssessment = _localPosStartupService.AssessAsync(
        Configuration,
        databaseProvider,
        connectionStringName,
        conn!,
        CreateDbConnection)
    .GetAwaiter()
    .GetResult();

if (_startupAssessment.CanEnterNormalApplication)
{
    _runtimeModeGate.EnterNormalMode(_startupAssessment);
}
```

`OnStartup` then routes to:

```csharp
await StartNormalApplicationAsync(e.Args);
```

If the gate is normal, MainWindow opens:

```csharp
if (!_runtimeModeGate.CanStartOperationalWorkers)
{
    ShowInstallationV0DiagnosticWindow(_runtimeModeGate.LastAssessment?.TechnicalSummary ?? "Startup assessment is not Ready.");
    return RouteFromAssessment(_runtimeModeGate.LastAssessment, mainWindowConstructed: false, mainWindowShown: false, mainWindowVisible: false);
}

visibleRouteResult = await ShowMainWindowForActivatedRuntimeAsync(appHost.Services);
```

Conclusion: both routes converge on `LocalPosStartupService` for final MainWindow eligibility. The prompt140 `RuntimeProfileStartupAssessmentService` guard is not the normal startup authority.

## 5. Complete Relevant C# Predicate Excerpts

### MainWindow eligibility

```csharp
// 4POS/NailSalonNet8/App.xaml.cs
if (!_runtimeModeGate.CanStartOperationalWorkers)
{
    WriteTargetStartupDiagnostic(
        $"StartNormalApplicationAsync blocked mode={_runtimeModeGate.Mode} state={_runtimeModeGate.LastAssessment?.State.ToString() ?? "(none)"}");
    ShowInstallationV0DiagnosticWindow(_runtimeModeGate.LastAssessment?.TechnicalSummary ?? "Startup assessment is not Ready.");
    return RouteFromAssessment(_runtimeModeGate.LastAssessment, mainWindowConstructed: false, mainWindowShown: false, mainWindowVisible: false);
}

visibleRouteResult = await ShowMainWindowForActivatedRuntimeAsync(appHost.Services);
```

### MainWindow launch

```csharp
// 4POS/NailSalonNet8/App.xaml.cs
var win = services.GetRequiredService<MainWindow>();
Current.MainWindow = win;
win.Show();
win.Activate();
```

### Active startup assessment predicate

```csharp
// 4POS/NailSalonNet8/Services/Startup/LocalPosStartupService.cs
var runtimeProfileRows = await ScalarLongAsync(conn,
    """
    SELECT COUNT(*)
    FROM dbo."TblPosRuntimeProfile"
    WHERE "RuntimeState" = 'Activated'
    """,
    cancellationToken);
if (runtimeProfileRows != 1)
{
    return Complete(LocalPosStartupDecision.InstallationIncomplete,
        "The local runtime activation projection is incomplete.",
        "MANDATORY_RUNTIME_PROFILE_ABSENT: expected exactly one Activated TblPosRuntimeProfile row.",
        databaseProvider,
        connectionStringName,
        connectionString,
        sw);
}
```

```csharp
// 4POS/NailSalonNet8/Services/Startup/LocalPosStartupService.cs
var tenantInDb = await ScalarLongAsync(conn,
    "SELECT COUNT(*) FROM dbo.\"TblTenant\" WHERE \"TenantGuid\" = @tenant",
    cancellationToken,
    ("tenant", identity.TenantGuid));

var posInDb = await ScalarLongAsync(conn,
    """
    SELECT COUNT(*)
    FROM dbo."TblPosLocal"
    WHERE "TenantGuid" = @tenant
      AND "PosGuid" = @pos
      AND "TenantId" = @slot
    """,
    cancellationToken,
    ("tenant", identity.TenantGuid),
    ("pos", identity.PosGuid),
    ("slot", identity.PosSlotNumber));
```

```csharp
// 4POS/NailSalonNet8/Services/Startup/LocalPosStartupService.cs
return Complete(LocalPosStartupDecision.Ready,
    "POS database is ready.",
    "PostgreSQL connection, EF schema, runtime profile, and local Tenant/POS identity passed.",
    databaseProvider,
    connectionStringName,
    connectionString,
    sw,
    localIdentity: identity);
```

No `Phase2TrialCompletionMarker`, permission-count, or employee-count validation appears in this active `Ready` branch.

### Prompt140 guard in non-active assessment service

```csharp
// 4POS/NailSalonNet8/Services/Startup/RuntimeProfileStartupAssessmentService.cs
var baseline = await VerifyBootstrapBaselineAsync(connection, profile, cancellationToken);
var localRuntimeReady = tenantTablePresent &&
    posTablePresent &&
    tenantConsistent &&
    posConsistent &&
    baseline.Ready &&
    profile.RuntimeState == PosRuntimeState.Activated &&
    IsSupportedSchemaVersion(profile.SchemaVersion);
```

```csharp
// 4POS/NailSalonNet8/Services/Startup/RuntimeProfileStartupAssessmentService.cs
var markerTablePresent = await ScalarBoolAsync(connection, "SELECT to_regclass('dbo.\"Phase2TrialCompletionMarker\"') IS NOT NULL", cancellationToken);
var permissionTablePresent = await ScalarBoolAsync(connection, "SELECT to_regclass('dbo.\"TblEmployeePermission\"') IS NOT NULL", cancellationToken);
var employeeTablePresent = await ScalarBoolAsync(connection, "SELECT to_regclass('dbo.\"TblEmployee\"') IS NOT NULL", cancellationToken);
if (!markerTablePresent || !permissionTablePresent || !employeeTablePresent)
{
    return new(false, 0, 0, 0);
}
```

```csharp
// 4POS/NailSalonNet8/Services/Startup/RuntimeProfileStartupAssessmentService.cs
return new(markerCount == 1 && permissionCount >= 7 && employeeCount >= 20, markerCount, permissionCount, employeeCount);
```

This code correctly blocks when marker/baseline is missing, but current App startup does not call it.

## 6. Marker/Baseline SQL and C# Owner Comparison

Migration SQL creates:

```sql
CREATE TABLE IF NOT EXISTS dbo."Phase2TrialCompletionMarker" (
    "MarkerGuid" uuid NOT NULL DEFAULT gen_random_uuid(),
    "TenantGuid" uuid NOT NULL,
    "PosGuid" uuid NOT NULL,
    "Version" text NOT NULL,
    "Status" text NOT NULL,
    "CreatedAtUtc" timestamp with time zone NOT NULL DEFAULT NOW(),
    CONSTRAINT "PK_Phase2TrialCompletionMarker" PRIMARY KEY ("MarkerGuid")
);

CREATE UNIQUE INDEX IF NOT EXISTS "UX_Phase2TrialCompletionMarker_Tenant_Pos_Version"
    ON dbo."Phase2TrialCompletionMarker" ("TenantGuid", "PosGuid", "Version");
```

Phase2 seed inserts/verifies marker before activation:

```csharp
var marker = targetRows.Single(row => row.TableName == Phase2TrialConstants.CompletionMarkerTable);
await AssertMarkerHardGateAsync(connection, transaction, plan.Identity, targetDatabaseName, marker, permissionRows, employeeRows, cancellationToken);
if (!await ExistsAsync(connection, transaction, marker, cancellationToken))
{
    await InsertAsync(connection, transaction, marker, cancellationToken);
    inserted++;
}
```

Prompt140 startup guard query uses the same schema/table/case:

```csharp
SELECT COUNT(*)
FROM dbo."Phase2TrialCompletionMarker"
WHERE "Version" = @version
  AND "TenantGuid" = @tenantGuid
  AND "PosGuid" = @posGuid
  AND "Status" = 'Completed';
```

The query itself is not a false positive. The defect is that this query lives in `RuntimeProfileStartupAssessmentService`, while the active normal startup authority is `LocalPosStartupService`.

## 7. Runtime Values at the Branch, Sanitized

Live branch values could not be collected because no `NailSalonNet8`/`InstallationV0` process was available. Minimum debugger observations still needed from operator if physical confirmation is required:

1. Breakpoint `4POS/NailSalonNet8/App.xaml.cs` constructor at `LocalPosStartupService start`.
2. Breakpoint `4POS/NailSalonNet8/Services/Startup/LocalPosStartupService.cs` at the final `Complete(LocalPosStartupDecision.Ready, ...)`.
3. Breakpoint `4POS/NailSalonNet8/App.xaml.cs` at `visibleRouteResult = await ShowMainWindowForActivatedRuntimeAsync(...)`.
4. Observe sanitized values: build label, ProductRoot, DB name, `runtimeProfileRows`, `tenantInDb`, `posInDb`, `_startupAssessment.State`, and whether `RuntimeProfileStartupAssessmentService.VerifyBootstrapBaselineAsync` is hit.

Previously proven sanitized DB state from the prompt140 physical lane showed:

| Value | Sanitized result |
|---|---|
| runtime profile row count | `1` |
| runtime profile state | `Activated` |
| Phase2 marker table | missing |
| setup/login/payment/baseline owner counts | `0` for required baseline owners |
| local outbox total | `0` |

Those values are sufficient to explain why `RuntimeProfileStartupAssessmentService` would block, but not enough to prove the exact physical branch without the live process.

## 8. Duplicate/Stale Owner Inventory

| Owner/path | Classification | Evidence |
|---|---|---|
| `App.xaml.cs` + `LocalPosStartupService` | canonical active owner | Constructor and retry path both call `LocalPosStartupService.AssessAsync` |
| `RuntimeProfileStartupAssessmentService` | duplicate/parallel route | Contains prompt140 marker/baseline guard but is not called by App startup path found in source search |
| `InstallationV0Window` + `Phase2StartupHydrationService` | active InstallationV0 UI-only gate | Enables/disables Open button using Phase2 hydration result |
| `InstallationV0VerificationService` | parallel verification owner | Has marker/baseline verification, but not shown as normal startup gate |
| `LocalApplicationReadinessExecutor` | obsolete but still present | Still writes ApplicationReady-style runtime state; source search found class but no active construction in Window flow |
| `PostgresPosRuntimeProfileRepository` | repository/helper | Reads activated profile; not the final MainWindow route decision |
| tests under `NailSalonNet8.Tests` | test-only | Static/source tests assert strings but do not prove physical branch |

## 9. Exact Root-Cause Classification

Primary: `C5_BASELINE_VALIDATION_NOT_EXECUTED`.

Reason: the App startup code evaluates `LocalPosStartupService`, whose `Ready` branch does not execute marker/baseline validation. The prompt140 marker/baseline validation exists in `RuntimeProfileStartupAssessmentService`, but that service is a parallel owner, not the active startup authority for normal process startup or the final Open button handoff.

This is not `C4_MARKER_QUERY_FALSE_POSITIVE`; the marker SQL in the prompt140 service is case-correct and returns blocked when the table or required counts are missing.

This is not proven as `C1_RUNNING_STALE_BINARY` because no live process was available. Build output string scan also did not prove the compiled prompt label, so stale binary remains an uncollected physical observation, not the primary source-level root cause.

## 10. Minimal Proposed Correction

Do not implement in this prompt.

Minimal correction for a later approved prompt:

1. Make `LocalPosStartupService` execute the same mandatory Phase2 completion marker and baseline-count checks before returning `LocalPosStartupDecision.Ready`; or
2. Replace the App startup authority with a single canonical assessment service that includes those checks and is used by both normal startup and InstallationV0 Open handoff.

The smallest safe correction is option 1 because current App wiring already treats `LocalPosStartupService` as the canonical active owner.

## 11. Exact Files/Methods That Would Change Later

Later approved source changes should be limited to:

| File | Method |
|---|---|
| `4POS/NailSalonNet8/Services/Startup/LocalPosStartupService.cs` | `AssessAsync` |
| `4POS/NailSalonNet8/Services/Startup/LocalPosStartupService.cs` | new/private baseline verification helper, or shared helper reused from existing guard |
| `4POS/NailSalonNet8.Tests/Startup/LocalPosStartupResultTests.cs` or focused startup test file | tests proving missing marker/baseline blocks normal startup |
| `4POS/NailSalonNet8.Tests/InstallationV0/InstallationV0Phase1Tests.cs` | optional static wiring assertion that active startup owner contains marker/baseline checks |

Areas that should remain unchanged: pairing, API token flow, SignalR, outbox sync, POS checkout, payment, employee/customer/business tables, launchSettings, ProductRoot configuration, and database contents.

## 12. Build/Test Status

No build or tests were run for prompt141. This prompt was read-only investigation/report only.

## 13. Zero-Mutation and Zero-Secret Confirmation

No C#, SQL, configuration, launchSettings, ProductRoot, DB state, migration, seed, token, pairing code, or runtime state was changed. No password, connection string, raw GUID, token, pairing code, Google identity, employee name, or private business payload is included.

