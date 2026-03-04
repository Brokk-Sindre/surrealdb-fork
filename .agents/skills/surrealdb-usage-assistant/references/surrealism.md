# Surrealism Modules (Repo-Verified)

Verification sources:
- `surrealdb/core/src/sql/statements/define/module.rs`
- `surrealdb/core/src/sql/statements/remove/module.rs`
- `surrealdb/core/src/sql/module.rs`
- `surrealdb/core/src/rpc/protocol.rs`
- `surrealdb/core/src/dbs/capabilities.rs`
- `surrealdb/server/src/cli/module/mod.rs`
- `surrealdb/server/src/cli/module/build.rs`
- `surrealism/test/test.surql`

## Scope

This repo supports module-based Surrealism flows.
This file uses only module-based SQL syntax that exists in this repository.

## SQL module definition surface

From SQL statement implementations:
- define: `DEFINE MODULE ...`
- remove: `REMOVE MODULE ...`

Minimal pattern:
```sql
DEFINE BUCKET test BACKEND "file:/absolute/path";
DEFINE MODULE mod::test AS f"test:/demo.surli";
```

In-repo example (`surrealism/test/test.surql`) uses this exact `DEFINE MODULE mod::...` shape.

## Module naming/runtime forms

From `sql/module.rs` and RPC `run` dispatch:
- module name form: `mod::<name>`
- silo form also exists in runtime naming (`silo::<org>::<pkg><major.minor.patch>`)

`run` receiver handling (`protocol.rs`) supports:
- `mod::...` (module runtime)
- `silo::...` (requires semantic version string)

## Capability gating

From `capabilities.rs` + `protocol.rs`:
- `mod::...` and `silo::...` execution requires experimental capability `surrealism`.
- Without it, RPC returns parameter error indicating surrealism is not enabled.

Typical startup capability usage:
- `--allow-experimental surrealism`

## CLI module commands

From `server/src/cli/module/mod.rs`:
- `surreal module build`
- `surreal module info`
- `surreal module sig`
- `surreal module run`

### Build constraints

From `build.rs`:
- requires `surrealism.toml` in source path
- runs `cargo build --target wasm32-wasip1 --release`
- strips/optimizes wasm, then packs a Surrealism package

### Run/sig/info usage shape

From CLI command definitions:
- `run`: takes module file path, optional `--fnc`, repeatable `--arg`
- `sig`: takes module file path, optional `--fnc`
- `info`: takes module file path

## Removal

```sql
REMOVE MODULE mod::test;
```

## Practical rules

1. Use `DEFINE MODULE` / `REMOVE MODULE` syntax for module lifecycle operations.
2. Keep surrealism marked as experimental-capability-gated.
3. For execution examples, use `run` receiver forms (`mod::...` / `silo::...`) that match `protocol.rs`.
