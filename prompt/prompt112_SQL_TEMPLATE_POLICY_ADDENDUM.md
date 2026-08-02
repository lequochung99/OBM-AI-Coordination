# Binding Addendum to Prompt 112 — Canonical PostgreSQL creation-script location and finalization policy

This addendum is binding context for prompt112 and all later OBM-POS database/schema tasks.

## Operator-authoritative canonical location

The operator has designated this local folder as the canonical storage location for the two PostgreSQL database creation templates:

```text
E:\Project2026\2SQL PostgreSQL
```

There are two existing script lanes in that folder:

```text
1. WPF / local OBM-POS PostgreSQL database creation template
2. ApiServer PostgreSQL database creation template
```

Inspect the actual existing file names and formats before changing anything. Do not guess or silently create a third competing script lane.

## Current prompt112 rule

Prompt112 is a runtime credential/startup and one-group happy-path task. It must not prematurely rewrite the two canonical SQL templates.

For prompt112:

```text
- inventory the two existing canonical script paths and safe file names;
- record their current version/checksum when available;
- preserve them unchanged;
- do not treat them as a credential source;
- do not place passwords, tokens, connection strings, tenant/POS identifiers, or business data in them.
```

The prompt112 public report does not need to expose private absolute file contents. The private artifact should record that the canonical folder policy was read and preserved.

## When the scripts must be finalized

The two scripts must be regenerated/finalized only after the applicable database schemas are complete and accepted, including all remaining schema-affecting work such as:

```text
Service Category Weight
Customer / Booking Weight
any accepted TurnEngine schema changes
any other approved WPF local schema changes
any matching ApiServer event/delivery/domain schema changes
```

Do not call the database scripts final while accepted migrations are still expected to change.

## Source-of-truth rule

The scripts under `E:\Project2026\2SQL PostgreSQL` are reproducible deployment/bootstrap artifacts derived from the accepted source migration chains.

They are not a parallel schema authority.

Canonical authority remains:

```text
WPF EF Core/Npgsql migration chain
API ExternalDbContext EF Core/Npgsql migration chain
accepted model mappings, constraints, and indexes
```

Never fix only the SQL template while leaving the migration/model source incorrect. Correct source first, prove migration-from-zero, then regenerate the derived script.

## Required final outputs

At schema freeze, produce exactly two canonical script outputs using the existing two lanes:

```text
WPF/local creation script
API creation script
```

Each must be able to reproduce its target from an empty approved PostgreSQL environment and must be physically verified on a newly created disposable database.

Required proof for each script:

```text
PostgreSQL/Npgsql only
UTF8 database
correct target database role/purpose documented without embedded credentials
complete accepted migration/schema chain
__EFMigrationsHistory consistent with the accepted source chain when applicable
pending migrations = 0 after apply
all required tables, columns, constraints, indexes, and foreign keys present
non-persistent write probes pass
no SQL Server/LocalDB/SQLite fallback
no manual divergence from EF model/migrations
```

The API script must include the accepted API schema, including the canonical event/delivery/group-ACK structures. The WPF script must include the accepted local schema, including `TblLocalOutbox` and all final TurnEngine/Weight structures.

## Database creation versus schema application

Preserve the existing script style when valid. If the current format separates PostgreSQL database creation from schema/migration application, retain that explicit boundary.

Do not place `CREATE DATABASE` inside an invalid transaction block. Do not use `EnsureCreated` as a replacement for migrations.

If psql meta-commands such as `\connect` are used, document that the script is a psql script. If scripts are runner-oriented instead, document the required runner. Do not mix execution models ambiguously.

## Seed boundary

The two canonical creation scripts must not contain customer/business/runtime data.

Do not seed:

```text
Invoice
TblOutputInfo
Booking
Customer
employee/service/customer business records
queue/turn runtime history
TblEventLog/TblEventDelivery history
production tenant/POS identities
```

Only explicitly accepted minimal baseline seed may be included, and only when the project has frozen that seed contract. Prefer keeping schema creation and seed scripts logically separated when the existing folder conventions support it.

## Versioning and preservation

Follow OBM artifact versioning policy:

```text
never overwrite or delete the previous accepted version automatically
create a new versioned script or versioned folder
retain previous versions for rollback/audit
maintain a clear latest/current pointer or index
record SHA-256 checksums
```

If the folder currently uses stable file names, preserve the prior versions in a versioned archive before updating the stable current copies. Do not destroy historical scripts.

## Final acceptance gate

Before production readiness, a dedicated task must prove:

```text
fresh WPF database created from the canonical WPF/local script
fresh API database created from the canonical API script
both match their accepted migration/model snapshots
both have pending migrations = 0
both pass physical schema and write probes
checksums and latest pointers are recorded
```

This final SQL-template gate is deferred until the schema is complete. Prompt112 must only preserve and document the canonical location.
