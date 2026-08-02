# Prompt 118 Addendum — Investigate redeem token/refresh-token persistence before any broader startup work

This addendum is binding and must be read together with:

```text
prompt/prompt118.md
prompt/prompt118_MINIMAL_WPF_STARTUP_RECOVERY_ADDENDUM.md
```

It overrides any execution order that begins with a broad ProductRoot/startup architecture audit.

## Authoritative operator observation

The operator previously completed Pairing Code redemption and observed that the WPF received both the access/bootstrap token and refresh token through the accepted installation flow.

The current startup now shows:

```text
Phase 2 Local DB Baseline: awaiting Phase 1 resume
Local POS status: Unknown
Open OBM-POS: disabled
```

The first hypothesis to prove or disprove is therefore:

```text
the previously redeemed protected token state is missing, deleted, unreadable, selected from the wrong ProductRoot, rejected by checkpoint validation, or not resumed by startup
```

Do not state that the token was deleted until direct evidence proves deletion. A missing Phase 1 resume can also be caused by wrong ProductRoot selection, DPAPI scope/decryption failure, token rotation/cleanup, refresh-path failure, checkpoint identity mismatch, or startup ignoring valid persisted state.

## Exact priority

Before investigating Category Weight, routing, TblTenantPosDevice, CompanionApp, terminals, sync E2E, API roles, or any other architecture, answer this question:

```text
What happened to the access/bootstrap token and refresh token that were received after successful Pairing Code redemption, and why can the current WPF startup not resume them?
```

## Frozen work

Do not inspect, design, implement, or modify in this task:

```text
Service Category Weight
Booking Weight
Price Weight business/save semantics
TblTenantPosDevice writer/schema/migration
TblPosLocal/TblTenantPosDevice routing
CompanionApp or terminal modeling
API destination routing
sync happy path
API schema/roles/grants/migrations/data
POS1-POS10 PlatformAppV0 UI or Pairing Code behavior
canonical Provider runtime-sync behavior
```

Do not reset either database. Do not create a new ProductRoot, token store, authentication flow, refresh service, startup coordinator, or installation module.

## Canonical token boundary to audit

Audit the exact current source call chain:

```text
Pairing Code redeem response
-> access/bootstrap token and refresh token DTO fields
-> validation/canonicalization
-> protected machine-side persistence
-> checkpoint/metadata persistence
-> process exit/restart
-> protected read/decrypt
-> access-token validity evaluation
-> refresh-token path when access token is expired
-> bootstrap/me or protected resume call
-> Phase 1 completion/resume state
-> Phase 2/MainWindow eligibility
```

Use actual source names. Do not invent a `WpJWT` table or second token store.

## Phase 1 — Prove what redeem returned and what was written

Using the accepted prompt009–prompt011/InstallationV0 evidence and current source, record without exposing token values:

```text
REDEEM_RESPONSE_ACCESS_TOKEN_PRESENT=yes/no
REDEEM_RESPONSE_REFRESH_TOKEN_PRESENT=yes/no
ACCESS_TOKEN_FIELD_NAME=<actual source name>
REFRESH_TOKEN_FIELD_NAME=<actual source name>
ACCESS_TOKEN_EXPIRATION_METADATA=<timestamp/seconds only, no token>
REFRESH_TOKEN_EXPIRATION_METADATA=<timestamp/seconds or NOT_DEFINED>
PERSISTENCE_OWNER_CLASS=<exact class>
PERSISTENCE_METHOD=<exact method>
PROTECTED_STORE_TYPE=<DPAPI/file/existing exact mechanism>
PROTECTED_FILE_OR_RECORD_NAMES=<names only>
PRODUCT_ROOT_USED_AT_REDEEM=<exact path>
ATOMIC_WRITE_OR_CHECKPOINT_BOUNDARY=<exact method>
WRITE_RESULT_AT_REDEEM=success/failure/not-proven
```

Trace the complete successful-redeem code path to the final durable write. A successful HTTP redeem response is not sufficient proof that both credentials were persisted.

## Phase 2 — Inventory current protected state without reading secrets

For the ProductRoot used at redeem and every ProductRoot selectable by the operator-equivalent launch, record:

```text
path
configuration source
protected credential file/record exists yes/no
refresh credential file/record exists yes/no
checkpoint exists yes/no
file size greater than zero yes/no
last-write timestamp
ACL/current-user access yes/no
DPAPI scope expected
bound safe database name
current launch selects this root yes/no
```

Do not print file contents, ciphertext, tokens, GUID values, or raw identity values.

The currently observed root must be included:

```text
E:\Project2026\_dev\WpfInstallationV0Phase1\WpfProductRoot
```

## Phase 3 — Prove read/decrypt/resume behavior

Run the same protected-state reader used by production startup and record:

```text
SELECTED_PRODUCT_ROOT=<exact path>
ACCESS_TOKEN_RECORD_FOUND=yes/no
REFRESH_TOKEN_RECORD_FOUND=yes/no
CHECKPOINT_FOUND=yes/no
DPAPI_UNPROTECT_ACCESS=success/failure/not-reached
DPAPI_UNPROTECT_REFRESH=success/failure/not-reached
SANITIZED_EXCEPTION_TYPE=<exact type or none>
SANITIZED_EXCEPTION_MESSAGE=<no secret values>
CHECKPOINT_CONTRACT_VALID=yes/no
CHECKPOINT_IDENTITY_MATCH=yes/no
ACCESS_TOKEN_EXPIRED=yes/no
REFRESH_TOKEN_USABLE=yes/no/not-defined
REFRESH_PATH_INVOKED=yes/no
REFRESH_PATH_RESULT=success/failure/not-reached
PHASE1_RESUME_RESULT=success/failure/not-reached
FIRST_FAILED_METHOD=<exact class/method/line>
```

Use the same Windows user and machine context as the operator-equivalent Visual Studio launch. A test running under a different user/scope is not acceptance.

## Phase 4 — Audit every deletion, cleanup, rotation, and invalidation path

Search complete current source and recent local diffs for every operation that can remove, replace, clear, rotate, invalidate, or ignore the protected token state, including:

```text
File.Delete / directory cleanup
credential-store Clear/Delete/Reset methods
failed-resume cleanup
expiration cleanup
redeem-conflict rotation
installation restart/reset
ProductRoot cleanup
Phase 2 completion cleanup
logout/deactivation/replacement
prompt107/108 DB reset scripts
prompt109 legacy-secret cleanup
prompt117 exception recovery changes
startup fallback/profile changes
```

For each candidate provide:

```text
class/method/script
trigger
which file/record is affected
whether access token is deleted
whether refresh token is deleted
whether checkpoint is deleted
whether it executed in the operator environment
proof source/log/timestamp
```

Do not infer execution from the presence of a delete method alone.

## Required classification

Choose exactly one primary result, supported by direct evidence:

```text
T1_PROTECTED_TOKEN_STATE_DELETED
T2_NORMAL_STARTUP_SELECTED_WRONG_PRODUCTROOT
T3_PROTECTED_STATE_PRESENT_BUT_DPAPI_DECRYPTION_FAILED
T4_ACCESS_TOKEN_EXPIRED_REFRESH_TOKEN_NOT_USED
T5_TOKEN_STATE_ROTATED_OR_CLEARED_BY_EXISTING_CLEANUP
T6_CHECKPOINT_OR_IDENTITY_MISMATCH_REJECTED_VALID_TOKEN_STATE
T7_TOKEN_STATE_PRESENT_AND_VALID_BUT_STARTUP_IGNORES_IT
T8_REDEEM_RESPONSE_SUCCEEDED_BUT_DURABLE_PERSISTENCE_NEVER_COMPLETED
T9_OTHER_EXACTLY_PROVEN_TOKEN_RESUME_DEFECT
```

Do not use a generic `Phase1 incomplete` classification.

## Smallest permitted correction

Apply only the correction corresponding to the proven classification.

Examples:

```text
T1/T5: stop the incorrect deletion/cleanup at its owning boundary; do not fabricate old token values
T2: restore the existing normal profile to the same canonical ProductRoot used by redeem; do not copy protected files between roots
T3: correct the existing DPAPI/user/machine-scope usage; do not weaken protection
T4: reconnect the existing refresh path and preserve atomic credential rotation
T6: correct only the proven checkpoint canonicalization/identity comparison defect
T7: restore the existing protected-state resume call in startup
T8: complete the existing atomic persistence/checkpoint write after successful redeem
```

Rules:

```text
no manual token entry
no hardcoded token
no copying ciphertext between ProductRoots
no new authentication endpoint
no new token store
no Firebase fallback
no WpfJwt authorization weakening
no automatic new Pairing Code/redeem merely to hide the persistence defect
no manual completion-marker write
```

If the old protected credentials are irrecoverably absent and no valid refresh credential remains, keep InstallationV0 alive and report the exact existing operator recovery action. Do not automatically re-pair.

## Local-first distinction

After the token-resume defect is proven, also distinguish:

```text
A. Phase 2/local activation was already complete
   -> missing/expired remote token must not route normal startup back to InstallationV0; MainWindow should open local-first with cloud/sync degraded

B. Phase 2 genuinely requires the still-valid Phase 1 credential to complete
   -> resume through the existing protected access/refresh-token boundary; do not fabricate completion
```

Do not conflate remote bootstrap-token health with normal runtime-sync Provider credentials or PostgreSQL credentials.

## Physical acceptance

Use visible label:

```text
prompt118
```

PASS requires direct operator-equivalent proof that:

```text
redeem token/refresh-token persistence owner is identified
current protected state existence is known
exact deletion/read/decrypt/refresh/resume result is proven
normal launch selects the correct existing ProductRoot
valid persisted token state resumes without a new redeem
or, when access token expired, the existing refresh path rotates it safely
no token/checkpoint is deleted by a recoverable API/bootstrap outage
no unhandled exception/process exit occurs
when local installed state is complete, MainWindow opens directly and remains local-first
```

If protected state is irrecoverably missing, return a narrow blocker with the exact deletion/missing boundary and keep the process alive in recoverable InstallationV0. Do not claim MainWindow restored.

## Evidence additions

Add these files to the prompt118 private artifact:

```text
REDEEM_TOKEN_RESPONSE_CONTRACT.md
TOKEN_PERSISTENCE_CALL_CHAIN.md
PROTECTED_STATE_INVENTORY.md
TOKEN_READ_DECRYPT_PROOF.md
REFRESH_TOKEN_RESUME_PROOF.md
TOKEN_DELETE_ROTATION_CALL_SITES.md
TOKEN_STATE_TIMELINE.md
TOKEN_RESUME_CLASSIFICATION.md
TOKEN_PERSISTENCE_MINIMAL_DIFF.patch
```

## Public report additions

`report/report118.md` must include:

```text
Redeem access/bootstrap token returned yes/no
Redeem refresh token returned yes/no
Durable token persistence proven yes/no
ProductRoot used at redeem
ProductRoot used at current startup
Same ProductRoot yes/no
Access-token protected record exists yes/no
Refresh-token protected record exists yes/no
Checkpoint exists yes/no
DPAPI access-token read result
DPAPI refresh-token read result
Access token expired yes/no
Refresh token usable yes/no/not-defined
Refresh path invoked yes/no
Phase1 resume result
Deletion/cleanup path executed yes/no
Primary classification T1-T9
Exact failed class/method/line
Token persistence/resume correction applied yes/no
New redeem required yes/no and exact reason
Process crash/exit after fix yes/no
MainWindow opens directly yes/no
Operator MainWindow screenshot ready true/false
Manual POS1 test ready false
```

## Verdicts

PASS when token persistence/resume and normal startup are physically restored:

```text
OBM_WPF_PROTECTED_TOKEN_RESUME_AND_NORMAL_MAINWINDOW_STARTUP_PHYSICALLY_RESTORED_READY_FOR_OPERATOR_SCREENSHOT
```

Narrow blockers:

```text
BLOCKED_WPF_ACCESS_TOKEN_STATE_DELETED
BLOCKED_WPF_REFRESH_TOKEN_STATE_DELETED
BLOCKED_WPF_PROTECTED_STATE_WRONG_PRODUCTROOT
BLOCKED_WPF_PROTECTED_STATE_DPAPI_READ
BLOCKED_WPF_REFRESH_PATH
BLOCKED_WPF_CHECKPOINT_IDENTITY_MISMATCH
BLOCKED_WPF_REDEEM_PERSISTENCE
BLOCKED_WPF_TOKEN_RESUME_PHYSICAL_PROOF
```

Do not create prompt119 before report118 is reviewed.
