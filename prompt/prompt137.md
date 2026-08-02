# prompt137 — Restore existing Platform administrator Identity login before bootstrap fallback

## Objective

Restore PlatformAppV0 administrator authorization by wiring the existing active Platform administrator mapping as the primary authorization path for `POST /api/platform-v0/admin/google/exchange`.

Prompt136 proved that the current PlatformAppV0 state already contains exactly one active administrator mapping with identity fields present. Therefore do **not** add a second bootstrap identity, do **not** create a duplicate administrator, and do **not** weaken authorization.

The intended order is:

```text
validate Google ID token
-> normalize and inspect verified Google identity
-> lookup existing active Platform administrator mapping by the established provider identity contract
-> if existing mapping matches, authorize and issue the existing Platform session/token
-> only when no Platform administrator exists at all may the approved bootstrap identity gate be evaluated
```

## Canonical boundaries

Repositories:

```text
PlatformAppV0: E:\Project2026\PlatformAppV0
ApiServer01:    E:\Project2026\1ApiServer\ApiServer01
```

Current runtime ports:

```text
ApiServer01:   http://127.0.0.1:7161
PlatformAppV0: https://localhost:7012
```

Existing protected operator configuration file:

```text
E:\ThongTinBaoMat\Client_id_secret_demo.txt
```

That file currently provides Google OAuth ClientId/ClientSecret. Do not add ApprovedAdminEmail or ApprovedGoogleSubject unless Phase A proves there is no active administrator mapping after all. Prompt136 already reported one active mapping, so the expected path is existing-identity wiring, not bootstrap configuration.

## Phase A — Read-only identity and call-chain audit

Before editing source, inspect and document privately:

1. Exact database/state owner containing the active Platform administrator mapping.
2. Exact table/entity/model names.
3. Whether the mapping stores:
   - Google provider subject (`sub`),
   - normalized email,
   - provider name,
   - active/enabled flag,
   - Platform administrator role/claim.
4. Exact `/api/platform-v0/admin/google/exchange` call chain from controller/endpoint through token validation, bootstrap policy, identity lookup, and token/session issuance.
5. Current decision order that produces `PLATFORM_ADMIN_BOOTSTRAP_IDENTITY...` despite an existing active administrator mapping.
6. Whether the current Google account matches the stored mapping by provider subject, verified normalized email, or both. Do not print the values.

Produce a private decision table:

| Decision concern | Current owner | Current order | Required order | Minimal correction |
|---|---|---|---|---|
| Google token validation | | | | |
| Existing administrator lookup | | | | |
| Active/role check | | | | |
| Bootstrap fallback | | | | |
| Platform session/JWT issuance | | | | |

If the stored mapping cannot be matched safely to the verified Google identity, stop with:

```text
BLOCKED_PROMPT137_EXISTING_ADMIN_IDENTITY_MISMATCH_REQUIRES_OPERATOR_REVIEW
```

Do not silently bind a different account.

## Phase B — Minimal wiring correction

When the existing active mapping matches the verified Google identity, implement the smallest correction using existing owners:

```text
Google ID token valid
-> existing active administrator mapping found
-> authorize existing administrator
-> issue existing Platform authorization result
```

Bootstrap policy must become a fallback used only when:

```text
no active Platform administrator mapping exists
```

Required invariants:

- Existing administrator mapping remains the source of authority after first bootstrap.
- Bootstrap identity does not gate normal login for an already-mapped active administrator.
- Disabled/inactive mapping remains denied.
- A valid Google account with no existing mapping remains denied when an administrator already exists.
- No automatic second administrator creation.
- No authorization by email alone when the proven source contract requires Google subject.
- Preserve existing audit logging and token/session issuance.
- Reuse existing entity/store/service/controller owners.

Do not create:

- a second identity table,
- a second Google exchange endpoint,
- a second Platform JWT/session scheme,
- a new secret store,
- a permissive development bypass.

## Required tests

Add or update focused tests proving at minimum:

1. Existing active administrator mapping + matching verified Google identity -> authorization succeeds without bootstrap configuration.
2. Existing inactive/disabled administrator mapping -> denied.
3. Existing administrator already present + unmatched Google identity -> denied; bootstrap fallback cannot create a second admin.
4. No administrator exists + approved bootstrap identity configured -> existing first-admin bootstrap behavior still works.
5. No administrator exists + no approved bootstrap identity -> fail closed.
6. Fake/invalid Google token -> `GOOGLE_ID_TOKEN_VALIDATION_FAILED` or established equivalent.
7. Existing API response contract remains deserializable by PlatformAppV0.

Run:

```text
PlatformAppV0 build/tests
ApiServer01 PlatformAppV0 focused build/tests
```

## Physical proof

After tests pass:

1. Start the latest ApiServer01 instance on `127.0.0.1:7161` using the existing protected runtime configuration.
2. Start PlatformAppV0 on `https://localhost:7012` in the operator Windows session with DataProtection access.
3. Sign in with the Google account that corresponds to the existing active administrator mapping.
4. Click `Authorize Platform Administrator`.
5. Prove:

```text
Google login state = PASS
Platform administrator authorization state = PASS
existing administrator mapping reused = yes
new administrator mappings created = 0
Create/Select Tenant and POS1 = enabled
```

Do not print email, Google subject, token, cookie, ClientId, ClientSecret, raw GUID, or private payload values.

## Frozen scope

Do not modify:

- V005 WPF installation flow,
- Pairing Code creation/redeem contracts,
- local PostgreSQL DB V1,
- migrations/bootstrap seed,
- sync/SignalR,
- Tenant/POS domain design,
- Companion/payment/Firebase configuration.

## Report

Create and push:

```text
report/report137.md
```

The report must include:

- exact root-cause classification,
- existing identity owner and safe structural description,
- old versus corrected authorization order,
- number of existing administrator mappings before/after,
- build/test totals,
- physical authorization results,
- zero secret/identity exposure confirmation,
- private artifact version and SHA-256,
- coordination commit SHA.

PASS verdict only when physical authorization succeeds:

```text
PLATFORMAPPV0_EXISTING_ADMIN_IDENTITY_AUTHORIZATION_PHYSICALLY_RESTORED
```

Otherwise use the exact blocker encountered and do not claim PASS.
