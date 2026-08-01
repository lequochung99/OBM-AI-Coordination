# Report 061 — Phase 2 V003 Usable Employee Operational PIN Seed

## 1. Verdict

`BLOCKED_PHASE2_V003_PIN_SEED_BUILD_OR_TEST`

Phase 2 V003 usable employee operational PIN seed source work is implemented for a future clean disposable retest. The required prompt061 test command remains blocked by one unrelated Payroll API migration artifact path pulled into the broad filter.

## 2. Original V001 gate evidence

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V001
CanonicalDocSha256=7044A02F29FE349FE531DE6800BA739E6B29EA473B9A867881B283DB8743BC72
```

## 3. Canonical V001 preservation path/hash

```text
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V001\INSTALLATION_RUNTIME_CANONICAL_V001.md
SHA256=7044A02F29FE349FE531DE6800BA739E6B29EA473B9A867881B283DB8743BC72
```

Current docs were preserved before replacement:

```text
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V006\CURRENT_TASK_V006.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V006\CURRENT_RESULT_V006.md
```

## 4. Canonical V002 created-before-code proof and final hash

```text
DOCS_UPDATED_BEFORE_CODE=PASS
CanonicalDocVersion=V002
CanonicalDocSha256=916AE7CE71ADA14D768DE1D11E7E5F3604416E8176AA3C59479353481AA2DFB2
```

Active canonical/current docs terminology guard scan: PASS.

## 5. Exact EmployeeType audit and Staff classification

Audited source:

```text
E:\Project2026\4POS\NailSalonNet8\Enums\EmployeeType.cs
```

Findings:

- `RealEmployee = 1`
- `VirtualAnyTechnician = 2`
- Phase 2 V003 applies operational PIN rules to real/login-capable employee seed rows.
- Staff classification uses `PermissionName = Staff`.
- Non-Staff real/login-capable rows are every other real employee permission.

## 6. Current seed implementation/placeholder root cause

Root cause: V002 treated `TblEmployee.LoginNumber` as an employee private field and reset it through the placeholder path. V003 removes `LoginNumber` from that private reset set and routes it through an operational PIN policy helper.

## 7. V003 seed version and marker convention

```text
Version=phase2-reference-driven-trial-v003-usable-employee-pins
PreviousEmployeeVersion=phase2-reference-driven-trial-v002-employees
CompletionMarkerStableKey=phase2-reference-driven-trial-v003-usable-employee-pins:complete
RequiredPriorVersion=phase2-reference-driven-trial-v001
```

## 8. PIN preservation and generation algorithm

Implemented in:

```text
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2EmployeeOperationalPinPolicy.cs
```

Behavior:

- Preserve only valid unique numeric source values with exact required length.
- Staff policy: exact 4 digits.
- Non-Staff policy: exact 5 digits, leading zero formatting allowed.
- Invalid/missing/placeholder/wrong-length/duplicate values are generated with cryptographic RNG.
- No raw operational PIN values are written to report/evidence.

## 9. Tenant-scoped uniqueness behavior

Implemented before V003 marker insertion:

- Staff shape count.
- Non-Staff shape count.
- Duplicate count within TenantGuid.
- Explicit failure result code if uniqueness fails.

## 10. Placeholder elimination proof

Source proof:

- `LoginNumber` removed from `EmployeePrivateColumnNames`.
- `ApplyEmployeeOperationalPins(transformedRows)` is called during V003 target row construction.
- `AssertEmployeeOperationalPinsAsync` runs before marker insertion.

## 11. Transaction/idempotency proof

V003 reuses the existing single-transaction PostgreSQL executor and advisory lock. Existing employee rows are adopted instead of reinserted; outbox rows and markers are checked before insert. Startup hydration distinguishes V001, V002, and V003 markers.

## 12. Existing-target duplicate behavior

The V003 marker gate fails explicitly with `PHASE2_V003_EMPLOYEE_PIN_TENANT_UNIQUENESS_FAILED` if duplicate eligible PINs exist in the target tenant.

## 13. Current physical DB non-mutation proof

Prompt061 prohibited mutating `obm_pos_dev_v0_pg`. The V003 seed was not run physically against the current operator DB. WPF was not launched.

## 14. Raw PIN secrecy/reporting proof

No raw generated or real operational PIN values are included in this report or evidence files. Evidence uses counts, booleans, hashes, and policy names only.

## 15. Exact source/docs/tests files changed

Prompt061-touched OBM files:

```text
E:\Project2026\4POS\NailSalonNet8\AGENTS.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V001\INSTALLATION_RUNTIME_CANONICAL_V001.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V006\CURRENT_TASK_V006.md
E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\history\V006\CURRENT_RESULT_V006.md
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Application\InstallationV0BuildInfo.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2EmployeeOperationalPinPolicy.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2ReferenceDrivenManifest.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2StartupHydrationService.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Phase2TrialConstants.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\PostgreSqlPhase2ReferenceSeedExecutor.cs
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\CURRENT.md
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Phase2\Trials\reference-driven-v003-usable-employee-pins\README.md
E:\Project2026\4POS\NailSalonNet8\InstallationV0\Presentation\InstallationV0Window.cs
E:\Project2026\4POS\NailSalonNet8.Tests\InstallationV0\InstallationV0Phase1Tests.cs
E:\Project2026\4POS\NailSalonNet8.Tests\Wiring\CanonicalRuntimeWiringTests.cs
```

## 16. Build/test commands and counts

```text
dotnet build E:\Project2026\4POS\NailSalonNet8\InstallationV0\InstallationV0.csproj -v minimal
Result: PASS, 0 warnings, 0 errors

dotnet build E:\Project2026\4POS\NailSalonNet8\NailSalonNet8.csproj -v minimal
Result: PASS, existing warnings, 0 errors

dotnet test E:\Project2026\4POS\NailSalonNet8.Tests\NailSalonNet8.Tests.csproj --filter "FullyQualifiedName~InstallationV0|FullyQualifiedName~Phase2|FullyQualifiedName~Seed|FullyQualifiedName~Employee|FullyQualifiedName~LoginNumber|FullyQualifiedName~Pin|FullyQualifiedName~Documentation|FullyQualifiedName~Naming" -v minimal
Result: FAIL, 184 passed, 1 failed, 0 skipped
Failure: E:\Project2026\1ApiServer\ApiServer01\SpacePos.Provisioning.Schema\Migrations\011_payroll_three_table_data_foundation.sql missing
```

Focused InstallationV0/Phase2/PIN subset after V003 assertion fixes:

```text
Result: FAIL, 84 passed, 1 failed, 0 skipped
Remaining failure: same unrelated Payroll API migration artifact path.
```

## 17. Evidence folder and hashes

```text
E:\Project2026\RecoveryReports\InstallationV0\Phase2UsablePinSeedV003
```

`SHA256SUMS.txt` was generated in the evidence folder.

## 18. Current canonical/current-task/current-result hashes

```text
INSTALLATION_RUNTIME_CANONICAL.md=916AE7CE71ADA14D768DE1D11E7E5F3604416E8176AA3C59479353481AA2DFB2
CURRENT_TASK.md=6F9C195D7CF502855F66022AEAA2DD8F5D6615720238153FCD6BEA8B33512986
CURRENT_RESULT.md=50E9A5A7196CB3279B546A77A85A870269E53B4A13D12F122EC91589AC2E98B4
AGENTS.md=2D5F9C90CFEB53BA6486791E34A37515C7A31BA8A25F79A258BA4DEA8753426B
```

## 19. Build-label decision

Changed because InstallationV0 source/UI changed:

```text
Build label: prompt061
Window title: OBM InstallationV0 Phase 1/2 - prompt061
```

## 20. Exact next clean disposable V003 seed retest steps

1. Resolve the missing Payroll API migration artifact path or approve a narrower prompt061 test command.
2. Rerun the exact prompt061 build/test sequence.
3. Prepare a clean disposable Phase 2 target database, not `obm_pos_dev_v0_pg`.
4. Run V003 seed once.
5. Verify V003 marker count = 1.
6. Verify placeholder count = 0 by count only.
7. Verify Staff 4-digit count/shape and Non-Staff 5-digit count/shape.
8. Verify tenant duplicate count = 0.
9. Rerun V003 and prove no marker/outbox duplication and no PIN rotation without printing raw PIN values.

## 21. Coordination commit SHA

Recorded in final response after commit/push.

