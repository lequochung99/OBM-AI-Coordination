# Prompt 114 Addendum — Canonical Provider owns runtime sync authentication

This addendum is binding and must be read together with `prompt/prompt114.md` before execution.

## Authoritative operator correction

The operator previously built one canonical Provider for WPF-to-API communication.

Its responsibility is to:

```text
obtain or resume the existing runtime application credential/token
attach the canonical authorization and identity headers
send the request through the standard API client path
allow the API to authenticate the application's legal/runtime identity
```

The caller must only call the Provider. The caller must not separately manage token details.

The canonical outbound path is therefore:

```text
WPF domain Save + TblLocalOutbox
-> existing periodic WPF outbox worker
-> existing canonical Provider
-> Provider obtains/resumes credential internally
-> Provider attaches canonical token/header set internally
-> canonical API grouped sync endpoint
-> API authenticates application identity and validates tenant/source identity
-> durable EventLog/Delivery commit
-> post-commit SignalR notification
```

## Mandatory architectural interpretation

Prompt114 must not treat token acquisition, token storage, header construction, refresh/resume, or request authentication as responsibilities of:

```text
Price Rule Save
TblLocalOutbox writer
OutboxPublisherWorker
transaction-group selector/claimer
Category Weight or Booking Weight code
individual sync call sites
ad-hoc HttpClient code
```

Those components call the canonical Provider and receive a typed success/failure result.

Do not create or restore:

```text
manual Bearer-token handling in the worker
manual authorization-header construction in the worker
manual tenant/source headers outside the Provider when the Provider already owns them
new token service
new token cache
new JWT scheme
new authentication endpoint
new provider
parallel HttpClient transport
Firebase email/password fallback
hardcoded test token in production source
```

Do not broaden installation-scoped `WpfJwt` merely to satisfy runtime sync. Audit the actual canonical Provider and preserve whichever existing runtime credential contract it already implements.

## Required audit before changing auth-related code

Identify the exact current production Provider and return direct evidence for:

```text
PROVIDER_INTERFACE=<exact interface>
PROVIDER_IMPLEMENTATION=<exact class>
PROVIDER_DI_REGISTRATION=<exact registration>
WORKER_TO_PROVIDER_CALL=<exact method chain>
TOKEN_OR_SESSION_RESOLUTION_OWNER=<exact Provider/helper owned by Provider>
AUTHORIZATION_HEADER_OWNER=<exact Provider/handler>
TENANT_SOURCE_HEADER_OWNER=<exact Provider/handler when applicable>
API_AUTH_SCHEME_OR_POLICY=<exact existing scheme/policy>
API_APPLICATION_IDENTITY_BINDING=<exact claims/header validation chain>
```

Inspect all alternative Provider implementations, HTTP clients, delegating handlers, token/session stores, and direct authorization-header assignments. Prove that only one production Provider path is reachable from the periodic outbox worker.

## Prompt114 role-contract work remains separate

The PostgreSQL role repair in prompt114 is an infrastructure database boundary.

Do not confuse:

```text
PostgreSQL runtime database credential
PostgreSQL migration/provisioning credential
WPF Provider runtime API credential/token
Pairing Code/WpfJwt installation bootstrap credential
Platform administrator authentication
```

Each has a separate purpose. Repairing PostgreSQL privileges must not redesign or replace the canonical WPF Provider authentication path.

## Happy-path requirements added to prompt114

After the PostgreSQL role contract is repaired, the physical happy path must prove:

```text
existing periodic WPF worker called the canonical Provider
canonical Provider invocation count = 1 for the group
manual token access outside Provider count = 0
manual authorization-header construction outside Provider count = 0
Provider resolved/resumed its credential internally
Provider attached the canonical authorization/identity headers internally
API authenticated the application's runtime identity
API matched authenticated identity to TenantGuid and SourceClientId
one grouped request reached the canonical API ingest action
no parallel Provider or direct HttpClient sync path was invoked
```

Evidence must redact token/header values. Report only header names, ownership, call-chain locations, result codes, and counts.

If the Provider cannot obtain/resume its existing credential, return a narrow blocker such as:

```text
BLOCKED_MAIN_DEV_CANONICAL_PROVIDER_CREDENTIAL
```

The blocker must identify the exact Provider method and protected credential/session boundary. Do not bypass the Provider or ask the operator to paste a token.

## Required private evidence additions

Add to the prompt114 private artifact:

```text
CANONICAL_PROVIDER_CALL_CHAIN.md
PROVIDER_AUTH_HEADER_OWNERSHIP.md
PROVIDER_APPLICATION_IDENTITY_PROOF.md
NO_MANUAL_TOKEN_HANDLING_PROOF.md
```

## Required public report additions

Add these fields to `report/report114.md`:

```text
Canonical WPF Provider identified yes/no
Existing periodic worker used canonical Provider yes/no
Canonical Provider invocation count
Provider resolved/resumed credential internally yes/no
Provider attached canonical auth/identity headers internally yes/no
Manual token access outside Provider count
Manual auth-header construction outside Provider count
API application identity authenticated yes/no
Authenticated tenant/source identity matched yes/no
Parallel Provider/direct HttpClient sync path invoked count
```

## Acceptance lock

`MANUAL_POS1_TEST_READY=true` is forbidden unless the physical happy path uses the canonical Provider exactly as above.

Service Category Weight and Booking Weight remain deferred until prompt114 passes. Future Weight modules must reuse this same Provider and must never implement their own authentication/header logic.
