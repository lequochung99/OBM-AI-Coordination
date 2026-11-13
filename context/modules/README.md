# Module Context Registry

Create one folder per stable business or technical module.

```text
context/modules/<ModuleName>/
  CURRENT.md
  V001/
    MODULE_CONTEXT.md
    CALL_CHAIN.md
    DATA_CONTRACT.md
    TEST_MAP.md
    KNOWN_RISKS.md
```

Only create files that provide durable value. Small modules may use a single `MODULE_CONTEXT.md`.

## Required module metadata

- module name and responsibility;
- source repositories and verified commit SHAs;
- entry points and direct call chains;
- owned tables/entities/endpoints;
- read/write and transaction boundaries;
- tests and runtime verification points;
- protected invariants;
- known gaps and invalidation conditions;
- numbered tasks that last verified or changed the module.

## Update policy

A task updates module context only when it discovers a reusable fact or changes the documented behavior. Cosmetic changes do not require a new module-context version.
