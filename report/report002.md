# Report 002 — PlatformAppV0 administrator authorization fix

## Verdict

`PLATFORMAPPV0_ADMIN_AUTHORIZATION_FIX_READY_FOR_USER_RUNTIME_TEST`

## Root cause

PlatformAppV0 was using the ASP.NET Core Google OAuth handler while the `Authorize Platform Administrator` action tried to read `id_token` later from the authenticated cookie/session path. In the live flow, Google login could complete, but the raw Google ID token was not available when the Blazor action ran, so the admin exchange failed closed with:

```text
Google ID token unavailable from authenticated session
```

The blocker was not a Tenant/POS issue and not a WPF issue. It was the PlatformAppV0 admin authorization token handoff after Google sign-in.

## Files changed

Source files changed under `E:\Project2026`:

- `E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\OBM.PlatformAppV0.csproj`
- `E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\Program.cs`
- `E:\Project2026\PlatformAppV0\src\OBM.PlatformAppV0\Pages\Home.razor`
- `E:\Project2026\PlatformAppV0\tests\OBM.PlatformAppV0.Tests\Phase1ContractTests.cs`

Important provenance note: `Home.razor` already contained substantial PlatformAppV0 UI work from earlier prompts. Report 002 only claims the administrator authorization and Tenant/POS gate correction, not authorship of every pre-existing UI line in that file.

## Exact fix

- Replaced the Google OAuth handler dependency with OpenID Connect for PlatformAppV0 Google sign-in.
- Configured Google OIDC with:
  - `Authority = https://accounts.google.com`
  - `ResponseType = code`
  - `SaveTokens = true`
  - scopes `openid`, `email`, `profile`
  - fixed callback path `/signin-google-callback`
- Added a server-side `/platform-admin/authorize` endpoint.
- The endpoint authenticates the existing cookie, reads the saved `id_token`, calls the canonical PlatformAppV0 API admin Google exchange, then persists only the approved admin proof into the encrypted authentication cookie.
- The UI now hydrates administrator authorization from safe cookie claims after reload/refresh.
- The `Authorize Platform Administrator` control is now a real server navigation to `/platform-admin/authorize`, not a Blazor circuit-local token lookup.
- `Create/Select Tenant and POS1` remains disabled until both Google login and Platform administrator authorization pass.

No token, secret, cookie, or raw Google credential is logged or written to this report.

## Build and test results

Command:

```text
dotnet build E:\Project2026\PlatformAppV0\PlatformAppV0.sln
```

Result:

```text
PASS — 0 warnings, 0 errors
```

Command:

```text
dotnet test E:\Project2026\PlatformAppV0\PlatformAppV0.sln --no-build --logger "console;verbosity=minimal"
```

Result:

```text
PASS — Failed: 0, Passed: 8, Skipped: 0, Total: 8
```

Command:

```text
dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0" --logger "console;verbosity=minimal"
```

Result:

```text
BLOCKED BY RUNNING PROCESS — ApiServer01 PID 50140 locked OBM.PlatformAppV0.Contracts.dll during build/copy.
```

Command:

```text
dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --no-build --filter "FullyQualifiedName~PlatformAppV0" --logger "console;verbosity=minimal"
```

Result:

```text
PASS — Failed: 0, Passed: 3, Skipped: 0, Total: 3
```

## Runtime evidence

Runtime physical clicking was not completed by Codex after the fix because the Google browser/operator flow requires the user to sign in and press the authorization button. Therefore this report does not claim the full physical PASS verdict.

Prepared state for user runtime test:

- Google sign-in route remains `/signin-google`.
- Google callback remains `/signin-google-callback`.
- Admin authorization route is now `/platform-admin/authorize`.
- Admin exchange still uses the canonical PlatformAppV0 API Google exchange.
- Tenant/POS action remains gated until admin authorization succeeds.
- Pairing Code remains gated until Tenant/POS prerequisite succeeds.

Expected user-visible result after retest:

```text
Google login state: PASS
Platform administrator authorization state: PASS
Approved identity summary: populated
Create/Select Tenant and POS1: enabled
```

## Remaining blocker

User/operator runtime validation is still required:

- Click `Continue with Google`.
- Complete Google sign-in.
- Click `Authorize Platform Administrator`.
- Confirm no `Google ID token unavailable from authenticated session` message appears.
- Confirm approved identity is populated.
- Confirm `Create/Select Tenant and POS1` becomes enabled and can be clicked.

## Source Git state

`E:\Project2026` is a shared parent repository with pre-existing dirty state and no source remote. Per prompt safety rules:

- No source commit was created.
- No source push was attempted.
- No `git add .`, reset, clean, stash, checkout, or restore was used.
- Source changes remain local for the operator to review in `E:\Project2026`.

Source local commit SHA:

```text
N/A — source was intentionally not committed.
```

## Confirmation source was not pushed

Confirmed. Source repository was not pushed.

## Exact user retest steps

1. Start ApiServer normally if it is not already running.
2. Start PlatformAppV0 from the canonical Visual Studio launch profile on `https://localhost:7012`.
3. Open `https://localhost:7012`.
4. Click `Continue with Google`.
5. Complete Google sign-in with the provisioned administrator account.
6. Click `Authorize Platform Administrator`.
7. Verify administrator authorization becomes `PASS`.
8. Verify the approved administrator identity is populated.
9. Verify `Create/Select Tenant and POS1` is enabled.
10. Click `Create/Select Tenant and POS1`.
11. Record the specific Tenant/POS result.
12. Do not start WPF redeem or Phase 2 in this retest.

## Next recommended task

Run the physical PlatformAppV0 retest and report whether the admin authorization gate reaches PASS and whether Tenant/POS1 creation or selection succeeds. If that passes, the next task can proceed to the Pairing Code prerequisite, still without Phase 2 or local POS database work.
