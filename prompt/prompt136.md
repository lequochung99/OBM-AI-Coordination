# prompt136 — Restore the approved Platform administrator bootstrap identity in the existing protected runtime configuration and physically prove authorization

## Objective

Restore the missing Platform administrator bootstrap identity required by the existing ApiServer01 PlatformAppV0 authorization flow, using the already active protected runtime configuration owner. Then physically prove that the approved Google administrator can authorize PlatformAppV0 and unlock Tenant/POS actions.

This is a configuration recovery task, not an Identity redesign.

## Current proven state

From report135:

- ApiServer01 is reachable on `http://127.0.0.1:7161`.
- `POST /api/platform-v0/admin/google/exchange` is mapped.
- Google ClientId/ClientSecret are loaded from the existing protected runtime configuration source.
- The current protected configuration does not provide an approved administrator bootstrap identity.
- The existing source consumes one of these keys:
  - `Authentication__Google__ApprovedAdminEmail`
  - `Authentication__Google__ApprovedGoogleSubject`
  - `PlatformAppV0__ApprovedAdminEmail`
  - `PlatformAppV0__ApprovedGoogleSubject`
- The operator-approved Google administrator account is the account previously identified by the operator in this coordination session. Do not print or commit its value.

## Mandatory security boundary

The local protected configuration source is:

`E:\ThongTinBaoMat\Client_id_secret_demo.txt`

Use it only on the operator machine.

Do not:

- commit this file or its contents;
- print email, Google subject, ClientId, ClientSecret, tokens, cookies, or full environment values;
- copy secrets into source, appsettings, launchSettings, Git, reports, screenshots, command history artifacts, or test fixtures;
- create a second secret/configuration mechanism;
- initialize .NET user-secrets unless direct source evidence proves that the active runtime already uses that owner;
- weaken or bypass `PLATFORM_ADMIN_BOOTSTRAP_IDENTITY` authorization;
- accept arbitrary Google accounts.

Public evidence may report only key presence, source ownership, result codes, and PASS/FAIL state.

## Phase A — Confirm the active configuration owner and key precedence

Before mutation, inspect the active ApiServer01 startup/configuration wiring and determine:

1. How `E:\ThongTinBaoMat\Client_id_secret_demo.txt` is currently loaded.
2. Its exact syntax and key/value format.
3. Which approved-admin key is consumed first by the active source.
4. Whether email or Google provider subject is already available safely from prior local protected evidence.
5. Whether a Platform administrator already exists in the API database.

Read-only DB check first:

- If an existing active Platform administrator mapping already exists and the exchange endpoint is incorrectly ignoring it, stop with:
  `BLOCKED_PROMPT136_EXISTING_IDENTITY_WIRING_REVIEW_REQUIRED`
- Do not create duplicate Platform users or roles.
- If no Platform administrator exists, continue with the one-time bootstrap configuration recovery.

## Phase B — Add the bootstrap identity to the existing protected owner

Prefer the smallest compatible key already consumed by active source.

Default preference when source semantics are equivalent:

`Authentication__Google__ApprovedAdminEmail`

Use `ApprovedGoogleSubject` only if a verified provider subject is already available through a safe local source and the active contract clearly prefers it.

Requirements:

- Add exactly one approved bootstrap identity value.
- Preserve the existing ClientId/ClientSecret entries unchanged.
- Preserve file encoding and existing syntax.
- Make a private local backup before editing, outside Git.
- Use an atomic or replace-safe local write when practical.
- Verify only that the key is present and non-empty; never echo the value.

## Phase C — Restart the latest runtime instances

1. Stop stale ApiServer01 and PlatformAppV0 processes.
2. Build the latest source.
3. Start ApiServer01 using the same protected runtime configuration path and the PlatformAppV0 Phase1 profile on `http://127.0.0.1:7161`.
4. Prove readiness HTTP 200.
5. Start PlatformAppV0 in the operator Windows session that owns the ASP.NET DataProtection key ring and HTTPS development certificate.
6. Confirm PlatformAppV0 is on `https://localhost:7012`.

Do not use a runner/session that lacks DataProtection key-ring access for the final UI proof.

## Phase D — Physical authorization proof

Using the approved Google administrator account:

1. Open PlatformAppV0.
2. Complete Google sign-in.
3. Click `Authorize Platform Administrator`.
4. Prove:
   - Google login state = PASS.
   - Platform administrator authorization state = PASS.
   - No `PLATFORM_ADMIN_BOOTSTRAP_IDENTITY` 403 remains.
   - `Create/Select Tenant and POS1` is enabled.
5. Refresh/reopen PlatformAppV0 once and prove the authorized state remains valid through the existing session/Identity path.

Do not create a Tenant or Pairing Code in this prompt unless required only to prove the button is enabled. This prompt ends at successful administrator authorization and unlock.

## Source-change rule

Configuration-only recovery is preferred.

Do not edit source unless physical evidence proves a narrow defect in existing key binding/precedence. If source change is required:

- stop before editing;
- document the exact proven defect;
- return `BLOCKED_PROMPT136_SOURCE_CHANGE_REQUIRES_REVIEW`.

## Tests

Run at minimum:

- PlatformAppV0 build;
- PlatformAppV0 tests;
- ApiServer01 PlatformAppV0 focused tests.

Do not add tests containing real identities or secret values.

## Report

Create and push:

`report/report136.md`

The public report must contain:

- active configuration owner classification;
- selected key name, but not its value;
- whether an existing DB Platform administrator was found;
- whether configuration-only recovery was sufficient;
- API readiness result;
- Google login PASS/FAIL;
- Platform administrator authorization PASS/FAIL;
- Tenant/POS action enabled PASS/FAIL;
- build/test totals;
- zero secret exposure confirmation;
- private artifact version and SHA-256 if created;
- real coordination commit SHA.

## PASS verdict

Use only when physically proven:

`PLATFORMAPPV0_BOOTSTRAP_ADMIN_IDENTITY_RESTORED_AUTHORIZATION_PHYSICALLY_PROVEN`

Otherwise return an exact blocker without claiming PASS.
