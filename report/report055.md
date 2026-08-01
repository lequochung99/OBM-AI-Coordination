# Report 055 - Canonical Installation/Runtime V001 and Code Gate

## 1. Verdict

OBM_POS_CANONICAL_V001_AND_CODE_GATE_READY

## 2. DOCS_READ_BEFORE_CODE_GATE Evidence and Read Order

Gate result:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V001
PreviousCanonicalVersion=None
CanonicalCreationMode=FirstVersion
```

Read order completed before documentation creation:

1. `report/report044.md`
2. `report/report045.md`
3. `report/report049.md`
4. `report/report050.md`
5. `report/report051.md`
6. `report/report052.md`
7. `report/report053.md`
8. `report/report054.md`
9. `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\README.md`
10. `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\SHA256SUMS.txt`
11. `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\installation-flow-current.mmd`
12. `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\installation-flow-target.mmd`
13. `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\installation-flow-subgraph.json`
14. `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\edge-inventory.csv`
15. `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\symbol-action-inventory.csv`
16. `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\redundant-link-candidates.md`
17. `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\dynamic-edge-checklist.md`
18. `E:\Project2026\RecoveryReports\InstallationV0\DependencyGraphAuditV001\aspnet-identity-trace-audit.md`
19. `E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md`
20. `E:\Project2026\4POS\NailSalonNet8\docs\wpfsummary.md`

Pre-creation target check:

- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation` did not exist.
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md` did not exist.
- No previous canonical file was archived or faked.

## 3. Graphify / Audit Evidence Hashes Used

- `report/report054.md`: `7479E27DCE6D88F8FAA2EBA4245991E56F3E112BE74435D65F18B68BCB5A5A27`
- `DependencyGraphAuditV001\SHA256SUMS.txt`: `9B573B5B3CAF760C88FD3FA64640FDEE372593BB7D0D31D42D4B18C36F70B786`
- Graphify version from report054: `graphify 0.9.26`
- Graphify classification from report054: `GRAPHIFY_AUDIT_PASS`

## 4. First-Version Confirmation

This run created the first canonical V001 at the approved path. There is no fake V000/V001 history file and no `history/V001` copy.

`CanonicalCreationMode=FirstVersion`

## 5. Files / Folders Created

Created folder:

- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation`

Created current documentation/instruction files:

- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md`
- `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\README.md`
- `E:\Project2026\4POS\NailSalonNet8\AGENTS.md`

Documentation-only status notice added:

- `E:\Project2026\4POS\NailSalonNet8\docs\wpfsummary.md`

## 6. Canonical V001 Outline and Authority Statement

`INSTALLATION_RUNTIME_CANONICAL.md` declares itself as:

- Title: `OBM-POS Installation and Runtime Canonical Contract`
- Version: `V001`
- Status: `Current Canonical Authority`
- Evidence basis: `report044-report054 and DependencyGraphAuditV001`

It states the canonical normal-runtime rule:

```text
Local PostgreSQL usable
-> open MainWindow
-> initialize API session afterward

API token valid
-> Online

API token expired / HTTP 401 / API unavailable
-> MainWindow remains open
-> local CRUD continues
-> API is Offline or Reauthorization Required
```

It also documents:

- local PostgreSQL credential boundary;
- `hung` as the normal runtime PostgreSQL username for the V0 development database;
- employee `TblEmployee.LoginNumber` as operational PIN, not password;
- Staff PIN exactly 4 digits and Non-Staff PIN exactly 6 digits as approved policy;
- API session after MainWindow only;
- Phase 1/Phase 2 proof as installation/verification only;
- ASP.NET Identity traces not proven as ordinary WPF pre-MainWindow dependencies.

## 7. Mermaid Diagram Validation

The canonical file contains three complete Mermaid blocks:

- Normal runtime.
- Installation versus runtime.
- Naming cleanup target.

Fence verification found matching opening/closing fences for:

- four `text` blocks;
- three `mermaid` blocks.

## 8. AGENTS.md Mandatory Gate Rules

`E:\Project2026\4POS\NailSalonNet8\AGENTS.md` was created as an instruction pointer, not a competing architecture authority.

It requires reading:

- `docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md`
- `docs\refactoryInstallation\CURRENT_TASK.md`
- `docs\refactoryInstallation\CURRENT_RESULT.md`

before AI/Codex tasks that change startup, installation, DB connectivity, runtime DB credential source, API auth/session, employee operational PIN, runtime state, MainWindow routing, service/type naming in these flows, or ASP.NET Identity cleanup.

It requires future implementation reports to include:

```text
DOCS_READ_BEFORE_CODE_GATE=PASS
CanonicalDocVersion=V001
CanonicalDocSha256=<actual hash>
```

## 9. CURRENT_TASK and CURRENT_RESULT Contents / Intent

`CURRENT_TASK.md` sets the next approved implementation task:

```text
Service/type/file naming cleanup based on DependencyGraphAuditV001 and canonical V001.
```

Required intent:

- remove compatibility aliases where no external contract exists;
- rename physical files to match public types;
- update DI registrations and tests;
- remove generic BootstrapRepair wording from active normal runtime;
- keep behavior unchanged;
- no DB/API/PIN contract changes;
- physical MainWindow retest after cleanup.

`CURRENT_RESULT.md` records:

- DependencyGraphAuditV001 completed;
- Canonical V001 created;
- AGENTS.md gate created;
- source naming cleanup not yet performed;
- no DB mutation;
- no WPF launch;
- next task is naming cleanup.

## 10. Older-Document Status Notices Added or Limitations

Notice added safely to:

- `E:\Project2026\4POS\NailSalonNet8\docs\wpfsummary.md`

Notice not added to:

- `E:\Project2026\CanonicalInstallationDocs\WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md`

Reason:

That file was already modified in the OBM worktree before this prompt. To avoid mixing prompt055 documentation-only edits with pre-existing user/source changes, it was left unchanged and recorded as a limitation.

The new canonical authority is still established through:

- `INSTALLATION_RUNTIME_CANONICAL.md` V001;
- `README.md`;
- `CURRENT_TASK.md`;
- `CURRENT_RESULT.md`;
- `AGENTS.md`.

## 11. Final SHA-256 Table for Current Docs

| File | SHA-256 |
| --- | --- |
| `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL.md` | `A70BE08FE143A34775A1F71844B7A6B672DDD28C24CB041B841E0604585B3915` |
| `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_TASK.md` | `2871956A986F46E96F2C00BF0FAD7B0DE6DCEA02EEC0871159BDB52AED33A652` |
| `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\CURRENT_RESULT.md` | `7E3093B8E9F4E59B55705D564C71BCEF73F363D5044A792927E366086E3B95F0` |
| `E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\README.md` | `47257AD5E6201D2E54F25FBAC283F0150CDEB8DE2965A1090008F272979996A4` |
| `E:\Project2026\4POS\NailSalonNet8\AGENTS.md` | `20CB5060FC42ADDA2DE5EEBF90F4630BFBFCAA9ED195615E0B5B8DA035905114` |

## 12. Proof No Implementation Source / Tests / Config Changed

Prompt055 intentionally changed documentation/instruction files only.

Prompt055-touched OBM paths:

- `4POS/NailSalonNet8/docs/refactoryInstallation/`
- `4POS/NailSalonNet8/AGENTS.md`
- `4POS/NailSalonNet8/docs/wpfsummary.md`

The existing dirty status of `CanonicalInstallationDocs/WPF-INSTALLATION-V0-TWO-PHASE-CONTRACT.md` predated this prompt and was not modified by this run.

No `*.cs`, `*.xaml`, `*.csproj`, `*.sln`, `launchSettings.json`, DI registration, or test file was edited by this prompt.

No build or implementation test was run because this prompt was documentation-only.

## 13. Proof No DB / API / PIN / Process Mutation

No PostgreSQL connection, migration, seed, role/password change, GRANT/REVOKE, Pairing Code redeem, API token mutation, employee PIN mutation, environment-variable write, WPF launch, API launch, process stop, or process restart was performed.

No password, connection string, token, Pairing Code, protected credential, or raw employee PIN was printed.

## 14. Coordination Commit SHA

This report is committed and pushed as `report/report055.md`.

The final pushed coordination commit SHA is returned by Codex after commit and push. Embedding the commit's own SHA inside this file before commit would change the commit hash.
