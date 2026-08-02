# prompt135 — Restore existing PlatformAppV0 / ApiServer local secret configuration from protected operator file

## Objective

Restore the previously working local Development configuration for PlatformAppV0 and ApiServer01 by reusing the existing secret/configuration owners and the operator-provided local protected file:

```text
E:\ThongTinBaoMat\Client_id_secret_demo.txt
```

This is a configuration-recovery and wiring task. Do not redesign authentication, add a new secret system, or expose any secret value.

Current runtime symptom:

```text
PlatformAppV0 Google login state = PASS
POST /api/platform-v0/admin/google/exchange reaches ApiServer01
ApiServer01 returns HTTP 403 PLATFORM_ADMIN_BOOTSTRAP_IDENTITY...
Platform administrator authorization remains Pending
```

ApiServer01 is confirmed running from:

```text
E:\Project2026\1ApiServer\ApiServer01\bin\Debug\net8.0\ApiServer01.exe
```

on:

```text
127.0.0.1:7161
```

PlatformAppV0 runs at:

```text
https://localhost:7012
```

## Mandatory security rules

1. Read `E:\ThongTinBaoMat\Client_id_secret_demo.txt` locally only.
2. Never copy raw file contents into source, Git, prompt/report, console transcript, screenshot, test output, artifact manifest, exception message, or logs.
3. Never commit secrets or generated secret-bearing files.
4. In the public report, identify keys only by safe semantic names and report whether each was found/configured; redact all values.
5. Do not print email addresses, client IDs, client secrets, tokens, private keys, passwords, or full connection strings.
6. Do not initialize a new secret mechanism merely because `ApiServer01.csproj` has no `UserSecretsId`.
7. Reuse the exact configuration mechanism that made this flow work previously.

## Phase A — Forensic configuration audit before mutation

Inspect the active source and local evidence to identify the existing owners and exact required configuration keys for:

```text
Google OAuth client ID
Google OAuth client secret
Google callback/redirect configuration
approved Platform administrator bootstrap identity
PlatformAppV0 -> ApiServer base URL
ApiServer Development environment/profile
```

Search, at minimum:

```text
E:\Project2026\PlatformAppV0
E:\Project2026\1ApiServer\ApiServer01
E:\Project2026\RecoveryReports
Git history/reflog and prior prompt artifacts from the working authorization flow
appsettings*.json
Properties\launchSettings.json
*.props / *.targets
*.ps1 / *.cmd / *.bat
process-level and user-level environment variable wiring
existing local secret loaders or protected config readers
```

Search source for the exact error and configuration boundary:

```text
PLATFORM_ADMIN_BOOTSTRAP_IDENTITY
BOOTSTRAP_IDENTITY
ApprovedAdmin
PlatformAdmin
Google client ID/secret
admin/google/exchange
```

Produce a private evidence table:

| Concern | Active consumer | Exact config key/name | Existing provider/source | Previous working evidence | Required action |
|---|---|---|---|---|---|

Do not mutate until the source of truth is proven.

## Phase B — Parse the operator file safely

Read the local file without echoing values. Determine whether it contains the required existing keys/values.

Rules:

- Use exact existing semantic mappings; do not invent duplicate aliases when a proven key exists.
- Normalize only where the existing contract requires it, for example trimming whitespace or case-insensitive email comparison.
- If the file lacks a required value, stop and report only the missing semantic key name, never nearby file contents.
- If multiple candidate values exist, stop for operator review rather than guessing.

## Phase C — Restore configuration using the existing mechanism

Configure the active Development runtime through the already-established owner discovered in Phase A. Acceptable examples only when proven by source/history include:

```text
existing environment-variable names used by launchSettings or startup scripts
existing local protected config file outside the repository
existing developer-only startup script
existing options binding already consumed by PlatformAppV0/ApiServer01
```

Do not:

```text
add hardcoded secrets to appsettings files
commit a .env or secret file
add UserSecretsId unless the previous working implementation demonstrably used .NET user-secrets and source currently lost only that wiring
create a second options class/provider
bypass the approved administrator identity guard
weaken 403 authorization behavior
accept any Google account
```

The approved Platform administrator identity must remain explicit and fail closed.

## Phase D — Runtime restart and physical verification

1. Stop only the current PlatformAppV0 and ApiServer01 Development instances.
2. Rebuild both active projects.
3. Start ApiServer01 first and prove it listens at `127.0.0.1:7161` from the latest Debug build.
4. Start PlatformAppV0 at `https://localhost:7012`.
5. Execute the real browser flow with the approved Google account.
6. Prove:

```text
Google login state = PASS
Platform administrator authorization state = PASS
Approved identity summary is populated safely
Create/Select Tenant and POS1 becomes enabled
no HTTP 404
no HTTP 403 PLATFORM_ADMIN_BOOTSTRAP_IDENTITY failure
no response-deserialization masking of an authorization error
```

Do not proceed to create Tenant/POS or Pairing Code unless administrator authorization physically passes.

## Scope guard

This task must not change:

```text
V005 installation ordering
WPF DB creation/migration/bootstrap
Pairing Code contract
Tenant/POS identity semantics
sync/SignalR
Category/Booking/Price weights
Companion/payment terminal
refresh-token architecture
```

If the previous working configuration mechanism cannot be proven or restoration requires an authentication redesign, stop with:

```text
BLOCKED_PROMPT135_SECRET_CONFIGURATION_OWNER_UNRESOLVED
```

## Tests

Run the smallest relevant focused tests for:

```text
Platform admin Google exchange
bootstrap identity authorization
PlatformApp API client status/error handling
configuration/options binding
```

Also build:

```text
ApiServer01
PlatformAppV0
```

## Required report

Create and push:

```text
report/report135.md
```

The report must contain only redacted/safe information:

```text
verdict
configuration owner discovered
safe names of required keys
whether each key was found/configured
source files/classes involved
whether any source change was required
build/test totals
runtime ports and process paths
physical authorization result
whether Tenant/POS button became enabled
secret exposure count = 0
destructive DB action count = 0
private artifact version and SHA-256
coordination commit SHA
```

Never include raw secret values or the contents of `Client_id_secret_demo.txt`.

## PASS verdict

Use only after direct physical proof:

```text
PLATFORMAPPV0_EXISTING_SECRET_CONFIGURATION_RESTORED_ADMIN_AUTHORIZATION_PHYSICALLY_PROVEN
```
