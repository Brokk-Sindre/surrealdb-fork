# CLI / Server (Repo-Verified)

Verification sources:
- `surrealdb/server/src/cli/mod.rs`
- `surrealdb/server/src/cli/module/mod.rs`
- `surrealdb/core/src/dbs/capabilities.rs`
- `tests/cli_integration.rs`
- `tests/ws_integration.rs`

## Server start patterns

```bash
surreal start --username root --password root memory
surreal start --username root --password root --bind 0.0.0.0:8000 memory
surreal start --username root --password root --web-crt cert.crt --web-key key.pem memory
```

Capability-related flags used in verified flows:
- `--allow-experimental files`
- `--allow-experimental surrealism`
- `--deny-arbitrary-query guest`
- `--deny-arbitrary-query record`
- RPC allow/deny examples in tests: `--deny-rpc --allow-rpc version,use,attach`

## Storage backends seen in repo docs/tests

- `memory`
- `rocksdb://...`
- `surrealkv://...`
- `tikv://...`

## CLI commands surface

From `cli/mod.rs`:
- `start`
- `import`
- `export`
- `version`
- `upgrade`
- `sql` (when `cli` feature is enabled)
- `ml`
- `module` (when `surrealism` feature is enabled)
- `isready`
- `validate`
- `fix`

## `surreal sql` auth-level behavior

`tests/cli_integration.rs` covers:
- root/namespace/database auth-level combinations
- required flags for each auth level
- token-based flows
- namespace/database resolution caveats

Practical rule:
- always set namespace/database explicitly unless token claims already encode required scope.

## Module CLI subcommands

When `module` command is compiled in (`surrealism` feature), available subcommands:
- `surreal module build`
- `surreal module info`
- `surreal module sig`
- `surreal module run`

## Import/export

`tests/cli_integration.rs` validates import/export command paths and expected auth/context behavior.
