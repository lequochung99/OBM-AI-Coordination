# Report 070 - Transaction Calculation Null Policy Public Summary

## 1. Verdict

`OBM_POS_TRANSACTION_CALCULATION_POLICY_REPAIR_READY_FOR_OPERATOR_APPLY`

The transaction calculation null-policy defect is source-corrected, but the local target environment still requires an operator-authorized policy setup repair before physical acceptance.

## 2. Root-cause classification

`REQUIRED_POLICY_SEED_MISSING`

The calculation path reached the legacy turn calculation with a valid transaction line item but without a resolved policy. The missing policy came from the active-policy lookup returning no policy row in `<LOCAL_TARGET_DB>`.

## 3. Policy contract

The policy is required.

No evidence proved that a missing policy is equivalent to a neutral/default policy, so the source fix does not silently create or substitute a fallback policy.

## 4. Required follow-up

Two actions are required before final physical acceptance:

- Source fix: implemented locally in repository-relative turn calculation code so missing policy becomes an explicit safe domain failure instead of a raw null dereference.
- Operator-authorized DB repair: still required for `<LOCAL_TARGET_DB>` to restore the required policy setup row(s). This report does not include or authorize the repair script.

## 5. Build/test evidence

Build:

```text
dotnet build <WPF_ROOT>/<WPF_PROJECT> -v minimal
```

Result: `PASS`

- Errors: `0`
- Warnings: `174`

Filtered regression tests:

```text
dotnet test <WPF_TEST_ROOT>/<WPF_TEST_PROJECT> --filter "<transaction/turn/factor/calculation/invoice/checkout/policy filter>" -v minimal
```

Result: `PASS`

- Failed: `0`
- Passed: `118`
- Skipped: `0`
- Total: `118`

## 6. Safe physical retest steps

1. Apply only an operator-approved repair that restores the required transaction calculation policy setup in `<LOCAL_TARGET_DB>`.
2. Launch the WPF POS manually.
3. Recreate the same transaction path that previously exposed the null policy.
4. Attempt checkout/payment once.
5. Verify no raw `NullReferenceException` appears.
6. If the policy is still absent, verify the transaction is blocked by the safe missing-policy setup failure.
7. Verify the failed attempt does not partially commit invoice, output, payment, turn, or outbox state.
8. After policy repair, retry once and verify one successful transaction with no duplicate rows.

## 7. No-mutation/no-secret confirmation

- No database mutation was performed by this task.
- No automated checkout/payment transaction was run.
- No WPF process was launched or clicked automatically.
- No API token, employee PIN, DB role, password, or connection secret was changed or printed.
- No customer, employee, service, price, raw identifier, GUID, connection string, machine path, Windows username, or business-row detail is included in this public report.
- The detailed report and evidence remain local only under artifact version `NullPolicyV001`.

## 8. Local evidence artifact

Detailed local evidence artifact version: `NullPolicyV001`

Public SHA-256 manifest:

```text
E875B44CA214FF032F25C088E1263CB623AFD144502607770446BD9470AF4ED3  physical-call-chain.mmd
7B7F76C40D4EAD1CB5360E57E8C548661C7A36D76581AFA9888E93CA96AF3BDF  policy-resolution-flow-before-after.mmd
70BB25D44AFC78D50CD779AAC6AD6042621E5C420B586170D85A655DA1EA2E32  policy-source-inventory.csv
12AB8B985104BF402FC96BE15A68E5BC0E5A27CA10DF9630F286B5424CAFE8E9  README.md
55F3108DEA873531C62ED1DAC92311B22FEC8B5570D9E5A673EF5FBA9A3686B0  safe-db-policy-counts.json
ABBAF2D2BE2CCDF6E82BE65B6DBBE70D848E6659898B7896454CF3180493C244  test-results.txt
FD41E1D84E5DD55E7765DF7899A007A1DDC8930F232ABA22468F1BED9EFF71C0  transaction-atomicity-proof.md
```

## 9. Coordination commit SHA

This public redacted report is committed and pushed in the coordination repository. The exact commit SHA is returned by Codex after push.
