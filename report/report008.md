# Report 008 — WPF_IDENTITY_MISMATCH after redeem HTTP 200

## 1. Verdict

`WPF_INSTALLATIONV0_IDENTITY_MISMATCH_CORRECTION_READY_FOR_USER_RETEST`

Root cause was identified and corrected. The physical mismatch was caused by credential expiration precision drift between the redeem response and `/bootstrap/me`: redeem returned a `DateTimeOffset` with sub-second ticks, while `/bootstrap/me` returned the JWT `exp` claim, which is Unix-second precision. WPF compared them exactly and blocked before DPAPI/checkpoint.

## 2. Physical symptom reproduction

User-observed physical result:

```text
Pairing Code redeemed: HTTP 200
WpfJwt received: WPF_IDENTITY_MISMATCH
Protected API call succeeded: WPF_BOOTSTRAP_IDENTITY_VERIFIED
WpfJwt scope verified: wpf.install.bootstrap
Tenant identity verified: OBMDEVV0
POS station identity verified: POS1
PosGuid verified: PASS
Local installation identity verified: PASS
Installation attempt verified: PENDING / mismatch
Credential protected locally: not started
Checkpoint persisted: not started
Restart/resume: not started
Local database touched = false
```

Safe local regression reproduced the same class of failure:

```text
Redeem ExpiresAtUtc: non-rounded DateTimeOffset with ticks
Bootstrap/me CredentialExpiresAtUtc: same instant truncated to Unix seconds
AttemptGuid: equal
AttemptVersion: equal
CredentialExpirationMatched: false
ApiAuthorized checkpoint: absent
```

## 3. Full safe comparison matrix

Physical UI evidence did not expose raw full GUIDs or timestamps. The matrix below combines the user-observed physical evidence with the source-level regression proof. No Pairing Code, WpfJwt, cookie, auth header, or secret is included.

| Field | Redeem value | Bootstrap/me value | Equal | Notes |
|---|---|---|---|---|
| scope | `wpf.install.bootstrap` | `wpf.install.bootstrap` | Yes | User UI showed scope verified. |
| TenantGuid | present | present | Yes | User UI showed Tenant verified. Full GUID was not captured in prompt. |
| TenantCode | `OBMDEVV0` | `OBMDEVV0` | Yes | User UI showed Tenant identity verified. |
| TenantName | present | present | Yes | Covered by Tenant identity comparison in WPF. |
| PosStationId | present | present | Yes | User UI showed POS station verified. |
| PosGuid | present | present | Yes | User UI showed PosGuid PASS. |
| PosName | `POS1` | `POS1` | Yes | User UI showed POS station identity verified. |
| SlotNumber | `1` | `1` | Yes | Platform physical lane is POS1 slot 1. |
| InstallationAttemptGuid | same persisted attempt | same persisted attempt from JWT/state | Yes after investigation | WPF prompt007 incorrectly grouped expiration into `AttemptMatched`, making this look like attempt mismatch. |
| AttemptVersion | `1` | `1` | Yes after investigation | Same persisted attempt record. |
| LocalInstallationGuid | same pending checkpoint value | same JWT/state value | Yes | User UI showed LocalInstallationGuid PASS. |
| CredentialExpiresAtUtc | `now.AddHours(4)` with sub-second ticks | JWT `exp` claim rounded to Unix seconds | No before fix, Yes after fix | Exact mismatched field. |
| ContractVersion | `platform-app-v0.phase1` | `platform-app-v0.phase1` | Yes | Shared contract value. |

Example of the pre-fix failure class:

```text
Redeem:       2026-07-31Txx:xx:xx.1234567+00:00
Bootstrap/me: 2026-07-31Txx:xx:xx.0000000+00:00
Equal: false
```

## 4. Exact mismatched field

```text
CredentialExpiresAtUtc
```

It was incorrectly surfaced as an installation attempt mismatch because prompt007 WPF code grouped:

```text
InstallationAttemptGuid
AttemptVersion
CredentialExpiresAtUtc
```

into one `AttemptMatched` boolean.

## 5. Exact source location creating each conflicting value

Redeem value before fix:

```text
File:
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Controllers\PlatformAppV0Phase1Controller.cs

Method:
Redeem

Source:
var expiresAt = now.AddHours(4);
return Ok(new RedeemPairingCodeResponse(..., expiresAt, ...));
```

JWT/bootstrap value:

```text
File:
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Security\PlatformAppV0TokenService.cs

Method:
CreateWpfToken / WriteJwt

Source:
Expires = expiresAtUtc.UtcDateTime
```

JWT `exp` is serialized as Unix seconds by `JwtSecurityTokenHandler`.

Bootstrap/me value:

```text
File:
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Controllers\PlatformAppV0Phase1Controller.cs

Method:
ResolveBootstrapIdentity

Source:
ClaimUnixTime("exp")
DateTimeOffset.FromUnixTimeSeconds(value)
```

## 6. Root cause

The redeem endpoint used a high-precision `DateTimeOffset`:

```text
now.AddHours(4)
```

The same value was put into the JWT. The JWT `exp` claim is represented in Unix seconds, so sub-second ticks are lost. `/bootstrap/me` correctly read the token `exp` claim back as a second-precision timestamp.

WPF then compared:

```text
redeem.ExpiresAtUtc == bootstrap.CredentialExpiresAtUtc
```

This failed even though the token, attempt, tenant, POS, and local installation identities were otherwise correct.

## 7. Why prompt007 tests did not catch it

Prompt007 tests built test identities from a single `TestIds.ExpiresAtUtc` value and used that exact same value in both fake redeem and fake bootstrap responses. That meant the test never simulated JWT `exp` second-precision truncation.

Prompt007 also grouped credential expiration into `AttemptMatched`, so UI showed the failure near installation attempt rather than as `Credential expiration verified = false`.

## 8. Exact files changed

Prompt008 direct changes:

```text
E:\Project2026\1ApiServer\ApiServer01\PlatformAppV0\Controllers\PlatformAppV0Phase1Controller.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\Phase1InstallationResult.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Infrastructure\Phase1InstallationService.cs
E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs
```

## 9. Exact correction

ApiServer redeem now canonicalizes credential expiration to JWT second precision before both issuing the token and returning the redeem response:

```text
var expiresAt = ToJwtSecondPrecision(now.AddHours(4));
DateTimeOffset.FromUnixTimeSeconds(value.ToUnixTimeSeconds())
```

WPF comparison now separates:

```text
InstallationAttemptGuid
AttemptVersion
CredentialExpiresAtUtc
ContractVersion
```

WPF proof UI now displays:

```text
AttemptVersion verified
Credential expiration verified
ContractVersion verified
```

WPF no longer reports a credential expiration mismatch as an installation-attempt mismatch.

## 10. Focused tests added

Added/updated tests under:

```text
E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs
```

Coverage added:

```text
JwtExpirationPrecisionMismatch_IsReportedSeparatelyFromAttemptIdentity
ApiServerRedeem_CanonicalizesCredentialExpirationToJwtSecondPrecision
```

Existing tests also cover:

```text
exact redeem/bootstrap identity equality
installation attempt GUID mismatch detection
attempt version mismatch detection
local installation mismatch detection
expiration mismatch handling
DTO JSON property mapping via fake HTTP serialization
no credential/checkpoint persistence on mismatch
successful full identity comparison proceeds to DPAPI/checkpoint
restart/resume without redeem
no database dependency/reference/activity in Phase 1
```

## 11. Build/test commands and counts

Command:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj
```

Result:

```text
PASS
0 warnings
0 errors
```

Command:

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj
```

Result:

```text
PASS
0 warnings
0 errors
```

Command:

```text
dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0" --logger "console;verbosity=minimal"
```

Result:

```text
PASS
Failed: 0
Passed: 16
Skipped: 0
Total: 16
```

Command:

```text
dotnet test E:\Project2026\1ApiServer\ApiServer01.Tests\ApiServer01.Tests.csproj --filter "FullyQualifiedName~PlatformAppV0" --logger "console;verbosity=minimal"
```

Result:

```text
PASS
Failed: 0
Passed: 13
Skipped: 0
Total: 13
```

## 12. Process/binary handoff status

Stopped stale processes before build/test:

```text
ApiServer01 PID 28904
NailSalonNet8 PID 60004
```

Final handoff:

```text
No ApiServer01 process left running by Codex.
No NailSalonNet8/WPF process left running by Codex.
Port 7161 is free for Visual Studio ApiServer debug.
```

PlatformAppV0 may still be user-managed separately on port 7012; this task did not require stopping it.

## 13. No Pairing Code/token logged

Confirmed:

```text
No Pairing Code plaintext in report.
No WpfJwt in report.
No Authorization header value in report.
No token/cookie/client secret/password in report.
```

## 14. No DB/Phase 2

Confirmed:

```text
No PostgreSQL connection.
No local database creation.
No migrations.
No seed.
No TblLocalOutbox.
No PlatformEnrollment.
No permanent device credential.
No Phase 2.
```

## 15. Exact user retest steps

1. Create a new one-time Pairing Code if the old code was already redeemed.
2. In Visual Studio, start ApiServer:

```text
ApiServer01 PlatformAppV0 Phase1
```

3. In Visual Studio, start WPF:

```text
OBM-POS InstallationV0 Phase1
```

4. Enter the Pairing Code manually.
5. Click `Redeem and Verify API Access`.
6. Expected:

```text
Pairing Code redeemed = PASS
WpfJwt received = PASS
Protected API call succeeded = PASS
Scope verified = PASS
Tenant verified = PASS
POS verified = PASS
PosGuid verified = PASS
InstallationAttemptGuid verified = PASS
AttemptVersion verified = PASS
LocalInstallationGuid verified = PASS
Credential expiration verified = PASS
ContractVersion verified = PASS
Credential protected locally = PASS
Protected credential readback = PASS
Machine checkpoint persisted = PASS
Checkpoint readback = PASS
Local database touched = false
```

7. Close WPF.
8. Start the same WPF profile again with the same ProductRoot.
9. Expected restart/resume:

```text
No Pairing Code requested.
No second redeem.
bootstrap/me revalidated.
Restart/resume verified = PASS.
```

10. Do not start Phase 2.

## 16. Coordination commit

This report is intended to be committed only to:

```text
lequochung99/OBM-AI-Coordination
report/report008.md
```
