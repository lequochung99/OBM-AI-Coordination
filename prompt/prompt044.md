# Prompt 044 — Re-audit and rewrite OBM-POS runtime connectivity, API-token, and operational-PIN documentation

## Operator correction — authoritative terminology

The previous runtime-security documentation used the word **password** too broadly.

The operator confirms that OBM-POS has **no application-user password model in the ordinary account-authentication sense**.

OBM-POS employee access uses a simple local PIN, physically represented in current/legacy code by fields and concepts such as:

```text
TblEmployee.LoginNumber
Employee PIN
Manager/Owner/Admin PIN
```

The PIN is intended primarily to:

```text
- discourage casual access to manager/owner functions;
- distinguish Staff from manager/non-Staff actions;
- identify which employee performed an action;
- support audit/log attribution.
```

It is **not** intended to be described as:

```text
- a full application account password;
- a PostgreSQL credential;
- an API credential;
- a cryptographic device identity;
- proof that the current machine is authorized by Platform;
- a replacement for access/refresh tokens.
```

Approved product direction to verify against current source:

```text
Staff PIN: 4 digits
Non-Staff PIN (Owner/Admin/SubAdmin/Manager-style): 6 digits
```

If current source still implements another non-Staff length, report the exact source behavior as an implementation gap; do not silently rewrite evidence.

A separate PIN/lock for Advanced Settings or terminal configuration, where present, is also a simple local operational guard, not a strong application password system.

The PlatformApp/PlatformAdmin side must not manage salon employee PINs. Employee PIN handling belongs to WPF/POS.

Do not print raw PIN values in the report. This is an operational privacy rule, not evidence that the PIN is a strong password.

## Three credential domains — keep them separate

The audit and rewritten documentation must separate exactly these three domains:

### Domain 1 — WPF to local PostgreSQL

This uses a genuine infrastructure credential:

```text
Host
Port
Database
Username
Password
```

The PostgreSQL username/password belongs to the database connection and role model. It is not an employee login password.

### Domain 2 — WPF to API

This should use genuine service credentials:

```text
short-lived access token
long-lived refresh token
```

Tokens belong to device/API authentication and sync. They are not employee PINs and must not be stored in PostgreSQL business tables.

### Domain 3 — employee operational PIN

This is a local POS convenience/control mechanism:

```text
role-sensitive UI gating
manager-area access control
actor attribution
audit/log correlation
```

It must not be documented as the core authentication mechanism for the database, API, device, tenant, or installation.

## Operator decision on runtime simplicity

The current InstallationV0/Development startup path has accumulated too many launch-provenance, ProductRoot, environment, and guard conditions. The operator believes this has become over-engineered and may be conflating installation safety with normal POS operation.

There are only two canonical runtime connections:

```text
1. WPF -> local PostgreSQL
   Standard PostgreSQL connection using host/port/database/user/password.
   The runtime database user has the required CRUD privileges.
   No hidden database security flags, launch provenance, environment provenance,
   ProductRoot provenance, API token, employee PIN, or pairing state is required
   to authorize ordinary local CRUD after PostgreSQL authentication succeeds.

2. WPF -> API
   Standard access token + refresh token.
   Tokens are stored securely on the machine.
   If API is unavailable, OBM-POS still opens and works against the local DB.
   Cloud/sync operations become offline and resume later.
```

This prompt is an **audit and documentation rewrite only**.

Do not change source or mutate any database.

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

Inspect relevant source under:

```text
E:\Project2026\4POS\NailSalonNet8
E:\Project2026\1ApiServer\ApiServer01
```

Do not print passwords, connection strings, access tokens, refresh tokens, Pairing Codes, protected credential contents, raw PIN values, or private identity values.

## Audit objective

Map the current physical runtime path from process start to:

```text
MainWindow
local PostgreSQL CRUD
API access/sync
employee PIN validation and audit attribution
```

Classify every mechanism into exactly one category:

```text
A. Required for local PostgreSQL connectivity
B. Required for API authentication/sync
C. Employee operational PIN / local UI and audit behavior
D. Installation-only state/proof
E. Development-only safety guard
F. Redundant/over-engineered normal-runtime dependency
G. Unknown — requires further proof
```

Determine whether current startup has incorrectly made any category C/D/E/F mechanism mandatory for ordinary local POS operation.

## Part 1 — employee PIN semantics audit

Inspect the exact current implementation of employee PIN behavior, including as applicable:

```text
TblEmployee.LoginNumber
employee PIN dialogs
manager/owner/admin PIN dialogs
PIN validation services
role/permission lookup
Staff versus non-Staff filtering
manager-area navigation gates
sensitive action gates
actor/audit logging
PIN reset/change UI
Advanced Settings/terminal setup PIN behavior
```

Answer explicitly:

1. Does the app have any real application-user password/account-authentication system?
2. Is `LoginNumber` treated as a PIN, password, employee identifier, or mixed concept in source?
3. What are the physically enforced current PIN lengths for Staff and non-Staff?
4. Which screens/actions require any PIN?
5. Which checks are only for audit attribution?
6. Which checks also enforce role level, such as Staff versus manager/non-Staff?
7. Does the app hash PINs, store them directly, or use another representation?
8. Is uniqueness enforced per tenant, and where?
9. Does PlatformApp or ApiServer manage employee PINs anywhere?
10. Does InstallationV0 seed copy, reset, omit, or generate employee `LoginNumber`/PIN values?

Do not assume the answer from names alone. Trace readers, writers, validators, and UI behavior.

### PIN documentation rule

Rewrite terminology so documentation uses:

```text
employee operational PIN
manager/non-Staff PIN
local manager-area gate
actor attribution / audit PIN
```

Do not use:

```text
employee password
manager password
application account password
```

unless a specific source component truly implements a separate password system; if so, identify it precisely.

### PIN seed-policy audit

Determine the safest minimal seed behavior from current source and operator intent.

The report must distinguish:

```text
- copying real reference PINs;
- deterministic placeholder/reset values;
- null/unconfigured values;
- user edits after installation;
- uniqueness requirements;
- UI behavior when PIN is unconfigured.
```

Do not propose a new password framework, mandatory cryptographic identity system, or Platform-managed salon PIN service.

Do not silently recommend hashing/auth frameworks merely because the field is called `LoginNumber`; first prove whether stronger security is a product requirement. The current operator intent is a simple local gate plus audit attribution.

## Part 2 — canonical local PostgreSQL runtime

Normal OBM-POS startup should require only:

```text
- resolved local database configuration;
- standard Npgsql connection fields:
  Host
  Port
  Database
  Username
  Password
- successful PostgreSQL authentication;
- required CRUD/schema privileges;
- expected schema/migrations present;
- local installation identity/state internally consistent;
- runtime state usable, normally Activated.
```

The PostgreSQL runtime role should have only the privileges physically required by the app, such as:

```text
CONNECT
USAGE on the application schema
SELECT / INSERT / UPDATE / DELETE on required tables
required sequence privileges
```

Clarify the exact current runtime DB role and credential source without printing secrets.

An installer/admin credential may be used temporarily for:

```text
database creation
migration
backup/restore
GRANT operations
```

but it must not become a hidden ongoing runtime dependency.

### Seed security boundary

The baseline seed transaction must not:

```text
- create or alter PostgreSQL users/roles;
- GRANT or REVOKE runtime privileges;
- store DB usernames/passwords in business tables;
- store API access/refresh tokens in PostgreSQL;
- set User/Machine environment variables;
- require API connectivity;
- use employee PIN as database authorization;
- encode launch-profile provenance as database authorization;
- add hidden security flags that later block ordinary CRUD.
```

`TblTenant`, `TblPosLocal`, `TblPosRuntimeProfile`, runtime-state history, and completion markers are installation identity/state records. They are not substitutes for PostgreSQL authentication.

Employee PIN rows/fields are business/operational configuration. They are not PostgreSQL security configuration.

## Part 3 — API credential model

Determine the current physical API credential model after Pairing Code redemption:

```text
bootstrap WpfJwt only?
short-lived access token?
long-lived refresh token?
device credential?
expiration and renewal path?
protected storage path?
401 handling?
startup dependency on API availability?
```

The canonical target model is:

```text
one short-lived access token
one long-lived refresh token
```

Storage requirements:

```text
- protected locally using the approved Windows protection mechanism;
- never stored in PostgreSQL business/config tables;
- never printed in logs/reports;
- Pairing Code never stored after redemption;
- refresh-token rotation/revocation handled by API contract.
```

Runtime behavior:

```text
access token valid
-> API online

access token expired + API reachable + refresh token valid
-> refresh and continue

API unavailable / timeout / DNS / server outage
-> MainWindow still opens
-> local PostgreSQL work continues
-> API/sync status becomes Offline
-> local outbox remains pending
-> reconnect/retry later

refresh token invalid/revoked
-> MainWindow still opens locally
-> cloud/sync functions require reauthorization
-> local checkout/business data remains usable
```

An employee PIN must never be reused as an API token, refresh token, device credential, or API login secret.

## Offline-first rule

Audit and document this as the intended hard runtime rule:

```text
Local DB healthy + schema ready + local runtime state usable
=> MainWindow may open even when API is unreachable.
```

API unavailability must not cause:

```text
InstallationV0Window to reopen
Pairing Code to be requested again
local DB CRUD to be rejected
RuntimeState to become RecoveryRequired solely due to network outage
local seed/migration to rerun
employee/checkout/settings data to become inaccessible
employee PIN validation to depend on API availability
```

Cloud-only functions may show offline status and remain disabled until connectivity returns.

## ProductRoot and Development policy

Audit these components and all callers:

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
ProductRoot = filesystem location for machine state/config/checkpoints/protected API credentials/logs.
```

ProductRoot is not:

```text
a PostgreSQL password
an employee PIN
an API access/refresh token
a daily runtime authorization credential
```

A strict Development profile may protect developers from accidentally opening a production database, but it must remain Development/debug launch safety and must not become part of production runtime authentication.

Identify exactly which current gates should be:

```text
removed from normal installed runtime
retained only in InstallationV0 diagnostics
retained only in Development/debug profile validation
simplified to one resolved ProductRoot path
replaced by direct local DB readiness and independent API token-state checks
```

Do not recommend removing all safety checks blindly. Recommend the smallest set that prevents destructive developer mistakes without blocking a correctly installed POS.

## Minimal startup decision tree

Propose this minimal decision tree and compare it with physical source:

```text
Start OBM-POS
    |
    +-- local DB config missing / DB unreachable / schema not ready
    |      -> installation/recovery UI with exact local DB reason
    |
    +-- local DB healthy and installed state internally consistent
           -> open MainWindow
           -> initialize employee PIN/role services locally
           -> initialize API client independently
                 |
                 +-- access/refresh succeeds -> Online
                 +-- API unavailable -> Offline Local Mode
                 +-- refresh invalid -> Offline / Reauthorization Required
```

Phase 1/Phase 2 markers may prove installation completion, but API availability, employee PIN state, and launch-profile provenance must not be required for every ordinary local startup unless current product behavior explicitly requires a specific local PIN configuration.

## Exact source audit requirements

### Employee PIN

Inspect exact files/methods for:

```text
LoginNumber model/schema
PIN length validation
PIN uniqueness
PIN lookup and duplicate handling
role-level checks
manager-area navigation
sensitive action authorization
actor/audit log writes
PIN reset/change behavior
seed transform behavior
```

### Local database

Inspect exact files/methods for:

```text
connection string/config locator
username/password retrieval
Npgsql connection creation
runtime DB role validation
startup DB health/schema checks
migration/provisioning credentials
DB privilege/GRANT logic
```

### API authentication

Inspect exact files/methods for:

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

Inspect:

```text
all reasons MainWindow can be blocked
all reasons InstallationV0Window opens
all direct reads of ProductRoot/provenance state
whether API reachability is a startup prerequisite
whether token expiry is a startup prerequisite
whether employee PIN configuration is a startup prerequisite
```

## Required report conclusions

The report must answer explicitly:

1. Does OBM-POS currently implement any real application-user password system?
2. What is the exact physical employee PIN model and purpose?
3. What PIN lengths are implemented versus approved policy (Staff 4 / non-Staff 6)?
4. What does InstallationV0 currently do with employee PIN/LoginNumber values?
5. Does WPF connect to `obm_pos_dev_v0_pg` using a standard PostgreSQL username/password?
6. What runtime DB privileges are required/granted?
7. Is any extra application-defined gate currently blocking local CRUD after DB authentication succeeds?
8. Does Phase 2 seed alter roles, DB credentials, API tokens, environment security, or employee PIN security semantics?
9. Does the API issue a true access-token + refresh-token pair?
10. Can WPF open and operate locally when API is offline?
11. Which startup guards are required, Development-only, installation-only, or redundant?
12. What is the smallest safe simplification plan?
13. Which prior documents/phrases must be corrected because they call employee PIN a password or security credential?

## Documentation rewrite requirements

Report044 must contain a clean replacement documentation section titled:

```text
Canonical OBM-POS Runtime Connectivity and Operational PIN Model
```

That section must be usable as the corrected project reference and must clearly document:

```text
- PostgreSQL infrastructure credential;
- API access/refresh token pair;
- employee operational PIN semantics;
- offline-first behavior;
- installation state versus runtime authentication;
- Development-only safety guards;
- seed boundaries;
- terminology to avoid.
```

Include a terminology correction table:

| Incorrect or ambiguous term | Correct term | Meaning |
| --- | --- | --- |
| employee password | employee operational PIN | local role gate + actor attribution |
| manager password | manager/non-Staff PIN | manager-area/sensitive-action gate + audit |
| database PIN | PostgreSQL username/password | infrastructure authentication |
| WPF password | API access/refresh token or DB credential, depending on context | must be named precisely |

Also list exact prior source/docs/report phrases that should be revised later. Do not modify those documents in this audit prompt.

## Classification table

Provide:

| Current mechanism | Domain | Current purpose | Keep / Scope / Remove | Replacement |
| --- | --- | --- | --- | --- |

Domains must use:

```text
PostgreSQL
API token
Employee PIN
Installation state
Development safety
Redundant runtime gate
```

## Phased implementation plan

Provide a minimal phased plan:

```text
S1 — Local DB startup independent of API and PIN provenance
S2 — Standard access/refresh token contract
S3 — Offline local mode and API reconnect
S4 — Scope/remove legacy startup provenance guards
S5 — Normalize PIN terminology and approved 4/6-digit policy without creating a password system
S6 — Clean-install regression from empty DB to MainWindow and offline operation
```

Each phase must identify:

```text
source files
rollback risk
tests
DB schema impact
API contract impact
whether PIN behavior changes or documentation only
```

## Safety and mutation policy

This prompt must not:

```text
modify OBM source
modify PostgreSQL
change roles/passwords
change employee PINs
create/rotate tokens
change environment variables
start WPF
stop services
commit/push OBM source
```

Read-only source and DB inspection only.

## Verification

Run only read-only checks and non-mutating source inspection.

Builds/tests are not required because no source changes are allowed.

If a command could write PostgreSQL, do not run it.

## Report 044

Create and push:

```text
report/report044.md
```

Required sections:

1. Verdict.
2. Executive conclusion: over-engineered or not, with exact evidence.
3. Canonical three-domain distinction: PostgreSQL credential / API token pair / employee operational PIN.
4. Current employee PIN physical implementation and purpose.
5. Current PIN lengths versus approved 4/6-digit policy.
6. Current PIN seed/reset/edit behavior.
7. Current local PostgreSQL connection/authentication path.
8. Current runtime DB role and privileges.
9. Current Phase 2 seed side effects — prove yes/no for roles, credentials, tokens, environment, PIN semantics.
10. Current API credential contract.
11. Access-token/refresh-token gap analysis.
12. Current API-offline startup/runtime behavior.
13. Startup gate inventory and classification A-G.
14. ProductRoot/environment/provenance simplification analysis.
15. Canonical minimal startup decision tree.
16. Mechanism keep/scope/remove table.
17. `Canonical OBM-POS Runtime Connectivity and Operational PIN Model` rewritten documentation.
18. Terminology correction table and prior-doc correction list.
19. Phased implementation plan S1-S6.
20. Exact source files/methods inspected.
21. Read-only DB evidence.
22. No mutation/no raw PIN/no secrets/no source push proof.
23. Coordination commit SHA.

## Valid verdicts

```text
OBM_POS_RUNTIME_CONNECTIVITY_AND_PIN_MODEL_AUDIT_COMPLETE
```

```text
BLOCKED_OBM_POS_RUNTIME_CONNECTIVITY_AND_PIN_AUDIT_INCOMPLETE
```
