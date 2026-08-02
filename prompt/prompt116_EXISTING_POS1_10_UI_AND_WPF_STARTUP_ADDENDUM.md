# Prompt 116 Addendum — Preserve the working POS1–POS10 Platform UI and close the WPF startup gate before physical sync proof

This addendum is binding and must be read together with:

```text
prompt/prompt116.md
prompt/prompt116_POS_STATION_DEVICE_AGGREGATE_ADDENDUM.md
```

## Authoritative operator facts

The operator confirms that an existing UI already supports:

```text
POS1 through POS10 selection/configuration
Pairing Code generation
POS/POS-slot details
working Platform-side behavior
```

Do not recreate, replace, redesign, or bypass this UI or its active API/application call chain.

The operator also confirms:

```text
current WPF does not start
```

A screenshot of the working POS1–POS10 UI may be provided later after WPF is operational. The screenshot is supplementary visual evidence, not a prerequisite for source/call-chain audit.

## Mandatory architectural interpretation

Treat the existing POS1–POS10 UI as the authoritative starting boundary for logical station selection and Pairing Code creation.

Prompt116 must trace the complete active call chain:

```text
PlatformAppV0 POS1–POS10 UI
-> page/view-model/component action
-> existing application/API service
-> tenant/POS station selection or creation
-> Pairing Code creation
-> current canonical database writers
-> WPF redeem
-> bootstrap/activation state
-> proven TblTenantPosDevice writer point, when appropriate
```

Do not assume the UI itself must write `TblTenantPosDevice`.

The correct physical-device registration point may be:

```text
Pairing Code redeem
bootstrap identity confirmation
installation activation/completion
device replacement/reactivation
```

Choose the writer point from complete lifecycle evidence, not convenience.

## Preserve the working Platform UI

Forbidden changes unless a narrowly proven defect is required for the existing lifecycle:

```text
new POS1–POS10 UI
new Pairing Code endpoint
parallel POS selection service
parallel tenant/POS writer
new pairing workflow
renaming or deleting the active POS slots
changing the current working UI contract merely to satisfy TblTenantPosDevice tests
```

Required proof:

```text
EXISTING_POS_SLOT_UI_IDENTIFIED=true
POS_SLOT_RANGE=1..10
PAIRING_CODE_UI_CALL_CHAIN=<exact component/service/API chain>
LOGICAL_POS_WRITER=<exact existing writer>
CURRENT_UI_BEHAVIOR_PRESERVED=true
PARALLEL_UI_OR_PAIRING_PATH_COUNT=0
```

## WPF startup is a hard prerequisite for the physical happy path

Prompt116 must audit and close the current WPF startup failure before attempting:

```text
Pairing Code redeem runtime proof
canonical Provider physical invocation
periodic outbox worker cycle
Price Rule physical Save-to-API proof
manual POS1 test readiness
```

Capture direct evidence before changing source:

```text
WPF_START_COMMAND_OR_PROFILE=<exact command/profile>
WPF_START_STAGE=<bootstrap/config/database/migration/UI/other>
WPF_LAST_SUCCESSFUL_MARKER=<exact sanitized marker>
WPF_FIRST_FAILED_CLASS_METHOD=<exact class/method>
WPF_SANITIZED_EXCEPTION_CHAIN=<types/messages without secrets>
POSTGRES_SQLSTATE=<value or NOT_AVAILABLE>
WPF_PROCESS_EXIT_OR_HANG_STATE=<exact state>
```

Use the smallest production-capable correction.

Do not solve WPF startup by:

```text
disabling Pairing Code/WpfJwt validation
restoring Firebase email/password
bypassing protected machine-state/checkpoint
changing to a noncanonical database
using SQL Server/SQLite/InMemory fallback
skipping migrations with EnsureCreated
removing the canonical Provider
adding a second startup lane
```

Local-first lock remains binding:

```text
when the canonical local PostgreSQL DB and runtime identity are valid,
API/cloud/token communication failure must not prevent normal local WPF startup and MainWindow operation,
except where an unfinished installation/activation state legitimately requires the existing installation UI.
```

If WPF startup cannot be repaired safely in this task, return a narrow blocker such as:

```text
BLOCKED_MAIN_DEV_WPF_STARTUP
BLOCKED_MAIN_DEV_WPF_RUNTIME_DB
BLOCKED_MAIN_DEV_WPF_BOOTSTRAP_STATE
BLOCKED_MAIN_DEV_WPF_MIGRATION_CURRENT
```

Include the exact failed boundary and keep:

```text
MANUAL_POS1_TEST_READY=false
```

## Ordering added to prompt116

Execute in this order:

```text
1. Audit and preserve the existing POS1–POS10 UI and Pairing Code call chain.
2. Prove the logical TblPosLocal/station writer already used by that UI.
3. Prove the correct physical-device registration lifecycle for TblTenantPosDevice.
4. Define the POS-station/device aggregate and component cardinality from source.
5. Implement the canonical TblTenantPosDevice writer and attached migration only after lifecycle proof.
6. Apply migration/grants and prove pending migrations = 0.
7. Repair and physically prove WPF startup through the canonical Development lane.
8. Create Development POS1/POS2 through the existing Platform/pairing/activation boundaries.
9. Complete the physical Price Rule -> periodic worker -> canonical Provider -> API happy path.
10. Set MANUAL_POS1_TEST_READY=true only after every physical gate passes.
```

## Required private evidence additions

Add to the prompt116 artifact:

```text
EXISTING_POS1_10_UI_CALL_CHAIN.md
PAIRING_CODE_EXISTING_BOUNDARY.md
LOGICAL_POS_WRITER_PROOF.md
WPF_STARTUP_FAILURE_BEFORE.md
WPF_STARTUP_CORRECTION.md
WPF_STARTUP_PHYSICAL_PROOF.md
```

## Required public report additions

Add to `report/report116.md`:

```text
Existing POS1–POS10 UI identified yes/no
Existing Pairing Code UI/API path preserved yes/no
Parallel POS/pairing path introduced count
Logical POS writer proven yes/no
WPF startup failure boundary identified yes/no
WPF startup physically succeeds yes/no
WPF canonical runtime DB proof yes/no
WPF pending migrations count
WPF reaches installation UI or MainWindow as expected yes/no
```

## Acceptance lock

A working Platform UI alone does not make the end-to-end lane ready.

`MANUAL_POS1_TEST_READY=true` remains forbidden until:

```text
existing POS1–POS10 UI is preserved
canonical TblTenantPosDevice writer/schema is proven
WPF starts physically
canonical Provider is invoked physically
one grouped POS1 -> API commit succeeds
SignalR occurs after commit
local outbox group becomes Sent atomically
```

Service Category Weight and Booking Weight remain deferred until this PASS state.
