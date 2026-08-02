# prompt144 — Controlled reset of over-engineered WPF installation and minimal single-folder rebuild

## Authority and intent

The operator has explicitly decided that the current WPF installation implementation is over-engineered, has lost control, and must be removed and rebuilt simply.

Canonical architecture authority remains:

`E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL_V005.md`

However, this prompt intentionally simplifies the implementation structure while preserving the V005 ordering:

```text
Test local PostgreSQL
-> redeem Pairing Code
-> persist protected Tenant/POS checkpoint
-> create/resume local DB
-> migrate
-> seed missing baseline data
-> mark ApplicationReady/Activated
-> open MainWindow
```

The implementation must return to the simple operator-approved principle:

```text
check required data
-> if missing, seed missing data idempotently
-> check again
-> if complete, enable/open POS
```

## Mandatory safety checkpoint before deletion

Before deleting or moving any source:

1. Record `git status --short`, current branch, HEAD SHA, and untracked installation-related files.
2. Create a local recovery artifact folder:
   `E:\Project2026\RecoveryReports\WpfInstallationResetPrompt144V001`
3. Copy the complete pre-reset installation-related source inventory and diffs into that private artifact.
4. Create a recoverable local Git checkpoint commit or branch containing the current WPF source state, including all relevant tracked and intentionally added untracked installation files.
5. Report the recovery branch/commit SHA without secrets.
6. Do not modify ApiServer01 or PlatformAppV0 source in this prompt.

If a complete recoverable checkpoint cannot be created, stop with:

`BLOCKED_PROMPT144_RECOVERY_CHECKPOINT_NOT_CREATED`

## Destructive scope — what must be removed

Delete the current over-engineered WPF installation implementation and duplicate installation/readiness owners, including installation-only code currently distributed outside the `InstallationV0` directory.

Required audit-and-delete candidates include, but are not limited to:

- existing `InstallationV0` implementation files that belong to the old architecture;
- duplicate readiness/status/verification owners;
- duplicate startup assessment semantics;
- Phase2 hydration readiness semantics;
- installation-only command-rule frameworks;
- installation-only verification/reporting services;
- obsolete local application readiness executors;
- legacy external schema-tool wrappers;
- obsolete V003/V004/V005 compatibility adapters that are not required by the rebuilt flow;
- installation-specific logic embedded in `App.xaml.cs` beyond a minimal adapter;
- installation-specific logic in:
  - `Services/Startup/LocalPosStartupService.cs`
  - `Services/Startup/RuntimeProfileStartupAssessmentService.cs`
  - `Modules/Configuration/SpacePosRuntimeConfiguration.cs`
  - any other file outside `InstallationV0` that exists only to implement installation/readiness.

Do not delete an entire shared file if it also owns unrelated runtime behavior. Instead remove installation-specific logic and leave only a minimal adapter to the rebuilt module.

## Frozen code that must not be redesigned or deleted

Do not redesign or delete:

- MainWindow and normal POS domain behavior;
- existing canonical WPF-to-API sync, outbox, Provider, API endpoint, or SignalR flow;
- PlatformAppV0 authorization and Pairing Code issuance;
- ApiServer pairing redeem endpoint and WpfJwt contract;
- payment terminal, CompanionApp, BookingConsole, Firebase, category/booking/price work;
- business CRUD entities or historical data;
- approved attached EF/Npgsql schema artifact unless a compile/runtime defect is proven;
- protected checkpoint/token stores if they are already working and can be reused with a thin adapter.

## Target folder boundary

All rebuilt WPF installation implementation must live under exactly:

`E:\Project2026\4POS\NailSalonNet8\InstallationV0`

Outside that directory, only minimal integration adapters are permitted:

### `App.xaml.cs`

May only:

```text
call InstallationV0 public startup API
receive one route result
open InstallationV0 window or MainWindow
```

It must not contain:

- SQL;
- table counts;
- marker logic;
- seed rules;
- checkpoint semantics;
- installation state-machine logic.

### Main WPF project registration

May only register the InstallationV0 public interface/service and invoke it.

No other installation owner may remain outside `InstallationV0`.

## Required minimal structure

Use a small structure similar to the following. Do not add additional layers unless compilation forces them and the report justifies each file.

```text
InstallationV0/
  InstallationV0.csproj
  Application/
    InstallationCoordinator.cs
    InstallationStatus.cs
    InstallationResult.cs
  Configuration/
    LocalDatabaseSettings.cs
    LocalDatabaseSettingsStore.cs
  Database/
    LocalDatabaseInstaller.cs
    WpfPostgreSqlMigrations.sql
  Bootstrap/
    BaselineSeeder.cs
    BaselineVerifier.cs
  Pairing/
    PairingCheckpointAdapter.cs
  Presentation/
    InstallationV0Window.xaml
    InstallationV0Window.xaml.cs
  Public/
    IInstallationV0Gateway.cs
    InstallationV0Gateway.cs
  InstallationV0BuildInfo.cs
```

Maximum canonical behavioral owners:

1. `InstallationCoordinator` — the one orchestration owner.
2. `LocalDatabaseInstaller` — create/resume DB and apply migrations.
3. `BaselineSeeder` — idempotently insert only missing approved baseline rows.
4. `BaselineVerifier` — read-only completion check.
5. `InstallationV0Gateway` — the single public adapter used by `App.xaml.cs`.

Do not create a second startup coordinator, second verifier, second bootstrap executor, second token/checkpoint store, or second DB configuration owner.

## Exact minimal behavior

### Startup

```csharp
var status = await gateway.GetStatusAsync();

if (status == Ready)
    open MainWindow;
else
    open InstallationV0;
```

### Installation button

```csharp
await coordinator.InstallOrResumeAsync();
var status = await coordinator.GetStatusAsync();
OpenButton.IsEnabled = status == Ready;
```

### InstallOrResumeAsync

The coordinator must perform exactly:

```text
1. Validate local PostgreSQL inputs.
2. Require an existing valid protected Pairing/Tenant/POS checkpoint.
3. Create DB if absent; otherwise reuse it.
4. Apply attached migrations idempotently.
5. Seed only missing approved baseline data.
6. Verify baseline from actual data.
7. Write exactly one current runtime profile row as Activated/ApplicationReady only after verification passes.
8. Commit baseline + completion proof + Activated together where the existing transaction boundary permits.
9. Return Ready.
```

No UI hydration framework may decide readiness.

## Baseline policy

Seed only the approved minimal baseline required to open the app safely:

- required settings;
- required parameters;
- printer defaults;
- default roles/permissions;
- required login/payment/setup defaults proven by the existing source;
- Tenant/POS/runtime setup identity rows.

Do not seed:

- employees unless an existing hard foreign-key or startup requirement proves one is mandatory;
- services;
- customers;
- invoices;
- TblOutputInfo;
- bookings;
- queue/turn history;
- event/delivery history.

The old legacy rule requiring `employeeCount >= 20` must not be preserved unless the operator explicitly approves it. Employee/business sample-data counts are not installation readiness.

Baseline seeding must be idempotent by stable key, not only by whole-table count:

```text
missing key -> insert
existing key -> leave unchanged
partial baseline -> insert only missing keys
complete baseline -> no-op
```

Installation/bootstrap must create zero `TblLocalOutbox` rows.

## Readiness rule

The one `BaselineVerifier` must check actual required data and identity.

Ready requires:

```text
schema current
+ valid protected Tenant/POS checkpoint
+ matching Tenant/POS local identity
+ all approved required baseline keys present
+ exactly one current runtime profile row in Activated/ApplicationReady semantic state
```

A marker/version row may be retained for audit, but it may not replace checking actual baseline keys.

If data is missing, return `BaselineMissing` and open InstallationV0. Never open MainWindow merely because an old `Activated` row exists.

## Existing development DB

Target development DB remains:

`obm_pos_dev_v1_pg`

The operator may have dropped/recreated it during testing. The rebuilt code must support both:

- DB absent;
- DB exists with schema only/partial baseline.

Do not automatically drop/recreate/reset/copy any database.

## UI requirements

Use the existing simple InstallationV0 window appearance where practical.

Required controls/state:

- PostgreSQL host/port;
- provisioning username/password;
- runtime username/password;
- database name;
- Pairing Code redeem or existing checkpoint status;
- one `Install/Resume Local Database` button;
- one `Open OBM-POS` button;
- one concise status line.

Button logic:

```text
Install/Resume enabled
= local inputs valid + checkpoint valid + not busy + not Ready

Open OBM-POS enabled
= BaselineVerifier says Ready
```

No hidden Phase2 hydration predicate may disable the install button.

Set visible label exactly:

`prompt144`

## Tests

Delete obsolete tests that only verify removed duplicate owners.

Create focused tests for the rebuilt flow:

1. DB absent -> create -> migrate -> seed -> Ready.
2. Schema-only DB -> seed -> Ready.
3. Partial baseline -> insert missing keys only -> Ready.
4. Complete baseline -> no duplicate writes.
5. Missing checkpoint -> install blocked.
6. Invalid DB inputs -> install blocked.
7. Old Activated row with missing baseline -> not Ready.
8. Baseline complete -> Ready/MainWindow route.
9. Installation creates zero outbox rows.
10. Restart status is Ready without re-redeem or reseed.
11. API/token unavailable after Ready does not block MainWindow.
12. No destructive SQL in active source.

Do not preserve large obsolete test suites merely to keep historical architecture alive.

## Physical acceptance

After build/tests, run the main WPF Debug lane when possible:

```text
E:\Project2026\4POS\NailSalonNet8\bin\Debug\net8.0-windows\NailSalonNet8.exe
```

Required proof:

1. Visible label `prompt144`.
2. Pairing checkpoint is loaded or a new Pairing Code is redeemed.
3. DB absent or schema-only is handled by the same Install/Resume button.
4. One click continues through migrate and baseline seed; no dead-end.
5. Required baseline data is present.
6. Installation outbox count is zero.
7. Open OBM-POS becomes enabled only after baseline verification.
8. MainWindow opens and remains responsive for 60 seconds with API offline.
9. Two restarts open MainWindow directly.

If operator interaction prevents physical proof, report blocked physical truthfully; do not claim PASS.

## Required report

Create and push:

`report/report144.md`

The report must include:

- recovery checkpoint branch/commit;
- complete deleted-file inventory;
- complete remaining installation-related code outside `InstallationV0`;
- justification for every remaining outside adapter;
- new folder tree;
- canonical owner count;
- exact baseline key/count summary without private values;
- build/test totals;
- destructive operation counts;
- physical results or exact blocker.

PASS verdict only if source boundary and physical flow both pass:

`OBM_WPF_INSTALLATION_MINIMAL_SINGLE_FOLDER_REBUILD_PHYSICALLY_PROVEN`

Otherwise use an exact BLOCKED verdict.