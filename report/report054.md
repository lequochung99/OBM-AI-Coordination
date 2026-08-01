# Report 054 - Installation V0 Dependency Graph Audit

## 1. Verdict

INSTALLATION_DEPENDENCY_GRAPH_AUDIT_READY_FOR_CANONICAL_V001

## 2. Scope

Prompt executed: `prompt/prompt054.md`

Audit scope:

- `E:\Project2026\4POS\NailSalonNet8`
- `E:\Project2026\4POS\NailSalonNet8.Tests`
- `E:\Project2026\1ApiServer\ApiServer01`
- Prior reports: `report044`, `report045`, `report049`, `report050`, `report051`, `report052`, `report053`

No WPF/API process was started, stopped, or restarted. No PostgreSQL data/schema was read by connection or mutated.

## 3. Evidence Folder

Evidence was written only under:

`E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001`

Key artifacts:

- `README.md`
- `graphify\graphify-out\graph.json`
- `installation-flow-current.mmd`
- `installation-flow-target.mmd`
- `installation-flow-subgraph.json`
- `edge-inventory.csv`
- `symbol-action-inventory.csv`
- `redundant-link-candidates.md`
- `dynamic-edge-checklist.md`
- `aspnet-identity-trace-audit.md`
- `source-search-*.txt`
- `SHA256SUMS.txt`

## 4. Graphify Result

Graphify was available at `C:\Users\lequo\.local\bin\graphify.exe`.

Version:

`graphify 0.9.26`

Command:

`graphify extract E:\Project2026\4POS\NailSalonNet8 --code-only --no-cluster --out E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\graphify`

Classification:

GRAPHIFY_AUDIT_PASS

Graph output:

- Nodes: 13456
- Edges: 30891
- Graph: `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\graphify\graphify-out\graph.json`

Known limitations:

- Some generated/project files produced zero graph nodes.
- SQL files were not parsed because `tree_sitter_sql` is not installed.
- No install or update was attempted.

## 5. Artifact Hashes

- `graphify\graphify-out\graph.json`: `E60A51F99BF00284D9485F8458836D64DC19F318CB9E220F5ECE6BF283901521`
- `SHA256SUMS.txt`: `9B573B5B3CAF760C88FD3FA64640FDEE372593BB7D0D31D42D4B18C36F70B786`

Full manifest:

`E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\SHA256SUMS.txt`

## 6. Current Runtime Startup Graph

Current source-level path:

`App` construction
-> `SpacePosRuntimeConfiguration.Build`
-> `DevelopmentProfileLaunchPolicy`
-> `LocalPosStartupService.AssessAsync`
-> local database startup assessment
-> `StartNormalApplicationAsync`
-> `ShowMainWindowForActivatedRuntimeAsync`
-> `MainWindow.Show`
-> `IApiSessionInitializer.StartAsync`

Important boundary:

API session initialization is after MainWindow route/opening logic. It must remain post-MainWindow and must not become a pre-MainWindow local startup gate.

## 7. InstallationV0 Open OBM-POS Handoff Graph

Current source-level path:

`InstallationV0Window`
-> `InstallationV0Module.RequestOpenObmPosAsync`
-> `App.OpenInstalledPosFromInstallationV0Async`
-> `TryApplyVerifiedInstallationV0ProductRoot`
-> `RetryStartupAssessmentAsync`
-> `LocalPosStartupService.AssessAsync`
-> normal startup route
-> `MainWindow.Show`

This path is the canonical installation-to-runtime handoff candidate.

## 8. Local Database vs API Cross Links

Observed source boundaries:

- Local startup readiness is centered on `LocalPosStartupService`.
- API/bootstrap proof code is present in InstallationV0 Phase 1 services and must stay installation-only.
- `IApiSessionInitializer` is the post-MainWindow API session boundary.
- `IAppJwtBootstrapper` remains as a compatibility name and should be removed/renamed in a later source cleanup.
- SignalR/sync workers appear operational/post-startup and were not proven to be pre-MainWindow gates.

## 9. Installation Proof Inventory

Installation-only proof concepts found:

- Phase 1 API authorization/checkpoint proof.
- Phase 2 local database proof.
- Identity-spine checks.
- Employee/permission/outbox proof checks.
- `InstallationV0CompletedReadinessService`, which appears to be the old name for an installation verification/readiness proof service.

These should not be treated as normal runtime startup gates.

## 10. Duplicate Decision Owners

Areas with duplicate or overlapping ownership:

- Local DB usable decision: `LocalPosStartupService`, legacy `RuntimeProfileStartupAssessmentService`, Phase 2 hydration/readiness services.
- MainWindow may open decision: `App.RouteFromAssessment`, `StartNormalApplicationAsync`, InstallationV0 handoff.
- API online/offline decision: InstallationV0 Phase 1 result, `ApiSessionInitializer`, and operational sync workers.
- Phase 2 complete/open POS decision: `Phase2StartupHydrationService` and InstallationV0 UI enablement/readiness services.

## 11. Old Symbol Inventory

Rename/delete candidates recorded in `symbol-action-inventory.csv`:

- `DatabaseStartupAssessmentService`: compatibility shim around `LocalPosStartupService`.
- `DatabaseStartupAssessment`: result model that should become `LocalPosStartupResult`.
- `ApplicationStartupCoordinator`: no active App startup caller found; delete candidate after recovery path review.
- `RuntimeProfileStartupAssessmentService`: ambiguous recovery/updater usage; review before merge/delete.
- `IAppJwtBootstrapper`: compatibility shim; replace with `IApiSessionInitializer`.
- `AppJwtBootstrapper`: old file/class naming; replace with `ApiSessionInitializer`.
- `InstallationV0CompletedReadinessService`: old name; replace with `InstallationV0VerificationService`.
- `BootstrapRepairRequired`: old result naming; replace with canonical recovery/repair result or remove.
- `POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED`: old route text; keep only as a forbidden-regression guard if needed.

## 12. Environment and Launch Routing

Observed source/config symbols:

- `SPACEPOS_PRODUCT_ROOT`: used for runtime ProductRoot resolution and development profile guardrails.
- `SPACEPOS_INSTALLATION_MODULE`: used for InstallationV0 diagnostics/module selection and cleared during same-process handoff.
- `DOTNET_ENVIRONMENT`: development profile marker.
- `ASPNETCORE_ENVIRONMENT`: API/server-side environment marker.
- Local runtime database path includes a development username marker `hung`; provisioning/admin paths separately reference `postgres`.

No credential values, passwords, protected secret material, or raw connection strings were copied into this report.

## 13. Runtime Username Boundary

Static evidence supports this split:

- Runtime POS startup should use the configured V0 runtime database identity.
- PostgreSQL owner/admin identity is provisioning-only.
- The audit did not connect to PostgreSQL and did not verify live credentials.

Canonical documentation should state this explicitly so future agents do not mix runtime database login with provisioning/admin ownership.

## 14. ASP.NET Identity Trace

ASP.NET Identity references were found primarily in API/Platform or legacy login paths. No evidence showed ASP.NET Identity tables as an ordinary WPF pre-MainWindow local runtime dependency.

Some WPF source/docs still contain historical password/login terminology. Those should be corrected later as documentation/source cleanup, not as part of this audit.

## 15. Dynamic Edge Checklist

Checked for:

- Reflection/assembly scanning.
- DI assembly registration.
- XAML resource dynamic references.
- JSON/config dynamic behavior.
- Environment variable access.

No hidden pre-MainWindow API/PIN/Phase-proof dependency was proven. Dynamic-edge evidence is static only and should be treated as a review guide, not a runtime proof.

## 16. Redundant Link Candidates

Top cleanup candidates before canonical V001 documentation:

| Candidate | Preliminary action |
| --- | --- |
| `DatabaseStartupAssessmentService` | Remove compatibility name after `LocalPosStartupService` is canonicalized. |
| `DatabaseStartupAssessment` | Rename to local POS startup result terminology. |
| `ApplicationStartupCoordinator` | Delete if no recovery/updater caller is required. |
| `IAppJwtBootstrapper` | Replace with `IApiSessionInitializer`. |
| `AppJwtBootstrapper.cs` | Rename/delete old bootstrapper file naming. |
| `InstallationV0CompletedReadinessService` | Rename to `InstallationV0VerificationService`. |
| `BootstrapRepairRequired` | Remove or rename to canonical local startup recovery term. |
| Historical `App.xaml.cs.codex-bak-*` files | Exclude from active source reasoning or archive/delete by operator-approved cleanup. |

## 17. Current Diagram

Current flow diagram:

`E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\installation-flow-current.mmd`

This diagram keeps installation-only and legacy/compatibility services visually separate from the normal runtime path.

## 18. Target Diagram

Target flow diagram:

`E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\installation-flow-target.mmd`

Target principle:

Local PostgreSQL runtime usability decides MainWindow route. API authorization/session initialization begins only after MainWindow is allowed to open.

## 19. Canonical V001 Documentation Requirements

Recommended next canonical document:

`E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md`

The first canonical V001 should define:

- Local database readiness is the normal runtime startup gate.
- Phase 1 API authorization proof is installation-only.
- Phase 2 database proof is installation-only.
- MainWindow must not require API/bootstrap/PIN proof before opening.
- API session initialization is post-MainWindow.
- Employee PIN/password login belongs after local runtime is available.
- Runtime database identity and provisioning database owner identity are separate.
- Old symbol names above are prohibited for new code.

## 20. Mutation / Safety Confirmation

No OBM implementation source was edited.

No canonical OBM documentation was edited.

No project file, launch profile, test file, environment variable, PostgreSQL schema/data, API state, WPF state, or running process was changed.

Only audit evidence was created under:

`E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001`

Coordination repository report created:

`report/report054.md`

Final classification:

INSTALLATION_DEPENDENCY_GRAPH_AUDIT_READY_FOR_CANONICAL_V001
