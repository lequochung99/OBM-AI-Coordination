# Prompt 047 — Fix false Development database rejection after local-DB-first startup simplification

## Physical operator evidence

Read completely before changing source:

```text
report/report045.md
report/report043.md
report/report044.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

The operator rebuilt the WPF source and physically retested after prompt045.

The InstallationV0 diagnostics window showed:

```text
Build label: prompt043
Target DB: obm_pos_dev_v0_pg (Development/Test)
ProductRoot: E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
Phase 2 Local DB Baseline: Phase 2 v002 Complete
```

Clicking `Open OBM-POS` still failed with:

```text
DEVELOPMENT_DATABASE_NOT_APPROVED

ResultCode=DEVELOPMENT_DATABASE_NOT_APPROVED
StageId=DevelopmentStartupGuard
RejectedPredicate=DatabaseApproved
LaunchProvenance=RuntimeDevelopmentProfile
EffectiveProductRootSource=LaunchProfileEnvironment
RootPresent=True
RootApproved=True
DatabaseApproved=False
EnvironmentApproved=True
InstallationModulePresent=False
DiagnosticsModeLatched=False
RuntimeHandoffAuthorized=False
ProfileName=DevelopmentProfile
CachedAssessmentReused=False
```

This evidence is authoritative.

Important interpretation:

- `Build label: prompt043` does **not by itself prove a stale prompt045 binary**, because prompt045 did not change `InstallationV0BuildInfo` and intentionally did not require a diagnostics label update unless diagnostics source changed.
- ProductRoot is present and approved.
- Environment is approved.
- Diagnostics mode is not latched.
- Cached assessment is not reused.
- The only failing predicate is `DatabaseApproved=False`.
- The UI independently shows the intended target database name `obm_pos_dev_v0_pg`.

Do not weaken the protected-database Development guard. Do not allow arbitrary database names. Do not mutate PostgreSQL. Do not set User/Machine environment variables.

## Objective

Correct the database-name resolution/normalization/source-selection bug so both supported routes accept the canonical Development database:

```text
obm_pos_dev_v0_pg
```

Routes:

```text
A. OBM-POS Runtime Development profile
   -> approved ProductRoot
   -> approved database obm_pos_dev_v0_pg
   -> local DB readiness
   -> MainWindow

B. InstallationV0 diagnostics -> Open OBM-POS
   -> verified ProductRoot handoff
   -> same canonical runtime bootstrap database
   -> local DB readiness
   -> exactly one MainWindow
```

The guard must continue rejecting:

```text
production/Royal/protected databases
legacy source databases
empty/unknown database names
ProgramData fallback in ambiguous Development
another unapproved Development lane
```

## First task — identify the exact database values and sources

Audit every database name involved in the failing call path:

```text
DevelopmentProfileLaunchPolicy.EvaluateStartupGuard
App.xaml.cs pre-configuration guard
App.xaml.cs post-configuration guard
RuntimeBootstrapLocator
runtime bootstrap metadata
RuntimeProfileStartupAssessmentService
connection-string parsing fallback
InstallationV0 service Target DB display
launchSettings.json Development profile
any approved database constants or allowlists
```

Report sanitized evidence for each value:

```text
RawDatabaseNameSource
RawDatabaseNameLength
NormalizedDatabaseName
NormalizedDatabaseNameLength
ApprovedDatabaseName
ApprovedDatabaseNameLength
OrdinalIgnoreCaseEqual
ContainsLeadingOrTrailingWhitespace
ContainsQuotes
ContainsSemicolonOrConnectionStringFragment
ContainsInvisibleUnicode
SourceClassification
```

Do not print the full connection string or database password.

The report must identify the exact value that made `DatabaseApproved=False` and why it differed from the UI-visible `obm_pos_dev_v0_pg`.

## Likely fault classes — verify, do not assume

Investigate all of these:

1. Guard reads a stale/default connection-string database while diagnostics UI reads runtime bootstrap metadata.
2. Guard receives a full connection string or quoted value instead of only the database name.
3. Leading/trailing whitespace or quote characters prevent equality.
4. An approved database constant still points to an older Development lane.
5. Prompt045 refactoring changed database source precedence and reintroduced the pre-prompt043 fault.
6. Pre-configuration guard runs before bootstrap metadata is available and incorrectly returns a final rejection instead of deferring to post-configuration validation.
7. Installation handoff preserves ProductRoot but does not rebuild the database-source context.
8. Two different `DevelopmentProfileLaunchPolicy` overloads or call sites use inconsistent normalization.

## Canonical database resolver

Use one authoritative resolver for Development database approval.

Required precedence after an approved ProductRoot is known:

```text
1. Runtime bootstrap metadata under the approved ProductRoot.
2. Explicit canonical database field from resolved local DB configuration.
3. Parsed connection-string database only as a controlled fallback.
4. Missing/rejected.
```

Normalization must be minimal and deterministic:

```text
trim surrounding whitespace
remove only legitimate surrounding quotes introduced by configuration serialization
compare database identifiers using OrdinalIgnoreCase
never accept a full connection string as a database name
never use substring/Contains matching
```

Canonical approved value:

```text
obm_pos_dev_v0_pg
```

Do not broaden approval to wildcard names.

## Pre-configuration versus post-configuration guard

If the pre-configuration stage cannot yet resolve the database name safely:

```text
- it may validate ProductRoot and reject known forbidden lanes;
- it must not falsely reject the canonical lane solely because database metadata is not loaded yet;
- final database approval must occur after runtime bootstrap/config is resolved.
```

Use a precise state such as:

```text
DATABASE_APPROVAL_DEFERRED_UNTIL_RUNTIME_BOOTSTRAP
```

rather than `DEVELOPMENT_DATABASE_NOT_APPROVED` when the value is genuinely unavailable at the early stage.

The post-configuration guard must then fail closed if the resolved database is missing or not exactly approved.

## Local-DB-first contract must remain intact

Do not reintroduce any prompt045-removed normal-runtime gates:

```text
Phase1 checkpoint
WpfJwt validity
API reachability
Phase2 exact marker/count evidence
employee count
permission count
outbox count
launch provenance as runtime authorization
employee operational PIN configuration
```

After database approval, use the prompt045 local readiness predicate:

```text
PostgreSQL auth succeeds
schema ready
singleton runtime profile
RuntimeState Activated
Tenant/POS/profile identity consistent
-> MainWindow
```

## Build label and diagnostics

Because this prompt changes the physical guard/diagnostic path, update:

```text
Build label: prompt047
Window title: OBM InstallationV0 Phase 1/2 - prompt047
```

Add safe diagnostics for the database resolver without printing connection strings or credentials:

```text
DatabaseNameSource=<RuntimeBootstrap|ResolvedConfig|ConnectionStringFallback|Missing>
ResolvedDatabaseName=obm_pos_dev_v0_pg
ApprovedDatabaseName=obm_pos_dev_v0_pg
DatabaseApproved=True|False
DatabaseApprovalStage=<PreConfigurationDeferred|PostConfigurationFinal>
```

## Tests

Add focused tests for at least:

```text
canonical runtime bootstrap DB -> approved
canonical resolved config DB -> approved
canonical connection-string fallback DB -> approved
case difference only -> approved
surrounding whitespace -> normalized and approved
legitimate surrounding quotes -> normalized and approved
full connection string passed as database name -> rejected
empty/missing pre-configuration DB -> deferred, not final false rejection
empty/missing post-configuration DB -> rejected
protected/Royal/production DB -> rejected
legacy source DB -> rejected
unknown Development DB -> rejected
same-process InstallationV0 handoff -> canonical DB source rebuilt -> approved
Runtime Development profile -> canonical DB -> MainWindow route
prompt045 local-DB-first rules remain in force
```

Run:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Startup|FullyQualifiedName~Bootstrap|FullyQualifiedName~DevelopmentProfile"
```

## Database safety

Database inspection must be read-only:

```text
BEGIN TRANSACTION READ ONLY
...
ROLLBACK
```

Do not:

```text
change database name
change runtime bootstrap
change username/password
change roles/GRANTs
run migration
run seed
rewrite marker/outbox/runtime state
```

## Physical retest target

Route A:

```text
OBM-POS Runtime Development
-> no InstallationV0Window
-> no DEVELOPMENT_DATABASE_NOT_APPROVED
-> MainWindow opens
```

Route B:

```text
InstallationV0 prompt047
-> Phase2 v002 Complete
-> Open OBM-POS
-> DatabaseNameSource=RuntimeBootstrap or ResolvedConfig
-> ResolvedDatabaseName=obm_pos_dev_v0_pg
-> DatabaseApproved=True
-> one MainWindow opens
-> diagnostics closes
```

## Do not execute prompt046 yet

`prompt/prompt046.md` is reserved for later legacy ASP.NET Identity/source-trace cleanup only after physical WPF MainWindow/local CRUD PASS.

This prompt047 is the immediate blocker correction and must not drop tables or remove Identity artifacts.

## Report 047

Create and push:

```text
report/report047.md
```

Required sections:

1. Verdict.
2. Physical prompt045 failure evidence.
3. Why prompt043 build label did or did not indicate stale binary.
4. Exact rejected database value/source.
5. Exact root cause of `DatabaseApproved=False`.
6. Pre-change database-source precedence.
7. Corrected canonical resolver and normalization.
8. Pre-configuration deferred versus post-configuration final behavior.
9. Protected database guard preservation.
10. Prompt045 local-DB-first contract preservation.
11. Same-process Open OBM-POS behavior.
12. Exact source files changed.
13. Build/test commands and counts.
14. Read-only DB evidence.
15. Operator physical retest steps.
16. Deferred prompt046 cleanup note.
17. No secrets/no DB mutation/no source push proof.
18. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_CANONICAL_DEVELOPMENT_DATABASE_RESOLVER_READY_FOR_USER_RETEST
```

```text
BLOCKED_OBM_POS_DEVELOPMENT_DATABASE_RESOLVER_INCOMPLETE
```
