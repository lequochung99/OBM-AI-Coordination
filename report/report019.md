# Report 019 - POS1 Trial Seed V001 Using Full Default Setting Rows

## 1. Verdict

BLOCKED_PHASE2_POS1_TRIAL_TARGET_SAFETY

Implementation work was completed far enough to build and pass focused tests, but the physical POS1 database mutation was not executed because the prompt did not provide an exact approved POS1 target database and rollback anchor. I did not infer a database target, did not mutate PostgreSQL, and did not touch any reference database.

## 2. Prompt018 Superseded

Confirmed. Prompt018 was superseded and was not executed as written.

## 3. POS1 Test Target Classification And Rollback Anchor

Physical POS1 target classification: blocked before mutation.

Reason:

- no exact approved target DB name was supplied in prompt019;
- no rollback anchor was supplied or verified;
- protected/reference exclusions could not be proven for a concrete target.

Result:

- no PostgreSQL seed was run;
- no reference DB mutation occurred;
- no rollback anchor was created or changed.

## 4. Selected Tables And Template Row Counts

Template version:

```text
phase2-pos1-trial-seed-v001
```

Selected tables:

```text
TblTenant: 1
TblPosLocal: 1
TblSetupWeird: 1
TblSetupServicesMethod: 1
TblSetupLoginMethod: 3
TblSetupPaymentMethod: 6
TblEmployeePermission: 3
TblParameterSetting: 2
TblSetupPrinter: 5
Phase2TrialCompletionMarker: 1
```

Total template rows: 24.

## 5. Full Canonical Printer Default Row/Type List

The v001 template inserts the complete current logical printer default set:

```text
CUSTOMER_TERMINAL
MERCHANT_TERMINAL
POS_CUSTOMER
POS_MERCHANT
POS_EMPLOYEE
```

These rows preserve logical printer purposes. They are not treated as a requirement for five physical printers.

## 6. Printer Column Handling

Preserved:

- `PrinterType`
- `PrinterName`
- `IsPrinterActived`
- `IsDeleted`
- logical notices

Replaced:

- `TenantGuid` from Phase 1 identity

Remapped/generated:

- `SetupPrinterGuid` deterministically from version + tenant + POS + table + stable key

Cleared:

- `PrinterPath1`
- `PrinterPath2`

Derived:

- `CreatedAt`
- `UpdatedAt`

Not applicable:

- `PosGuid` is not present on current `TblSetupPrinter`.

## 7. Dependency Order

```text
TblTenant
TblPosLocal
TblSetupWeird
TblSetupServicesMethod
TblSetupLoginMethod
TblSetupPaymentMethod
TblEmployeePermission
TblParameterSetting
TblSetupPrinter
Phase2TrialCompletionMarker
```

## 8. Generic Transformer Design

Implemented one versioned template dataset and one identity/value transformer:

- template rows define table, stable key, and logical values;
- transformer replaces Phase 1 identity fields;
- transformer generates deterministic row GUIDs;
- transformer derives timestamps;
- environment/private paths are cleared in the template;
- template row counts are authoritative from the dataset, not duplicated in production code.

## 9. One-Transaction Proof

Implemented a PostgreSQL transaction script builder that emits one transaction shape:

```text
BEGIN
  revalidate Phase 1 authorization/identity
  verify POS1 target and rollback anchor
  acquire tenant/POS/version advisory transaction lock
  insert-or-verify rows in dependency order
  verify operational/runtime deltas
  write completion marker last
COMMIT
```

Focused test `Phase2V001PostgreSqlScript_UsesOneTransactionAndMarkerLast` passed.

## 10. Completion-Marker-Last Proof

Completion marker table:

```text
Phase2TrialCompletionMarker
```

Stable key:

```text
phase2-pos1-trial-seed-v001:complete
```

Focused tests proved the marker is the final ordered row.

## 11. Idempotent Replay Proof

Focused in-memory transaction executor test passed:

- first execution inserts/verifies the template plan;
- replay with the same version, identity, and stable keys produces the same row counts;
- no duplicate logical rows are created by stable-key replay.

## 12. Outbox Delta Proof

Focused tests prove:

```text
TblLocalOutbox delta = 0
```

No outbox rows are part of v001.

## 13. Excluded-Table Delta Proof

Focused tests prove the v001 template excludes:

```text
TblEmployee
TblService
TblCustomer
TblInvoice
TblLocalOutbox
TblGiftCard
TblBooking
```

The implementation does include `TblEmployeePermission` roles as requested: `Owner`, `Admin`, `Sub_Manager`.

## 14. Exact Source Files Changed

Files touched for prompt019:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TrialConstants.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TrialIdentity.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TemplateRow.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TransformedRow.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TrialTemplateV001.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2IdentityTransformer.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TrialPlan.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TrialPlanBuilder.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TrialVerificationResult.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\IPhase2TrialTransactionExecutor.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\InMemoryPhase2TrialTransactionExecutor.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2PostgreSqlTransactionScriptBuilder.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\CURRENT.md
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\v001\README.md
E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs
```

Note: the OBM source worktree already had many unrelated dirty/untracked files before this prompt. I did not revert or clean them.

## 15. Build/Test Commands And Counts

Build:

```text
dotnet build InstallationV0\InstallationV0.csproj
```

Result:

```text
Build succeeded.
0 Warning(s)
0 Error(s)
```

Focused tests:

```text
dotnet test ..\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter FullyQualifiedName~InstallationV0Phase1Tests
```

Result:

```text
Passed: 34
Failed: 0
Skipped: 0
```

The broader test project emitted existing warnings outside the focused prompt019 implementation scope.

## 16. POS1 Physical DB Result And WPF Handoff

POS1 physical DB seed: not executed.

Reason:

```text
BLOCKED_PHASE2_POS1_TRIAL_TARGET_SAFETY
```

No WPF launch was performed for this prompt, because there is no seeded POS1 physical DB result to hand off.

## 17. Prompt019 Label Proof

Implemented visible label:

```text
prompt019
```

Expected title source:

```text
OBM InstallationV0 Phase 1/2 - prompt019
```

Focused label test passed.

## 18. No Reference DB Mutation / No Secret Leakage

Confirmed:

- no reference DB mutation;
- no PostgreSQL mutation;
- no credential, token, connection string, password, or secret printed;
- no DB fallback credential used.

## 19. Next Observed Missing Table/Row/Default

No WPF physical run was performed, so no runtime missing row/table/default was observed.

The next required action is to provide or approve a concrete POS1 test target and rollback anchor, then run the transaction against that target only.

## 20. Coordination Commit SHA

Pending at report creation time.
