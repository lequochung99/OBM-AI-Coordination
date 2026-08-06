# report144 — Setup Cost/Fee Merchant Canonical Initial Seed

## Verdict

`WPF_SETUP_COST_FEE_MERCHANT_CANONICAL_INITIAL_SEED_PASS`

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V005
CanonicalDocSha256=11841E618E461A40398BCCF4302ABC4E9E6BB67C035AD9607433C1A3887C07C3
```

Note: AGENTS.md pointers `INSTALLATION_RUNTIME_CANONICAL.md` / `CURRENT_TASK.md` / `CURRENT_RESULT.md` are absent at the repo root of `docs/refactoryInstallation/`; V005 history file was read as the current canonical authority.

## Task and source state

| Field | Value |
|---|---|
| Prompt | `prompt/prompt144.md` (local sequence; task packet `TASK-SETUP-COST-FEE-MERCHANT-SEED-V001`) |
| Executor | Cursor |
| Source repositories | `4POS/NailSalonNet8`, `OBM-AI-Coordination` |
| Starting commit (WPF) | `d7bd017786eb64ec686fd79abc5f7971bb05c3f2` |
| Ending commit (WPF) | uncommitted local changes on same HEAD (awaiting operator commit request) |
| Branch | `recovery/wpf-installation-reset-cursor-v001` |
| Source DB | `enailsalon_phasee1_pos1_pg` (READ-ONLY) |
| Target DB | `obm_pos_dev_v012_pg` |

## Reused context

- `context/database/TblSetupCostAndFeeMerchant/CURRENT.md` (was UNVERIFIED)
- `graphify/CURRENT.md` (NOT_REGISTERED; used live WPF graphify-out instead)
- Graphify queries persisted under `graphify/wpf/current/queries/SetupCostFeeMerchant/`
- Repeated investigation: `NONE`

## Root cause / finding

Operator lane had empty `TblSetupCostAndFeeMerchant`. Canonical spelling is **Fee** (entity/table/DbSet/DTO). **Free** appears only in mapper method names. Source DB held one safe configuration row (`ClientFeeInPercent=2`, amounts 0, `IsActived=false`, name `????`). InstallationV0 `BaselineSeeder` / `InitialSeedBootstrapOutbox` did not include this table; legacy `SeedDbProvider` had zeros. Canonical seed + bootstrap outbox wiring were added without a second seeder.

## Changes

| File or component | Change | Reason |
|---|---|---|
| `CostAndFeeMerchantSeedContract.cs` | Added | Canonical safe defaults + presence validation |
| `BaselineSeeder.cs` | Ensure + setup-group verify | Canonical initial seed |
| `InitialSeedBootstrapOutbox.cs` | Entity + stage + counts | Existing sync contract |
| `V007SeedProgress.cs` | Ordered/Display registration | Discovery/allowlist alignment |
| `MinimalBaselineCatalog.cs` | Stable key | Catalog consistency |
| `SeedDbProvider.cs` | Use contract values | Legacy demo seed alignment |
| Tests | New + count updates | Focused proof |

## Protected invariants

- Source DB `enailsalon_phasee1_pos1_pg` unchanged (same PK `f192c5b8-8a95-419b-a2e2-a8775b4374be`, values unchanged)
- No DB reset/drop/recreate
- Operator-preserving seed (existing non-deleted TenantGuid row is not overwritten)
- No secrets committed
- No UI verification performed (operator owns UI)

## Validation

| Check | Result | Evidence |
|---|---|---|
| Main WPF build | PASS | `dotnet build NailSalonNet8.csproj` succeeded (0 errors) after releasing DLL lock |
| Focused tests | PASS | 33 passed / 0 failed (`CostAndFeeMerchant*`, `InitialSeedBootstrapOutbox*`, `CashDrawer*`, `V007EmployeeRoster*`) |
| Empty/missing key seed | PASS | target pre=0 → post=1 canonical row |
| Idempotent rerun | PASS | second insert `inserted=0`, still 1 row |
| Existing row preserve | PASS | Ensure presence-guard; contract tests cover operator custom row presence |
| Source mutation count | PASS | `0` |
| Secret-copy audit | PASS | fee numeric/bool/name only |
| Outbox sync | PASS (wired) / live StageAndValidate blocked by pre-existing unexpected outbox | Contract tests + CostFee-only outbox 0→1 (`I`, Sent=0, ExpectedEventCount=1); full StageAndValidate returned `BLOCKED_INSTALLATION_CREATED_OUTBOX` (213 unexpected EntityTypes pre-existing) |
| Unrelated DB reset | PASS | none |

## Spelling / values used

- Canonical: `TblSetupCostAndFeeMerchant`
- Sanitized seed: `NameCostAndFee=????`, client amount `0`, client percent `2`, bank amount/percent `0`, `IsActived=false`

## Manual operator verification

```text
MANUAL_UI_CHECK_REQUIRED
Screen/runtime: Settings merchant terminal / customer transaction fee policy (operator-owned)
Steps:
1. Restart POS against obm_pos_dev_v012_pg.
2. Open fee policy settings UI.
3. Confirm client percent shows 2.00 and values load even if inactive flag is false.
4. Change a value, save, restart, confirm operator edit preserved (no overwrite on resume seed).
Expected result: defaults present when empty; operator edits preserved.
Control case: source DB enailsalon_phasee1_pos1_pg remains untouched.
Observed result: NOT RUN BY AI
```

## Reusable knowledge produced

- Module/DB context: `context/database/TblSetupCostAndFeeMerchant/V001/DB_CONTEXT.md`
- Graphify sanitized queries: `graphify/wpf/current/queries/SetupCostFeeMerchant/`
- Investigation index: updated one-line reference
- Decision record: NONE

## Known limitations and follow-up

- Live `StageAndValidateAsync` on `obm_pos_dev_v012_pg` is blocked by pre-existing unexpected outbox EntityTypes (hygiene/cleanup task, out of scope).
- `NameCostAndFee="????"` retained as literal source evidence; rename only with operator approval.
- WPF source changes not committed (awaiting explicit commit request).
