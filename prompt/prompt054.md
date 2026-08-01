# Prompt 054 — Graphify and source-proof audit of redundant InstallationV0/runtime links before canonical documentation

## Operator decision

Prompt053 correctly stopped because the intended canonical-document folder does not yet exist. Before creating the first canonical runtime document and before performing more naming/deletion work, perform one read-only dependency audit of the current WPF installation/startup implementation.

The operator has Graphify installed on this PC and wants it used when available to expose multi-hop relationships that ordinary text search may miss.

This prompt is **audit only**.

Do not modify implementation source, documentation, project files, launch profiles, tests, PostgreSQL data/schema, API state, environment variables, or running processes.

The purpose is to answer:

```text
Does the InstallationV0/runtime flow still contain redundant, duplicated, misleading,
or cross-domain links that should be removed before the canonical documentation is created?
```

## Read first

Read completely:

```text
report/report044.md
report/report045.md
report/report049.md
report/report050.md
report/report051.md
report/report052.md
report/report053.md
```

Inspect the current local OBM source tree, including all uncommitted prompt052 edits:

```text
E:\Project2026\4POS\NailSalonNet8
E:\Project2026\4POS\NailSalonNet8.Tests
```

Also inspect the current InstallationV0/API contracts needed to classify cross-domain dependencies:

```text
E:\Project2026\1ApiServer\ApiServer01
```

Do not print secrets, connection strings, PostgreSQL passwords, tokens, Pairing Codes, raw employee PINs, customer data, employee private data, or private identity values.

## Phase A — Graphify discovery and safe execution

First discover the installed Graphify capability without installing or upgrading anything.

Record sanitized evidence for:

```text
Get-Command graphify
where.exe graphify
graphify --version or equivalent supported version command
graphify --help
existing graphify-out folders
existing Graphify skill/MCP configuration for Codex
```

Do not assume command syntax. Use the locally installed version's help/skill instructions.

### If Graphify is available

Build or update a structural graph of:

```text
E:\Project2026\4POS\NailSalonNet8
```

Requirements:

- use the installed Graphify tool/skill in its supported local mode;
- use structural/static extraction; do not enable an external semantic-upload mode;
- do not install packages or update Graphify;
- do not commit Graphify output into the OBM source repository;
- do not overwrite a pre-existing graph without preserving it;
- record Graphify version and exact sanitized command/mode used;
- preserve the resulting graph artifacts in the versioned evidence folder defined below;
- preserve edge provenance such as extracted, inferred, or ambiguous when Graphify provides it.

Expected Graphify artifacts may include the installed version's equivalents of:

```text
graph.json
GRAPH_REPORT.md
graph.html
```

If Graphify writes a temporary `graphify-out` folder under the source repository, record whether it existed before the run. Copy new output to the versioned evidence folder. Remove only a newly generated temporary output after copying and hashing; never delete a pre-existing Graphify graph.

### If Graphify is unavailable or fails

Do not install it.

Record the exact safe reason and continue with the mandatory source-level fallback audit below.

Valid Graphify classifications:

```text
GRAPHIFY_AUDIT_PASS
GRAPHIFY_PRESENT_BUT_EXECUTION_BLOCKED
GRAPHIFY_NOT_DISCOVERED
```

Graphify is supporting evidence, not the sole deletion authority.

## Phase B — mandatory source-level dependency audit

Regardless of Graphify result, inspect source, DI, XAML, configuration, reflection, launch profiles, and tests.

Use safe tools such as:

```text
rg / source search
MSBuild project evaluation
C# symbol/reference inspection
Graphify queries when available
read-only PostgreSQL catalog queries when necessary
```

Do not modify source.

## Exact entrypoints and terminal nodes

Inventory exact files, types, and methods for these entrypoints:

```text
NailSalonNet8 App construction/startup
WPF OnStartup / startup event chain
OBM-POS Runtime Development launch profile
OBM-POS InstallationV0 Phase1 launch profile
InstallationV0Window creation
InstallationV0 Open OBM-POS click
normal MainWindow creation/show
post-MainWindow API/session initialization
installation/recovery UI routing
```

Terminal outcomes to map:

```text
MainWindow visible
InstallationV0 visible
Recovery UI visible
Application shutdown
Blocked/error result
Offline/Reauthorization API state after MainWindow
```

## Required graph questions

Use Graphify queries/path traversal when available and verify every important answer against source.

### 1. Direct runtime path

Find every path from:

```text
NailSalonNet8 process start
```

to:

```text
MainWindow.Show / MainWindow visible
```

Identify all intermediate services, result types, enums, callbacks, guards, configuration readers, and DI resolutions.

### 2. Installation handoff path

Find every path from:

```text
InstallationV0Window.Open OBM-POS
```

to:

```text
MainWindow.Show / MainWindow visible
```

Identify duplicate paths, adapters, compatibility shims, wrappers, callback conversions, and repeated readiness checks.

### 3. Local DB versus API cross-links

Find every edge from the local POS startup path to any of:

```text
WpfJwt
protected hello
Pairing Code
Phase1InstallationService
AppJwt / station access token
API availability
SignalR
sync workers
API bootstrap/session services
```

For each edge, classify:

```text
required before MainWindow
post-MainWindow only
installation-only
legacy/redundant
ambiguous/dynamic
```

The canonical target is:

```text
Local PostgreSQL usable -> MainWindow
API/session work -> after MainWindow
```

### 4. Installation proof links

Find every normal-runtime dependency on:

```text
Phase 1 checkpoint
Phase 2 marker
identity-spine audit
exact employee count
exact permission count
exact outbox count
InstallationV0 verification/readiness services
```

Classify whether each link is:

```text
normal-runtime required
installation-only
upgrade-only
recovery-only
legacy/redundant
```

### 5. Misleading names and compatibility paths

Map all definitions and references for current and legacy names, including:

```text
LocalPosStartupService
LocalPosStartupResult
LocalPosStartupDecision
ApplicationStartupCoordinator
DatabaseStartupAssessmentService
DatabaseStartupAssessment
DatabaseStartupMode
RuntimeProfileStartupAssessmentService
IApiSessionInitializer
ApiSessionInitializer
IAppJwtBootstrapper
AppJwtBootstrapper
InstallationV0VerificationService
InstallationV0CompletedReadinessService
BootstrapRepairRequired
POS_RUNTIME_ROUTE_BOOTSTRAPREPAIRREQUIRED
LaunchProvenanceContext
EffectiveProductRootContext
PosStartupRouteResult
```

For each symbol record:

```text
definition file
physical file name
active callers
DI registrations
factories
XAML/reflection/config references
tests-only references
historical-doc references
candidate final action
```

### 6. Duplicate decision ownership

Determine whether more than one component independently decides any of:

```text
local DB usable
installation required
recovery required
MainWindow may open
API online/offline
Phase 2 complete
Open OBM-POS enabled
```

Identify last-writer-wins UI assignments and duplicated status models.

### 7. Environment and launch-profile links

Map active reads/writes of:

```text
SPACEPOS_PRODUCT_ROOT
SPACEPOS_INSTALLATION_MODULE
DOTNET_ENVIRONMENT
ASPNETCORE_ENVIRONMENT
connection-string/database configuration sources
runtime PostgreSQL username selection
```

Classify each as:

```text
runtime configuration
Development-only safety
installation diagnostics selector
legacy/redundant
```

### 8. PostgreSQL role path

Prove, without printing secrets, which username normal WPF runtime configuration selects:

```text
hung
postgres
other
```

Map the exact configuration path from launch profile/bootstrap/config to Npgsql connection construction.

This is read-only. Do not change users, passwords, roles, GRANTs, or configuration.

### 9. ASP.NET Identity and password traces

As a secondary audit, map all active source/migration/DB references to:

```text
AspNetUsers
AspNetRoles
AspNetUserRoles
AspNetUserClaims
AspNetUserLogins
AspNetRoleClaims
AspNetUserTokens
Microsoft.AspNetCore.Identity
IdentityDbContext
IdentityUser
IdentityRole
UserManager
SignInManager
employee password
manager password
Login-With-Password
```

Do not drop tables or change source.

Classify each trace as:

```text
active WPF runtime
active API/Platform only
migration/history only
unused legacy
unknown
```

### 10. Dynamic-edge safety

Explicitly inspect patterns Graphify/static analysis may miss:

```text
reflection
Activator.CreateInstance
Type.GetType
assembly scanning
DI registration by convention
XAML type references
resource dictionaries
configuration-driven type names
JSON serialization type contracts
command-line switches
environment-variable routing
```

No delete recommendation may rely only on absence from Graphify.

## Read-only PostgreSQL catalog audit

When useful, query `obm_pos_dev_v0_pg` inside:

```text
BEGIN TRANSACTION READ ONLY;
...
ROLLBACK;
```

Only inspect sanitized metadata such as:

```text
table names
row counts for legacy Identity tables
foreign-key dependencies
views/triggers/functions referencing candidate tables
current_user classification
table privilege booleans
```

Do not print business rows, employee data, customer data, credentials, or connection strings.

## Versioned evidence folder

Create the next available versioned folder without overwriting prior evidence:

```text
E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\
```

If V001 exists, use V002, then V003, and so on.

Expected artifacts:

```text
README.md
SHA256SUMS.txt
graphify\...                         (when available)
installation-flow-current.mmd
installation-flow-target.mmd
installation-flow-subgraph.json
edge-inventory.csv
symbol-action-inventory.csv
redundant-link-candidates.md
dynamic-edge-checklist.md
aspnet-identity-trace-audit.md
```

Do not embed secrets or private row data.

## Machine-readable subgraph contract

Create `installation-flow-subgraph.json` with a stable structure similar to:

```json
{
  "version": "V001",
  "entrypoints": [],
  "nodes": [
    {
      "id": "...",
      "kind": "service|method|window|result|config|table|api",
      "name": "...",
      "file": "...",
      "line": 0,
      "domain": "local-db|api-session|installation|development-safety|ui|legacy",
      "active": true
    }
  ],
  "edges": [
    {
      "from": "...",
      "to": "...",
      "relation": "calls|resolves|reads|writes|routes|registers|inherits|references",
      "provenance": "extracted|inferred|ambiguous|source-verified",
      "normalRuntime": true,
      "classification": "keep|installation-only|api-post-mainwindow|development-only|delete-candidate|unknown",
      "evidence": ["file:line"]
    }
  ],
  "terminalOutcomes": [],
  "deleteCandidates": [],
  "unknowns": []
}
```

Use valid JSON. Do not include raw secrets or private identity values.

## Required diagrams

### Current physical graph

Create a Mermaid diagram that shows all currently active hops for:

```text
direct runtime -> MainWindow
InstallationV0 -> Open OBM-POS -> MainWindow
API/session path
installation verification path
```

Use dashed/red annotations in text labels for redundant or cross-domain links; do not rely on Mermaid color rendering alone.

### Target minimal graph

Create a Mermaid diagram showing only:

```text
LocalPosStartupService
-> local PostgreSQL usable?
-> MainWindow or Installation/Recovery

MainWindow
-> ApiSessionInitializer asynchronously

InstallationV0VerificationService
-> installation/audit only
```

## Proof standard for redundant-link candidates

Every delete/rename candidate must have:

```text
Graph path evidence when Graphify is available
source definition evidence
all-reference search evidence
DI/factory/XAML/reflection/config evidence
test-only versus runtime caller classification
risk if deleted
recommended action
```

Candidate actions:

```text
DELETE
RENAME
MERGE
KEEP_RUNTIME
KEEP_API_POST_MAINWINDOW
KEEP_INSTALLATION_ONLY
KEEP_DEVELOPMENT_ONLY
UNKNOWN_NEEDS_PHYSICAL_PROOF
```

Do not edit source in this prompt.

## Canonical-document foundation conclusion

The audit must conclude exactly what the first canonical documentation prompt should state and which old names/links it must prohibit.

Because the required canonical folder is currently absent, do not classify that absence as a permanent blocker. Instead, recommend creation of the **first** canonical version after this audit:

```text
INSTALLATION_RUNTIME_CANONICAL V001
```

There is no prior file at that path to preserve. Preserve existing older documents as `Evidence Only` or `Superseded`; do not pretend they are a prior version of a file that never existed.

Do not create the canonical docs in this audit prompt.

## Safety

This prompt must not:

```text
modify OBM source or tests
modify/create canonical documentation
rename/delete files
mutate PostgreSQL
run migration/seed
redeem Pairing Code
change tokens or PINs
set environment variables
launch or stop WPF/API services
install/update Graphify
commit/push OBM source
```

Read-only Graphify/source/DB inspection and versioned external evidence only.

## Report 054

Create and push:

```text
report/report054.md
```

Required sections:

1. Verdict.
2. Graphify discovery/version/execution classification.
3. Evidence folder path and artifact hashes.
4. Current direct-runtime call graph.
5. Current InstallationV0 handoff call graph.
6. Local-DB to API cross-link inventory.
7. Installation-proof link inventory.
8. Duplicate decision-owner inventory.
9. Old-name/symbol/file/DI inventory.
10. Environment/launch-profile dependency inventory.
11. Runtime PostgreSQL username resolution proof.
12. ASP.NET Identity/password trace audit.
13. Dynamic/reflection/XAML/config edge audit.
14. Redundant-link candidates with proof and risk.
15. Keep/installation-only/API-only/Development-only/delete table.
16. Current and target Mermaid diagrams.
17. Machine-readable subgraph artifact path/checksum.
18. Exact foundation requirements for canonical V001 documentation.
19. No source/docs/DB/process mutation proof.
20. Coordination commit SHA.

## Valid verdicts

```text
INSTALLATION_DEPENDENCY_GRAPH_AUDIT_READY_FOR_CANONICAL_V001
```

```text
BLOCKED_INSTALLATION_DEPENDENCY_GRAPH_AUDIT_INCOMPLETE
```
