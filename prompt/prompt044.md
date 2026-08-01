# Prompt 044 — Audit and simplify OBM-POS runtime security to local DB + standard API token pair

## Operator decision

The current InstallationV0/Development startup path has accumulated too many launch-provenance, ProductRoot, environment, and guard conditions. The operator believes this has become over-engineered and may be conflating installation safety with normal POS authentication.

The operator defines only two canonical runtime connections:

```text
1. WPF -> local PostgreSQL
   Standard PostgreSQL connection using host/port/database/user/password.
   The runtime database user has the required CRUD privileges.
   No hidden database security flags, launch provenance, environment provenance,
   ProductRoot provenance, API token, or pairing state is required to authorize local CRUD.

2. WPF -> API
   Standard access token + refresh token.
   Tokens are stored securely on the machine.
   If API is unavailable, OBM-POS still opens and works against the local DB.
   Cloud/sync operations become offline and resume later.
```

This prompt is an **audit and simplification design prompt only**. Do not change source or mutate any database. Produce a precise report that separates what is truly required from what should be removed or scoped to Development diagnostics only.

## Read first

```text
report/report037.md
report/report038.md
report/report039.md
report/report040.md
report/report041.md
report/report042.md
report/report043.md
E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md
```

Inspect the relevant OBM source under:

```text
E:\Project2026\4POS\NailSalonNet8
E:\Project2026\1ApiServer\ApiServer01
```

Do not print passwords, connection strings, tokens, refresh tokens, Pairing Codes, private GUIDs, or protected credential content.

## Audit objective

Map the current physical runtime path from process start to `MainWindow`, local DB access, and API access.

Classify every gate/configuration item into exactly one category:

```text
A. Required for local PostgreSQL connectivity
B. Required for API authentication/sync
C. Installation-only state/proof
D. Development-only safety guard
E. Redundant/over-engineered normal-runtime dependency
F. Unknown — requires further proof
```

The audit must determine whether current startup has incorrectly made any category C/D/E item mandatory for normal local POS operation.

## Canonical target architecture

### A. Local PostgreSQL runtime

Normal OBM-POS startup should require only:

```text
- a resolved local database configuration;
- standard Npgsql connection fields:
  Host
  Port
  Database
  Username
  Password
- successful connection using that runtime role;
- required schema/migrations present;
- the local installation identity/state is internally consistent;
- runtime state is usable, normally Activated.
```

The runtime DB role must have the explicit privileges needed by the app, such as:

```text
CONNECT
USAGE on the app schema
SELECT / INSERT / UPDATE / DELETE on required tables
required sequence privileges
```

Clarify the exact current runtime DB role and credential source without printing secrets.

An installer/admin credential may be used temporarily for database creation, migration, backup, or GRANT operations, but it must not become a hidden ongoing runtime dependency.

The baseline seed transaction must not:

```text
- create or alter PostgreSQL users/roles;
- store DB passwords in business tables;
- store API tokens in PostgreSQL;
- set User/Machine environment variables;
- require API connectivity;
- encode launch-profile provenance as database authorization;
- add hidden security flags that can later block ordinary CRUD.
```

`TblTenant`, `TblPosLocal`, `TblPosRuntimeProfile`, state history, and completion markers are installation identity/state records. They are not substitutes for PostgreSQL authentication.

### B. API runtime

Determine the current physical API credential model after Pairing Code redemption:

```text
- bootstrap WpfJwt only?
- access token?
- refresh token?
- device credential?
- token expiration and renewal path?
- protected storage path?
- startup dependency on API availability?
```

The canonical target model is:

```text
one short-lived access token
one long-lived refresh token
```

Storage requirements:

```text
- protected locally with the existing approved Windows protection mechanism;
- never stored in PostgreSQL business/config tables;
- never stored in plaintext logs or reports;
- Pairing Code never stored after redemption;
- refresh token rotation/revocation handled by the API contract.
```

Runtime behavior:

```text
access token valid
-> use API normally

access token expired + API reachable + refresh token valid
-> refresh and continue

API unavailable / timeout / DNS / server outage
-> open MainWindow and operate locally
-> mark API/sync offline
-> keep local outbox pending
-> retry later

refresh token invalid/revoked
-> local POS still opens
-> cloud/sync features are blocked or require reauthorization
-> local checkout/business data remains usable
```

The bootstrap token from Phase 1 must not be treated as the permanent daily runtime credential unless the API contract explicitly proves that it already implements access/refresh semantics.

## Offline-first rule

The following must be audited and proposed as a hard runtime rule:

```text
Local DB healthy + schema ready + local runtime state usable
=> MainWindow may open even when API is unreachable.
```

API unavailability must not cause:

```text
- InstallationV0Window to reopen;
- Pairing Code to be requested again;
- local DB access to be rejected;
- RuntimeState to be changed to RecoveryRequired solely due to network outage;
- local seed/migration to rerun;
- employee/checkout/settings data to become inaccessible.
```

Cloud-only functions may show offline status and remain disabled until connectivity returns.

## ProductRoot and environment policy

Audit these current components and all callers:

```text
SPACEPOS_PRODUCT_ROOT
SPACEPOS_INSTALLATION_MODULE
EffectiveProductRootContext
LaunchProvenanceContext
DevelopmentProfileLaunchPolicy
DevelopmentStartupGuard
SpacePosConfigurationPaths.OverrideProductRoot
runtime bootstrap metadata
launchSettings.json profiles
```

Target interpretation:

```text
ProductRoot = filesystem location for local machine state/config/secrets/checkpoints.
ProductRoot is not a database password, API token, or runtime authorization credential.
```

A strict Development profile may protect developers from accidentally opening a production database, but such protection must be scoped to Development/debug launch safety and must not become part of production runtime authentication.

Identify exactly which current gates can be:

```text
- removed from normal installed runtime;
- retained only in InstallationV0 diagnostics;
- retained only in Development/debug profile validation;
- simplified to one resolved ProductRoot path;
- replaced by direct local DB readiness and token-state checks.
```

Do not recommend removing all safety checks blindly. Recommend the smallest set that prevents destructive developer mistakes without blocking a correctly installed POS.

## Startup decision target

Propose a minimal decision tree:

```text
Start OBM-POS
    |
    +-- local DB config missing / DB unreachable / schema not ready
    |      -> installation/recovery UI with exact local DB reason
    |
    +-- local DB healthy and installed state internally consistent
           -> open MainWindow
           -> initialize API client independently
                 |
                 +-- access/refresh succeeds -> Online
                 +-- API unavailable -> Offline Local Mode
                 +-- refresh invalid -> Offline / Reauthorization Required
```

Phase 1/Phase 2 markers may prove installation completion, but API availability and launch-profile provenance must not be required for every normal local startup.

## Exact source audit requirements

Inspect and report exact files/methods for:

### Local database

```text
connection string/config locator
password/username retrieval
Npgsql connection creation
runtime DB role validation
startup DB health/schema checks
migration/provisioning credentials
any DB privilege/GRANT logic
```

### API authentication

```text
Pairing Code redeem response contract
WpfJwt creation and claims
credential expiration
protected credential write/read
access token storage
refresh token storage
refresh endpoint/client code
401 handling
API outage handling
outbox retry/sync behavior
```

### Startup gates

```text
all reasons MainWindow can be blocked
all reasons InstallationV0Window opens
all direct reads of ProductRoot/provenance environment state
whether API reachability is currently a startup prerequisite
whether token expiry is currently a startup prerequisite
```

## Required report conclusions

The report must answer explicitly:

1. Does WPF currently connect successfully to `obm_pos_dev_v0_pg` with a standard PostgreSQL runtime username/password?
2. What exact runtime DB privileges are currently required and granted?
3. Is any extra application-defined security condition currently blocking DB CRUD after connection succeeds?
4. Does the Phase 2 seed currently alter security/roles/environment/tokens?
5. Does the API currently issue a true access-token + refresh-token pair?
6. If not, what exactly is currently issued and how does it expire?
7. Can current WPF open and operate locally when API is offline?
8. Which existing startup guards are required, Development-only, installation-only, or redundant?
9. What is the smallest safe removal/simplification plan?
10. What exact implementation sequence should follow in prompt045 or later?

## Proposed simplification plan format

Provide a table:

| Current mechanism | Current purpose | Keep / Scope / Remove | Replacement |
| --- | --- | --- | --- |

Then provide a phased implementation plan:

```text
S1 — Local DB startup independent of API
S2 — Standard access/refresh token contract
S3 — Offline local mode and API reconnect
S4 — Remove/scoped legacy startup provenance guards
S5 — Clean-install regression from empty DB to MainWindow
```

Each phase must identify rollback risk, tests, and whether it changes DB schema/API contracts.

## Safety and mutation policy

This prompt must not:

```text
- modify OBM source;
- modify PostgreSQL;
- change roles or passwords;
- create or rotate tokens;
- change environment variables;
- start WPF;
- stop running services;
- commit/push OBM source.
```

Read-only source and database inspection only.

## Verification

Run only read-only checks and non-mutating source inspection. Builds/tests are not required because no source changes are allowed.

If a command could write to PostgreSQL, do not run it.

## Report 044

Create and push:

```text
report/report044.md
```

Required sections:

1. Verdict.
2. Executive conclusion: over-engineered or not, with specific evidence.
3. Current local PostgreSQL connection/authentication path.
4. Current runtime DB role and privileges.
5. Current Phase 2 seed security side effects — prove yes/no.
6. Current API credential contract.
7. Access token and refresh token gap analysis.
8. Current API-offline startup/runtime behavior.
9. Startup gate inventory and classification A-F.
10. ProductRoot/environment/provenance simplification analysis.
11. Canonical minimal startup decision tree.
12. Mechanism keep/scope/remove table.
13. Phased implementation plan S1-S5.
14. Exact source files/methods inspected.
15. Read-only DB evidence.
16. No mutation/no secrets/no source push proof.
17. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_RUNTIME_SECURITY_SIMPLIFICATION_AUDIT_COMPLETE
```

```text
BLOCKED_OBM_POS_RUNTIME_SECURITY_AUDIT_INCOMPLETE
```
