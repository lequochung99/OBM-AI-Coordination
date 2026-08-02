# Prompt 116 Addendum — POS station aggregate may own OBM-POS, CompanionApp, and payment-terminal assignments

This addendum is binding and must be read together with `prompt/prompt116.md` before execution.

## Authoritative operator clarification

The product supports logical station slots:

```text
POS1
POS2
...
POS10
```

A logical POS station may be associated with more than one runtime component. For example:

```text
POS1
- OBM-POS WPF application/device for POS1
- CompanionApp instance/device associated with POS1
- payment terminal or terminal configuration assigned to POS1
```

Likewise POS2 should be associated with CompanionApp2 and the terminal/payment configuration assigned to POS2.

This clarification means prompt116 must not assume that `TblTenantPosDevice` is necessarily exactly one row per logical POS station.

## Required conceptual separation

Audit the source against this candidate aggregate model:

```text
TblPosLocal
- logical tenant POS station/slot
- POS1..POS10 identity, name, slot, and station-level configuration

TblTenantPosDevice
- physical/application device registrations associated with a logical station
- potentially more than one device/application identity per station
- possible device/application roles include OBM-POS, CompanionApp, or another proven runtime client

Terminal/payment configuration tables
- payment-terminal configuration and merchant/device settings
- linked to the logical POS station and/or a proven device identity
- remain in their existing canonical tables when those tables already own the contract
```

This is an audit hypothesis, not permission to invent fields or force every component into `TblTenantPosDevice`.

## Mandatory audit expansion

Before finalizing the `TblTenantPosDevice` schema or writer, inspect all active and retained historical contracts for:

```text
CompanionApp identity registration
CompanionApp pairing/redeem/authentication
CompanionApp SourceClientId/SubscriberId format
CompanionApp SignalR registration and pull subscriptions
POS1..POS10 station-slot ownership
terminal-payment setup ownership
terminal assignment to POS station
TblTerminalDejavoo
TblTerminalPaymentInfo
all terminal setup/configuration tables and services
all fields using PosGuid, PosStationId, PosDeviceGuid, TerminalGuid, DeviceGuid, ClientId, or SubscriberId
```

Produce a station-component matrix:

```text
Component type
Logical station binding
Physical/application identity key
Canonical table owner
Canonical writer
Authentication required yes/no
SourceClientId/DestinationClientId applicable yes/no
SignalR/pull subscription applicable yes/no
Replacement/deactivation lifecycle
Cardinality per station
```

At minimum classify:

```text
OBM_POS_WPF
COMPANION_APP
PAYMENT_TERMINAL
OTHER_PROVEN_RUNTIME_CLIENT
STATION_CONFIGURATION_ONLY
```

Do not infer that a payment terminal is an authenticated API sync client merely because it is physically paired with a POS station.

## Cardinality must be proven

Do not assume any of these without evidence:

```text
exactly one TblTenantPosDevice row per station
exactly one CompanionApp per station
exactly one terminal per station
one shared device GUID for OBM-POS and CompanionApp
one shared SourceClientId for all station components
all station components receive the same subscriptions
```

Resolve the actual supported cardinality from code and schema. The likely production shape may be:

```text
one logical TblPosLocal station
-> zero or one active OBM-POS runtime registration
-> zero or one active CompanionApp registration
-> zero, one, or more terminal assignments according to the proven terminal contract
```

But prompt116 must report the proven result rather than forcing this candidate shape.

## Device-role and uniqueness requirements

If evidence proves that multiple runtime application/device rows belong in `TblTenantPosDevice`, the schema must include or derive a proven discriminator such as:

```text
DeviceType
ApplicationType
ClientType
SubscriberType
or another existing canonical equivalent
```

Do not invent a new enum when an existing canonical discriminator already exists.

Unique constraints must prevent conflicting active registrations while permitting legitimate station components. Resolve and prove constraints such as:

```text
Tenant + station + component/device role + active registration
physical DeviceGuid uniqueness
SourceClientId uniqueness within its proven scope
replacement device deactivates or supersedes the prior active row
```

Do not use one uniqueness rule that incorrectly prevents POS1 from having both an OBM-POS device and CompanionApp1.

## Writer lifecycle requirements

The canonical writer lifecycle may differ by component:

```text
OBM-POS registration may occur at pairing redeem/bootstrap activation
CompanionApp registration may occur through its own existing pairing/authorization boundary
payment-terminal assignment may occur through terminal setup rather than application pairing
```

Prompt116 must not create one generic writer that fabricates all three components during WPF pairing.

For each component, prove:

```text
writer service and method
create trigger
update trigger
replacement/deactivation trigger
logical station binding
idempotency key
whether API authentication identity is created
whether sync routing eligibility is created
```

Only reconnect or implement the writer required for the current POS1-to-API grouped-sync happy path. Preserve and document other proven component contracts without widening this task unnecessarily.

## Destination-routing eligibility

`TblTenantPosDevice` must not cause every component under a station to receive every sync entity.

Destination selection must continue to require the proven combination of:

```text
same TenantGuid
active/non-revoked runtime registration
eligible application/device role
matching SubscriberId/entity subscription
concrete DestinationClientId
source-client exclusion
```

Examples:

```text
OBM-POS destination may subscribe to TblTurnPolicy/TblTurnAmountRule
CompanionApp may have a different subscriber/entity set
payment terminal must not receive general application sync deliveries unless an explicit existing contract proves it is an API sync subscriber
```

Do not route by logical station alone.

Do not create deliveries for terminal configuration objects merely because a terminal is assigned to POS2.

## Terminal configuration ownership

Audit whether terminal settings already belong to tables such as:

```text
TblTerminalDejavoo
TblTerminalPaymentInfo
or other current terminal setup tables
```

When an existing table owns merchant IDs, terminal IDs, processor configuration, connection settings, or payment behavior:

```text
keep those settings in that canonical table
link them to TblPosLocal and/or a proven device registration using existing keys
store only routing/identity metadata in TblTenantPosDevice
```

Do not move payment secrets, merchant credentials, processor tokens, or detailed terminal configuration into `TblTenantPosDevice`.

Do not expose terminal credentials or identifiers in public reports.

## Required prompt116 private evidence additions

Add:

```text
POS_STATION_COMPONENT_AGGREGATE.md
COMPANIONAPP_IDENTITY_AND_WRITER_AUDIT.md
TERMINAL_ASSIGNMENT_AND_SETTINGS_OWNER.md
DEVICE_ROLE_CARDINALITY_AND_UNIQUENESS.md
ROUTING_ELIGIBILITY_BY_COMPONENT.md
```

## Required report116 additions

Add these public, sanitized fields:

```text
POS station aggregate model resolved yes/no
Logical station table owner
Runtime component/device registry owner
Proven component/device roles
OBM-POS registrations per station cardinality
CompanionApp registrations per station cardinality
Payment-terminal assignment cardinality
CompanionApp canonical writer identified yes/no/not-in-scope
Terminal configuration canonical table owner identified yes/no
Terminal treated as general sync destination yes/no and reason
Routing filters by eligible component role yes/no
One station can contain both OBM-POS and CompanionApp registrations yes/no/not-proven
```

## Acceptance lock

Prompt116 may create the `TblTenantPosDevice` migration only after proving a schema that supports the real station-component model without collapsing distinct logical station, application-device, CompanionApp, and terminal-setting responsibilities.

For the current happy path, it is sufficient to register the Development POS1 and POS2 OBM-POS runtime identities needed for routing. Do not fabricate CompanionApp or terminal rows merely to populate the new table.

`MANUAL_POS1_TEST_READY=true` remains forbidden until the original prompt116 physical happy path passes through the canonical Provider, API routing, durable commit, post-commit SignalR, and atomic local completion.
