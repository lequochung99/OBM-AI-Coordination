# prompt141 — 100% C# evidence audit for why WPF still routes directly to MainWindow despite missing bootstrap baseline

## Objective

Do not modify source in this prompt. Produce a complete, evidence-backed C# audit explaining exactly why the latest physical WPF launch still went directly to MainWindow after prompt140, even though the current DB had previously been proven to have missing Phase2 completion marker and zero approved baseline counts.

The operator explicitly reports that after rebuilding and launching the latest WPF, the application still routes directly into the POS application instead of blocking and returning to InstallationV0/repair flow.

This prompt is investigation/report-only. No source edits, no DB mutations, no secret/config changes, no state patching, no reset, no migration, no seed, and no process automation beyond safe read-only inspection.

## Canonical authority

Use:

`E:\Project2026\4POS\NailSalonNet8\docs\refactoryInstallation\INSTALLATION_RUNTIME_CANONICAL_V005.md`

Key rule:

```text
ApplicationReady/Activated alone is not sufficient.
Startup may route to MainWindow only when the canonical bootstrap completion proof and required baseline state are present.
```

## Required investigation scope

### A. Prove which binary and source are actually running

Capture sanitized evidence for:

- exact WPF executable path;
- process name and PID;
- build output path;
- visible build label value;
- `InstallationV0BuildInfo` value compiled into the running binary;
- assembly file timestamp/version if available;
- whether the process belongs to the source tree that contains prompt140 corrections.

Do not merely state that Visual Studio was rebuilt. Prove the exact executable and compiled label.

### B. Trace the complete startup C# call chain

Starting from WPF process startup, report every class/method that can decide between InstallationV0 and MainWindow.

At minimum inspect and quote the relevant complete C# blocks from owners such as:

- `App` startup/launch methods;
- InstallationV0 module/bootstrap registration;
- `LocalPosStartupService`;
- `RuntimeProfileStartupAssessmentService`;
- runtime-profile repository;
- any MainWindow eligibility helper;
- any fallback or legacy startup path;
- any direct `new MainWindow()`, `Show()`, `ShowDialog()`, or equivalent launch path;
- any branch using only `Activated`, `ApplicationReady`, tenant/POS identity, or DB connectivity;
- any exception fallback that opens MainWindow;
- any debug/development bypass;
- any second startup assessment path not updated by prompt140.

For every hop provide:

| Order | File | Class | Method | Input | Condition evaluated | Returned decision | Next call |
|---:|---|---|---|---|---|---|---|

### C. Show the exact C# predicates

Quote the complete current C# predicates that determine:

1. `Activated/ApplicationReady` recognition;
2. completion-marker existence;
3. baseline-count validation;
4. Tenant/POS identity validation;
5. MainWindow eligibility;
6. InstallationV0/recovery eligibility;
7. any fallback when marker table is missing;
8. any fallback when a SQL query throws;
9. any fallback when baseline table/query is missing;
10. any direct route used by restart/startup versus the Open OBM-POS button.

Do not provide paraphrase only. Include enough surrounding C# to prove control flow, while avoiding secrets and private identity values.

### D. Compare the two potentially different routes

Audit separately:

#### Route 1 — Clicking `Open OBM-POS` from InstallationV0

```text
InstallationV0Window
-> command handler
-> module/app owner
-> startup assessment
-> MainWindow launch
```

#### Route 2 — Normal process startup/restart

```text
App startup
-> local configuration load
-> startup route assessment
-> InstallationV0 or MainWindow
```

Determine whether prompt140 updated only one route while the physical launch used another.

This comparison is mandatory.

### E. Read-only runtime instrumentation

Using safe diagnostics or debugger/breakpoints without source mutation, capture the actual runtime values for the physical startup decision:

```text
running build label
resolved ProductRoot
resolved DB name
runtime profile row count
runtime profile state
completion marker table exists: true/false
completion marker row count
baseline count results for each required owner
startup assessment result enum/code
exact branch selected
exact method that opens MainWindow
```

Do not print passwords, connection strings, tokens, pairing codes, raw Tenant/POS GUIDs, Google identity, or private business payloads.

If the UI cannot be driven by Codex, provide exact breakpoint locations and ask the operator only for the minimum debugger observations needed. However, still complete all static C# analysis first.

### F. Check migration/marker query correctness

Inspect whether the prompt140 completion marker table:

- exists in the migration SQL actually embedded/copied to output;
- uses the exact same schema/table/column names as the startup query;
- is queried with correct PostgreSQL quoting/casing;
- is read from the same database and ProductRoot configuration as runtime;
- is not silently treated as optional when missing;
- cannot return a false positive due to `to_regclass`, `EXISTS`, default values, catch blocks, or fallback logic.

Report the exact SQL strings and the C# methods that execute them.

### G. Search for duplicate or stale C# owners

Search the complete source tree for:

```text
Activated
ApplicationReady
MainWindow
RuntimeProfileStartupAssessmentService
LocalPosStartupService
TblPhase2
completion marker
baseline
OpenInstalledPos
ShowMainWindow
```

Identify every distinct owner/path. Classify each as:

- canonical active owner;
- test-only;
- obsolete but still reachable;
- duplicate/parallel route;
- unrelated.

### H. Root-cause conclusion

Choose exactly one primary classification, supported by direct evidence:

```text
C1_RUNNING_STALE_BINARY
C2_NORMAL_STARTUP_BYPASSES_PROMPT140_ASSESSMENT
C3_OPEN_COMMAND_AND_NORMAL_STARTUP_USE_DIFFERENT_ASSESSORS
C4_MARKER_QUERY_FALSE_POSITIVE
C5_BASELINE_VALIDATION_NOT_EXECUTED
C6_EXCEPTION_FALLBACK_OPENS_MAINWINDOW
C7_DUPLICATE_LEGACY_STARTUP_PATH_REMAINS_ACTIVE
C8_DATABASE_OR_PRODUCTROOT_MISMATCH
C9_OTHER_PROVEN_CSHARP_CONTROL_FLOW_DEFECT
```

Do not use `unknown` unless all mandatory evidence has been collected and the precise missing proof is listed.

## Required report structure

Create:

`report/report141.md`

The report must contain:

1. Verdict.
2. Running binary proof.
3. Full startup call-chain table.
4. Route 1 versus Route 2 comparison.
5. Complete relevant C# predicate excerpts.
6. Marker/baseline SQL and C# owner comparison.
7. Runtime values at the branch, sanitized.
8. Duplicate/stale owner inventory.
9. Exact root-cause classification.
10. Minimal proposed correction, but do not implement it.
11. Exact files/methods that would change in a later approved prompt.
12. Build/test status only if no mutation is required.
13. Zero-mutation and zero-secret confirmation.

## Evidence standard

Because several previous fixes passed tests but failed physically, this prompt requires 100% direct proof. Statements such as “should,” “likely,” “appears,” or “tests cover it” are insufficient for the final root-cause claim.

At least one direct code excerpt is required for every control-flow assertion.

## Frozen / forbidden

Do not:

- modify any C# or SQL;
- change build label;
- edit launchSettings;
- change ProductRoot/configuration;
- run migration or seed;
- insert/update runtime state;
- create/drop/reset/copy any DB;
- add a new marker/readiness table;
- create a second startup coordinator;
- bypass marker/baseline checks;
- change pairing/API/sync/SignalR;
- expose secrets or raw identities.

## Expected verdict

Use one of:

```text
PROMPT141_CSHARP_STARTUP_BYPASS_ROOT_CAUSE_PROVEN_READY_FOR_MINIMAL_FIX
```

or a precise blocked verdict naming the single missing physical/debugger observation.
