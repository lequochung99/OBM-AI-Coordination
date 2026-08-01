# Report 047 — Development database resolver correction

## 1. Verdict

OBM_POS_CANONICAL_DEVELOPMENT_DATABASE_RESOLVER_READY_FOR_USER_RETEST

## 2. Physical prompt045 failure evidence

The operator rebuilt and physically retested after prompt045. InstallationV0 diagnostics showed:

- Build label: `prompt043`
- Target DB: `obm_pos_dev_v0_pg (Development/Test)`
- ProductRoot: `E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot`
- Phase 2 Local DB Baseline: `Phase 2 v002 Complete`

Clicking `Open OBM-POS` failed at `DevelopmentStartupGuard` with:

- `ResultCode=DEVELOPMENT_DATABASE_NOT_APPROVED`
- `RejectedPredicate=DatabaseApproved`
- `LaunchProvenance=RuntimeDevelopmentProfile`
- `EffectiveProductRootSource=LaunchProfileEnvironment`
- `RootPresent=True`
- `RootApproved=True`
- `DatabaseApproved=False`
- `EnvironmentApproved=True`
- `InstallationModulePresent=False`
- `DiagnosticsModeLatched=False`
- `RuntimeHandoffAuthorized=False`
- `ProfileName=DevelopmentProfile`
- `CachedAssessmentReused=False`

The only failing predicate was database approval.

## 3. Why prompt043 build label did or did not indicate stale binary

The `prompt043` label did not prove a stale prompt045 binary by itself. Prompt045 did not require changing `InstallationV0BuildInfo`, so the physical diagnostics label could remain `prompt043` while still running newer source. Because prompt047 changes the diagnostics and guard path, the build label was updated to `prompt047`.

## 4. Exact rejected database value/source

The rejected value was the parsed app connection-string database field:

- RawDatabaseNameSource: `ConnectionStringFallback`
- RawDatabaseName: `obm_pos_v1`
- RawDatabaseNameLength: `10`
- NormalizedDatabaseName: `obm_pos_v1`
- NormalizedDatabaseNameLength: `10`
- ApprovedDatabaseName: `obm_pos_dev_v0_pg`
- ApprovedDatabaseNameLength: `17`
- OrdinalIgnoreCaseEqual: `False`
- ContainsLeadingOrTrailingWhitespace: `False`
- ContainsQuotes: `False`
- ContainsSemicolonOrConnectionStringFragment: `False`
- ContainsInvisibleUnicode: `False`
- SourceClassification: stale/default app connection-string fallback

The full connection string was not printed.

## 5. Exact root cause of DatabaseApproved=False

The post-configuration Development guard resolved the database from a stale/default app connection string when the approved V0 ProductRoot did not yet provide runtime bootstrap metadata. The diagnostics UI independently displayed the canonical local DB target from InstallationV0 constants, so the visible `Target DB` and the guard input diverged.

## 6. Pre-change database-source precedence

The pre-change post-configuration path effectively used:

1. Runtime bootstrap metadata only if it was present and loadable.
2. Parsed app connection-string database fallback.
3. Missing/rejected.

This made an approved ProductRoot with no bootstrap config fall through to stale `appsettings.json`.

## 7. Corrected canonical resolver and normalization

The corrected resolver uses this precedence after an approved ProductRoot is known:

1. Runtime bootstrap metadata: `RuntimeBootstrap`.
2. Canonical resolved configuration fallback: `ResolvedConfig`, fixed to `obm_pos_dev_v0_pg`.
3. Parsed connection-string database only as controlled fallback: `ConnectionStringFallback`.
4. Missing/rejected.

Normalization is deterministic:

- trim surrounding whitespace;
- remove only paired surrounding single or double quotes;
- compare with `StringComparison.OrdinalIgnoreCase`;
- reject full connection-string fragments;
- reject invisible control characters;
- do not use substring or wildcard matching.

## 8. Pre-configuration deferred versus post-configuration final behavior

Pre-configuration may return `DATABASE_APPROVAL_DEFERRED_UNTIL_RUNTIME_BOOTSTRAP` when the database name is genuinely unavailable. Post-configuration is final and fails closed with `DEVELOPMENT_DATABASE_NOT_APPROVED` if the resolved database is missing or not exactly canonical.

## 9. Protected database guard preservation

The guard still approves only `obm_pos_dev_v0_pg`. The tests reject full connection strings, empty/missing post-configuration values, protected/Royal/production values, legacy source values, and unknown Development lanes such as `obm_pos_dev_v99_pg`.

## 10. Prompt045 local-DB-first contract preservation

Prompt047 did not reintroduce Phase 1 checkpoint, WpfJwt, API reachability, Phase 2 count-marker, employee-count, permission-count, outbox-count, operational PIN, or provenance-as-runtime-authorization gates. After database approval, normal startup remains governed by the prompt045 local-readiness path.

## 11. Same-process Open OBM-POS behavior

`App.xaml.cs` now rebuilds a safe database source context during post-configuration and retry startup assessment. Safe diagnostics include:

- `DatabaseNameSource`
- `ResolvedDatabaseName`
- `ApprovedDatabaseName`
- `DatabaseApproved`
- `DatabaseApprovalStage`

No connection string or credential is printed.

## 12. Exact source files changed

Source files changed for prompt047:

- `E:\Project2026\4POS\NailSalonNet8\App.xaml.cs`
- `E:\Project2026\4POS\NailSalonNet8\Modules\Configuration\DevelopmentProfileLaunchPolicy.cs`
- `E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\R4DevelopmentProfileLauncherTests.cs`
- `E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs`

## 13. Build/test commands and counts

Commands run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap|FullyQualifiedName~DevelopmentProfile" -v minimal
```

Results:

- InstallationV0 build: PASS, 0 warnings, 0 errors.
- NailSalonNet8 build: PASS, 0 warnings, 0 errors.
- Filtered tests: PASS, 128 passed, 0 failed, 0 skipped.
- Test compile emitted existing nullable/analyzer warnings outside the prompt047 resolver change.

## 14. Read-only DB evidence

Read-only PostgreSQL verification used protected local configuration without printing secrets. The SQL session used `BEGIN READ ONLY` and did not run migrations, seeds, updates, deletes, inserts, grants, or schema changes.

Observed safe evidence:

- Database: `obm_pos_dev_v0_pg`
- User: `postgres`
- `dbo` base tables: `137`
- `public` base tables: `2`
- `dbo.TblEmployeePermission`: `7`
- `dbo.TblEmployee`: `20`
- `dbo.TblLocalOutbox`: `62`
- `dbo.Phase2TrialCompletionMarker`: `2`
- `dbo.TblPosRuntimeProfile`: `1`
- `dbo.TblPosRuntimeStateHistory`: `1`
- RuntimeState `Activated`: `1`

## 15. Operator physical retest steps

Route A:

1. Start `OBM-POS Runtime Development`.
2. Verify InstallationV0Window does not open.
3. Verify no `DEVELOPMENT_DATABASE_NOT_APPROVED`.
4. Verify MainWindow opens.

Route B:

1. Start InstallationV0 prompt047 diagnostics.
2. Confirm Phase 2 v002 complete.
3. Click `Open OBM-POS`.
4. Verify safe diagnostics show `DatabaseNameSource=RuntimeBootstrap` or `ResolvedConfig`.
5. Verify `ResolvedDatabaseName=obm_pos_dev_v0_pg`.
6. Verify `DatabaseApproved=True`.
7. Verify exactly one MainWindow opens and diagnostics closes.

## 16. Deferred prompt046 cleanup note

`prompt/prompt046.md` remains deferred. Prompt047 did not drop tables, remove Identity artifacts, or perform legacy ASP.NET Identity cleanup.

## 17. No secrets/no DB mutation/no source push proof

No database password, full connection string, token, or protected credential was printed. PostgreSQL access was read-only. The OBM source tree was not committed or pushed by this task. Only the coordination report is intended for commit and push.

## 18. Coordination commit SHA

Coordination commit SHA: reported by the final Codex response after this report is committed and pushed.
