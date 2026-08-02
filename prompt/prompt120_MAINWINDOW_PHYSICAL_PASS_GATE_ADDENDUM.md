# Prompt120 Binding Addendum — MainWindow physical proof is the only PASS gate

This addendum is binding and overrides any broader or conflicting interpretation in `prompt/prompt120.md`.

## Authoritative operator correction

The operator has confirmed that the WPF still does not enter the main OBM-POS application and remains in the InstallationV0 flow. Therefore the current state is **not PASS**.

Do not report PASS merely because any of the following succeed:

```text
build/tests pass
migrations apply
pending migrations = 0
baseline rows exist
Phase2 service returns success
process stays alive
InstallationV0 remains responsive
API is offline without crashing
```

Those are prerequisites only. They do not prove that OBM-POS is usable.

## Single PASS condition

PASS requires direct physical proof of the actual normal WPF Development launch:

```text
visible label = prompt120
canonical existing ProductRoot selected
canonical WPF DB = obm_pos_dev_v0_pg
provider = Npgsql/PostgreSQL
required local schema/readiness tables exist
pending migrations = 0
minimal baseline valid
Phase2 completion valid
local runtime activation valid
local station identity valid
API port 7161 has no listener
normal launch opens the real OBM-POS MainWindow directly
InstallationV0 is not shown, flashed, or left behind
MainWindow remains alive and responsive for at least 60 seconds
application closes normally
second normal launch again opens MainWindow directly
```

The physical MainWindow must be the real production WPF shell used by the operator, not a test window, runner, mocked shell, hidden process, or manually invoked navigation substitute.

## InstallationV0 means BLOCKED

If the application still shows InstallationV0 for any reason, the task must return a blocker. It must not return PASS.

Allowed blockers:

```text
BLOCKED_WPF_LOCAL_SCHEMA_READINESS
BLOCKED_WPF_PHASE2_BASELINE_COMPLETION
BLOCKED_WPF_LOCAL_ACTIVATION
BLOCKED_WPF_STATION_IDENTITY
BLOCKED_WPF_MAINWINDOW_ROUTING
BLOCKED_WPF_MAINWINDOW_PHYSICAL_PROOF
```

The report must name the exact missing table, marker, row, predicate, class, method, and owning boundary. Do not return generic text such as `SchemaMigrationRequired`, `Phase2 incomplete`, or `Installation state invalid` without exact evidence.

## No bypasses

Do not:

```text
open MainWindow blindly
hard-code readiness true
write completion/activation markers manually
fabricate station identity
skip required migrations
use EnsureCreated
reset/drop/recreate the WPF DB
redeem a new pairing code
add a refresh-token system
require API/WpfJwt validity for local MainWindow after local activation is valid
```

Use only the existing canonical migration and Phase2 installation boundaries.

## Required public report fields

`report/report120.md` must explicitly include:

```text
Verdict
Visible label physically observed
Exact MainWindow type/title observed
MainWindow opened directly yes/no
InstallationV0 shown or flashed yes/no
Canonical ProductRoot proof
Canonical WPF DB/provider proof
Exact readiness tables/markers before and after
Pending migrations count
Minimal baseline proof
Phase2 completion proof
Local activation proof
Station identity proof
API listener at 7161 during acceptance yes/no
60-second MainWindow proof yes/no
Second-launch MainWindow proof yes/no
Operator MainWindow screenshot ready true/false
Manual POS1 test ready false
```

The only PASS verdict remains:

```text
OBM_WPF_LOCAL_PHASE2_READY_MAINWINDOW_OFFLINE_PHYSICALLY_RESTORED_READY_FOR_OPERATOR_SCREENSHOT
```

That verdict is forbidden unless `MainWindow opened directly = yes`, `InstallationV0 shown or flashed = no`, `60-second MainWindow proof = yes`, and `Second-launch MainWindow proof = yes` are all physically proven.

Until then:

```text
OPERATOR_MAINWINDOW_SCREENSHOT_READY=false
MANUAL_POS1_TEST_READY=false
CATEGORY_WEIGHT=DEFERRED
BOOKING_WEIGHT=DEFERRED
```
