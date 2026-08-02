# Prompt127 — Read-only connectivity, authentication, secret-source, and application-boundary inventory

## Purpose

Perform a complete **read-only audit and inventory** of the current OBM application connectivity and security boundaries.

This is a documentation task only.

```text
NO CODE CHANGES
NO SOURCE EDITS
NO CONFIG EDITS
NO DATABASE MUTATION
NO MIGRATION
NO SEED
NO BUILD OUTPUT CHANGES
NO SERVICE START/STOP
NO TOKEN REDEEM
NO PAIRING CODE CREATION
NO SECRET VALUE OUTPUT
```

The only permitted writes are the requested Markdown audit/report files under the versioned RecoveryReports artifact and `report/report127.md` in the coordination repository.

## Canonical authority

Read completely before auditing:

```text
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL_V004.md
```

If V004 is not present, stop with:

```text
BLOCKED_CANONICAL_V004_NOT_FOUND
```

V004 is the architectural target. The audit must distinguish:

```text
CURRENT_SOURCE_IMPLEMENTATION
CANONICAL_V004_TARGET
DIFFERENCE_OR_GAP
```

Do not silently reinterpret current source to match V004.

## Known project roots

Inspect these roots completely where relevant:

```text
WPF OBM-POS:
E:\Project2026\4POS\NailSalonNet8

ApiServer:
E:\Project2026\1ApiServer\ApiServer01

PlatformAppV0:
E:\Project2026\PlatformAppV0

BookingOnline / GuestWeb:
E:\Project2026\6Website
```

Locate and report the exact roots for:

```text
CompanionApp
BookingConsoleFlutterApp
```

If either root cannot be located, report the missing root explicitly. Do not guess.

## Read-only safety rules

Allowed:

```text
source-code search
configuration-file inspection
git status/log/diff inspection
project-reference inspection
package/dependency inspection
static call-chain tracing
read-only file metadata
read-only database schema inspection only when absolutely necessary
```

Forbidden:

```text
editing source/config/project files
formatting files
running migrations
creating or dropping databases
updating rows
starting or stopping applications/services
calling endpoints that create/change state
printing secret values
printing token values
printing full connection strings
printing Firebase service-account JSON
printing OAuth client secrets
printing passwords
```

Report secret/key **names only**, never values.

## Required master connection matrix

Create one master matrix with one row per proven application-to-application connection.

Required columns:

```text
Source application
Destination application/service
Direction
Direct or indirect
Transport/protocol
Endpoint/route/hub
Host/port source
TLS/loopback/public exposure
Authentication method
Credential/token type
Credential issuer
Credential storage owner
Refresh/renewal behavior
Identity fields/claims/headers
Configuration source
Retry/reconnect behavior
Offline/degraded behavior
Owning source class/method
Implementation status: Active / Partial / Legacy / Absent / Unproven
V004 compatibility
Risk/gap summary
```

Do not infer a direct connection merely because two apps share the same API.

## Audit area 1 — WPF OBM-POS to ApiServer

Trace all current WPF-to-API paths, including installation/bootstrap and normal runtime.

At minimum identify:

```text
API base URL owner and configuration source
HTTP client/provider owner
normal runtime Provider call chain
who constructs Authorization headers
who attaches tenant/POS/device/source identity headers
WpfJwt issuance, use, storage, expiry, and rejection behavior
whether any other access-token type exists
whether a refresh token/path exists
Pairing Code redeem endpoint and caller
bootstrap/me or protected-hello endpoint and caller
normal grouped-sync endpoint and caller
SignalR hub URL, registration identity, and authentication
401 handling
API-offline behavior
whether API/token currently gates MainWindow
legacy Firebase email/password path count
manual HttpClient/auth-header paths outside the canonical Provider
```

Produce an exact call-chain diagram for:

```text
WPF feature/save
-> local outbox/worker
-> Provider
-> API authentication
-> API grouped sync
-> SignalR notification
```

Also separate:

```text
Phase 1 local installation
Phase 2 cloud pairing
normal runtime API session
```

according to current source and V004.

## Audit area 2 — WPF OBM-POS to CompanionApp

Determine the actual current connection model.

Prove whether communication is:

```text
direct local HTTP
local gateway
SignalR
WebSocket
gRPC
named pipe
file/shared DB
API-mediated only
or absent/unproven
```

Identify:

```text
exact endpoint/port owner
which app hosts the endpoint
which app initiates the connection
authentication or trust mechanism
pairing/handoff mechanism
device/application identity
credential storage
loopback/LAN exposure
request/response models
retry/reconnect behavior
offline behavior
whether CompanionApp can affect WPF installation/runtime state
whether WPF can launch CompanionApp or vice versa
```

Do not treat payment-terminal configuration as CompanionApp authentication unless source proves it.

## Audit area 3 — WPF OBM-POS to BookingConsoleFlutterApp

Determine whether a direct WPF-to-BookingConsole connection exists.

Trace separately:

```text
WPF -> BookingConsole direct path, if any
WPF -> API -> BookingConsole indirect path
BookingConsole -> API/SignalR path
BookingConsole tenant/device registration
BookingConsole credential/onboarding flow
SignalR client registration values
message/event types exchanged
source/destination identifiers
reconnect/offline behavior
```

If no direct WPF-to-BookingConsole path exists, state:

```text
DIRECT_WPF_BOOKINGCONSOLE_CONNECTION=ABSENT
```

and show the proven indirect architecture.

## Audit area 4 — `.env.local` and `.env.production`

Audit all loaders and consumers for both files.

### `.env.local` — Development role

Report:

```text
exact expected path(s)
loader class/method/script
which projects read it
when it loads
configuration precedence
whether process environment overrides it or it overrides process environment
whether it is gitignored
whether Visual Studio and command-line scripts use the same source
which API/WPF/Platform/Booking settings come from it
Development ports and base URLs
Development DB-name/connection key names
secret key names only
Development-only flags
```

### `.env.production` — Production role

Report:

```text
exact expected path(s)
loader class/method/service/deployment boundary
which projects read it
configuration precedence
filesystem protection expectations
Production ports and public/internal base URLs
Production database key names
JWT/signing/OAuth/Firebase/SignalR key names only
whether Development flags can leak into Production
whether Production can accidentally load `.env.local`
```

### Required environment-key classification

For every discovered key name, classify it as one of:

```text
Database credential/configuration
JWT/signing
Google/Platform authentication
Firebase Admin/server-side
Retired Firebase WPF email/password
SignalR
Syncfusion/UI license
Development-only behavior
EmpApp/OTP
Legacy/unused/unproven
```

Never print values.

Identify duplicated, conflicting, retired, or ambiguous keys, but do not edit them.

## Audit area 5 — PlatformAppV0 to WPF OBM-POS

Trace the full current contract:

```text
Platform administrator authorization
Tenant/POS1-POS10 selection
Pairing Code issuance
pairing authorization persistence
WPF redeem
redeem response
WPF protected credential storage
restart/resume behavior
```

Report every field carried across the boundary.

Pay special attention to prompt125 changes:

```text
LocalPosDatabaseName
```

Classify every current occurrence as:

```text
Active current implementation
Compatibility-only
Test-only
Legacy
Contradicts V004
```

V004 target requires PlatformApp not to own or transport local PostgreSQL host, port, username, password, connection string, or database name. Report current deviations only; do not remove them in this task.

## Audit area 6 — PlatformAppV0 to ApiServer

Trace:

```text
PlatformApp API base URL resolution
browser/server-side HTTP path
Google OAuth/admin authorization flow
cookie/JWT/session ownership
PlatformJwt issuer/audience/scope
PlatformApp-to-API authorization headers/cookies
login/me endpoints
Tenant/POS management endpoints
Pairing Code endpoint
error/401 handling
Development and Production configuration sources
```

Clarify whether PlatformApp calls ApiServer:

```text
from browser directly
from Blazor server/BFF-side client
or both
```

Identify CSRF/cookie/SameSite/HTTPS expectations only where source proves them.

## Audit area 7 — BookingOnline / GuestWeb to ApiServer

Trace the complete request path for public booking.

At minimum inspect:

```text
public browser -> GuestWeb/BookingOnline
GuestWeb/BFF -> ApiServer
service-menu endpoint
booking create/read endpoints
Tenant resolution by domain/path/header
public versus authenticated endpoints
customer/member authentication when present
API base URL configuration
server-side credential or anonymous access
SignalR usage, if any
asset/media access
retry/error behavior
API-offline behavior
Development and Production environment sources
```

Produce a Mermaid sequence diagram for:

```text
customer opens website
-> tenant resolved
-> services/employees loaded
-> booking submitted
-> ApiServer commits
-> any notification/event path
```

Do not claim a notification/event path unless source proves it.

## Cross-application security inventory

Create a token/credential ownership table covering at minimum:

```text
PostgreSQL local credential
WpfJwt
PlatformJwt
Google OAuth credential/session
CompanionApp credential/handoff
BookingConsole credential
BookingOnline/customer credential
SignalR bearer token or identity
Firebase Admin service credential
retired Firebase WPF email/password
```

Required columns:

```text
Credential name/type
Issuer
Consumer
Purpose
Storage location/owner
Protection mechanism
Lifetime/expiry
Refresh/renewal
Revocation/re-pair behavior
What failure affects
What failure must not affect
Current status
```

## Required diagrams

Include Mermaid diagrams for:

```text
1. Full OBM application connectivity overview
2. WPF -> API authentication and sync flow
3. WPF <-> CompanionApp proven flow
4. WPF / API / BookingConsole proven flow
5. PlatformApp -> API -> WPF pairing flow
6. BookingOnline -> GuestWeb/BFF -> API flow
7. Development `.env.local` configuration flow
8. Production `.env.production` configuration flow
```

## Required discrepancy analysis

For each area, distinguish:

```text
CURRENT_IMPLEMENTATION
CANONICAL_V004_EXPECTATION
MATCH / PARTIAL / CONFLICT / UNPROVEN
```

Do not propose a broad redesign.

Recommendations must be limited to:

```text
Keep
Clarify/document
Remove later
Consolidate later
Security risk requiring separate approved task
```

No implementation prompt may be generated inside this audit.

## Required private artifact

Create a versioned artifact:

```text
E:\Project2026\RecoveryReports\ConnectivitySecurityInventoryV001
```

Required files:

```text
PRIVATE_HANDOFF.md
DOCS_READ.md
PROJECT_ROOTS.md
MASTER_CONNECTION_MATRIX.md
WPF_API.md
WPF_COMPANIONAPP.md
WPF_BOOKINGCONSOLE.md
ENV_LOCAL_PRODUCTION.md
PLATFORM_WPF.md
PLATFORM_API.md
BOOKINGONLINE_API.md
TOKEN_CREDENTIAL_OWNERSHIP.md
PORT_ENDPOINT_INVENTORY.md
MERMAID_ARCHITECTURE.md
CURRENT_VS_V004.md
RISKS_GAPS.md
SOURCE_EVIDENCE_INDEX.md
FINAL_STATE.md
SHA256SUMS.txt
AGGREGATE_SHA256.txt
```

## Required public report

Create and push:

```text
report/report127.md
```

It must include:

```text
Verdict
Canonical V004 read yes/no
All project roots resolved yes/no
WPF->API status and auth summary
WPF->CompanionApp direct/indirect status
WPF->BookingConsole direct/indirect status
.env.local role summary
.env.production role summary
Platform->WPF status
Platform->API status
BookingOnline->API status
Active credential/token types
Retired credential paths found
Direct manual auth/header paths outside canonical owners
Current vs V004 conflicts count
High-risk findings count
Unproven boundaries count
Source files changed count = 0
Config files changed count = 0
Database mutation count = 0
Service start/stop count = 0
Private artifact path and aggregate SHA-256
```

PASS verdict means the inventory is complete, not that all architecture is correct:

```text
CONNECTIVITY_SECURITY_READONLY_INVENTORY_COMPLETE_READY_FOR_ARCHITECTURE_REVIEW
```

Allowed blockers:

```text
BLOCKED_CANONICAL_V004_NOT_FOUND
BLOCKED_PROJECT_ROOT_NOT_FOUND
BLOCKED_SOURCE_EVIDENCE_INCOMPLETE
BLOCKED_SECRET_EXPOSURE_RISK
```

Do not edit code or configuration to resolve a blocker.
