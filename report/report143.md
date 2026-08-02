# report143 - Fresh-create continuation to bootstrap/ApplicationReady

## Verdict

`BLOCKED_PHYSICAL_WPF_LANE_NOT_RUN_SOURCE_CONTINUATION_REPAIRED_AND_TESTED`

Source correction was implemented and verified by focused builds/tests. Final physical PASS is not claimed because the operator-visible WPF lane was not driven in this Codex session.

## Root-Cause Classification

`POST_MIGRATION_BOOTSTRAP_RETRY_DISABLED_BY_ADMIN_PASSWORD_CLEARED`

Exact dead-end predicate:

```csharp
LocalDatabaseInputsArePresentAndSafe()
```

Before this prompt, that predicate required `_adminPassword.Password` even after `EnsureLocalDatabaseSetupAsync()` had already persisted protected runtime DB settings and then cleared the provisioning admin password in `finally`. If bootstrap was incomplete after migration, `HydratePhase2Async()` recomputed the command state, saw local DB inputs as invalid, and disabled `Install/Resume Local Database` instead of allowing bootstrap resume.

## Before Call Chain

| Order | File | Class | Method | Input state | Condition | Returned state | Next call |
|---:|---|---|---|---|---|---|---|
| 1 | `4POS/NailSalonNet8/InstallationV0/Presentation/InstallationV0Window.cs` | `InstallationV0Window` | `RunPhase2Async` | checkpoint + valid DB/admin inputs | button click | busy | `EnsureLocalDatabaseSetupAsync` |
| 2 | same | `InstallationV0Window` | `EnsureLocalDatabaseSetupAsync` | target absent/schema-only | create/resume + migrations | success | clears admin password |
| 3 | same | `InstallationV0Window` | `RunPhase2Async` | protected config exists | invokes `PostgreSqlPhase2ReferenceSeedExecutor` | success/failure | `HydratePhase2Async` |
| 4 | same | `InstallationV0Window` | `HydratePhase2Async` | bootstrap incomplete | calls `CanEnableLocalInstall` | disabled if admin password empty | operator dead-end |
| 5 | `4POS/NailSalonNet8/InstallationV0/Application/InstallationV0CommandRules.cs` | `InstallationV0CommandRules` | `Evaluate` | invalid local inputs | `LocalDatabaseInputsValid` was not required for `canInstall` | ambiguous/old result | button state drift |

## After Call Chain

| Order | File | Class | Method | Input state | Condition | Returned state | Next call |
|---:|---|---|---|---|---|---|---|
| 1 | `InstallationV0Window.cs` | `InstallationV0Window` | `RunPhase2Async` | checkpoint + DB inputs | no protected config | create/resume + migrate | bootstrap |
| 2 | same | `InstallationV0Window` | `RunPhase2Async` | protected config present | `CanResumeBootstrapFromProtectedLocalConfig()` | skips admin-required create step | bootstrap |
| 3 | same | `InstallationV0Window` | `RunPhase2Async` | migrated/schema-only/partial | `PostgreSqlPhase2ReferenceSeedExecutor.ExecuteAsync` | idempotent bootstrap | `HydratePhase2Async` |
| 4 | same | `InstallationV0Window` | `HydratePhase2Async` | bootstrap incomplete | protected config is enough for resume | `Install/Resume` re-enabled | retry allowed |
| 5 | `InstallationV0CommandRules.cs` | `InstallationV0CommandRules` | `Evaluate` | invalid inputs without protected config | `LocalDatabaseInputsValid` required | disabled with safe reason | operator-visible status |

## Files/Methods Changed

| File | Method/Type | Change |
|---|---|---|
| `4POS/NailSalonNet8/InstallationV0/Application/InstallationV0BuildInfo.cs` | `CoordinationPromptLabel` | Updated visible label to `prompt143` |
| `4POS/NailSalonNet8/InstallationV0/Application/InstallationV0CommandRules.cs` | `Evaluate` | Requires `LocalDatabaseInputsValid` before enabling install/resume |
| same | `ResolveInstallDisabledCode` | Adds `LOCAL_INSTALL_RESUME_INVALID_LOCAL_DB_INPUTS` |
| `4POS/NailSalonNet8/InstallationV0/Presentation/InstallationV0Window.cs` | `RunPhase2Async` | Skips admin-required create/migrate when protected local config exists, then resumes bootstrap |
| same | `LocalDatabaseInputsArePresentAndSafe` | Treats protected local runtime config as sufficient for bootstrap resume |
| same | `CanResumeBootstrapFromProtectedLocalConfig` | New safe config-read helper; no secrets emitted |
| `4POS/NailSalonNet8.Tests/InstallationV0/InstallationV0Phase1Tests.cs` | focused static/command tests | Updated label assertions and invalid-input command-state expectation |

## Behavior Table

| State | Behavior after prompt143 |
|---|---|
| DB absent | one Install/Resume action persists protected config, creates DB, applies migrations, then immediately invokes approved bootstrap executor |
| DB schema-only with protected config | skips admin-required create step and resumes bootstrap |
| DB partially bootstrapped | resumes approved bootstrap idempotently |
| DB complete | shared verifier/hydration reports complete; Open OBM-POS eligible |
| unsafe existing business/user data | remains blocked by existing clean DB safety classification; no destructive action |

## Build/Test Totals

| Command | Result |
|---|---|
| `dotnet test ... --filter "FullyQualifiedName~InstallationV0"` | Passed: 69, Failed: 0, Skipped: 0 |
| `dotnet test ... --filter "FullyQualifiedName~Startup"` | Passed: 67, Failed: 0, Skipped: 0 |
| `dotnet build ...\InstallationV0.csproj` | 0 warnings, 0 errors |
| `dotnet build ...\NailSalonNet8.csproj` | 0 warnings, 0 errors |

## Label Evidence

Source evidence:

```text
InstallationV0BuildInfo.CoordinationPromptLabel = "prompt143"
```

`rg prompt140` over the InstallationV0 source/test scope returned no matches.

Note: a stale older `bin\Debug\net8.0-windows\InstallationV0.dll` still exists locally and may still contain older label content. The operator should rebuild/debug from the updated source and confirm the visible UI label reads `prompt143`.

## Destructive Operation Counts

| Operation | Count |
|---|---:|
| DB drop/recreate/reset/copy | 0 |
| Manual marker/runtime/baseline insert/update | 0 |
| Migration/seed execution by Codex against operator DB | 0 |
| Secret/config/ProductRoot edit by Codex | 0 |
| API/pairing/sync/SignalR changes | 0 |

## Physical Result

Physical lane was not run by Codex. Required operator proof remains:

```text
launch latest WPF debug and confirm label prompt143
use existing protected checkpoint
click Install/Resume once on absent/schema-only DB
verify create/resume -> migrations -> bootstrap without dead-end
verify pending migrations = 0
verify completion marker + approved baseline
verify exactly one Activated/ApplicationReady runtime profile after bootstrap
verify installation/bootstrap TblLocalOutbox rows = 0
open MainWindow and keep responsive with API offline
restart twice and verify direct MainWindow without InstallationV0 flash
```

## Sanitization

No passwords, connection strings, pairing codes, raw GUIDs, tokens, employee names, or private business payloads are included.

