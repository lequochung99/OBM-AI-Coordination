# OBM AI Coordination Constitution V001

## 1. Authority

The operator owns product intent, UI acceptance, production actions, destructive approvals, and the final decision to accept or reject a task.

Source code, database state, tests, runtime evidence, and operator physical verification outrank AI summaries.

## 2. Core engineering invariants

- OBM-POS remains local-first.
- Domain save and canonical `TblLocalOutbox` creation occur in the approved transaction boundary.
- Existing periodic upload, API sync, API commit, and SignalR notification flows must be reused; do not create duplicate sync lanes.
- Protected data, production environments, payment behavior, migration, installation, authentication, and security require elevated verification.
- No agent may claim physical or UI PASS without operator evidence.

## 3. Task contract

One numbered prompt defines one complete task whenever practical. Every task declares:

- goal and business acceptance;
- task type and risk level;
- recommended executor and reviewer;
- in-scope and protected areas;
- reusable context references;
- investigation limits;
- implementation requirements;
- focused validation;
- manual operator checks;
- report and reusable-context update requirements.

## 4. Risk levels

- **L0 — Trivial:** text, labels, documentation, cosmetic changes.
- **L1 — Local:** isolated UI, validation, localized CRUD, compile repair.
- **L2 — Cross-component:** WPF/API, booking availability, outbox mapping, shared contracts.
- **L3 — Critical:** payment, checkout, tip, receipt, migration, installation, security, production data, canonical sync.

L0-L1 normally use one executor and focused validation. L2 normally uses one executor plus diff-focused review. L3 requires a strong executor, independent review, and physical evidence where applicable.

## 5. Agent assignment

Model choice is based on total expected cost, including retries and operator time.

- Cursor is the default interactive engineer for local and medium-scope work.
- Codex 5.6 SOL Medium or stronger is the minimum preferred Codex tier for hard implementation.
- Cheaper models are optional for read-only investigation only when their output prevents meaningful duplicate work.
- Do not add an agent merely to increase the number of reviewers.

## 6. Reusable knowledge

Investigation is an asset. Findings must be written with source commit, inspected symbols, facts, inferences, unknowns, conflicts, validity, and invalidation conditions.

Graphify may be refreshed frequently because generation is inexpensive. AI-generated interpretation is updated only for materially changed modules or conflicts.

Agents verify the map but do not rediscover the entire territory.

## 7. Versioning

- Never overwrite accepted historical artifacts.
- Use `V001`, `V002`, and later version folders.
- Maintain explicit `CURRENT.md` pointers.
- Preserve numbered prompts and reports.
- Corrections supersede old findings; they do not silently erase them.

## 8. Evidence

Deterministic evidence should be produced by scripts where practical. AI summaries should explain root cause, design, risk, limitations, and verdict rather than restating full logs.

PASS requires evidence appropriate to risk. Missing physical proof must remain explicitly blocked or pending.

## 9. Security

The coordination repository must contain no secrets, private customer data, protected production artifacts, or full proprietary source. Because the repository is public, raw Graphify artifacts are allowed only after proving they are public-safe and sanitized.
