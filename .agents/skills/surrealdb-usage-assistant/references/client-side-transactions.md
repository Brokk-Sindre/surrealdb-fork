# Transactions (Repo-Verified Scope)

Verification sources:
- `language-tests/tests/language/statements/transaction/throw_error_handling.surql`
- `language-tests/tests/language/statements/use/ns_db_in_tx.surql`
- `surrealdb/tests/api_integration/basic.rs`
- `surrealdb/tests/api_integration/session_isolation.rs`
- `surrealdb/core/src/rpc/protocol.rs`
- `surrealdb/server/src/rpc/http.rs`
- `surrealdb/server/src/rpc/websocket.rs`

This file covers only:
- SurrealQL transaction semantics
- Rust SDK client-side transactions
- WS RPC transaction semantics

## 1. SurrealQL transactions

```sql
BEGIN TRANSACTION;
    CREATE order:1 SET total = 100;
    UPDATE customer:alice SET open_orders += 1;
COMMIT TRANSACTION;
```

Rollback/cancel:

```sql
BEGIN;
    CREATE person:temp;
CANCEL;
```

Behavior from language tests:
- errors inside transaction abort the transaction
- explicit `THROW` in transaction path aborts commit path
- `USE NS ... DB ...` can be exercised in transaction-oriented statement coverage (`use/ns_db_in_tx.surql`)

## 2. Rust SDK client-side transactions

From `surrealdb/tests/api_integration/basic.rs`:

```rust
let tx = db.begin().await?;
tx.query("CREATE user:one SET name = 'One';").await?;
let db = tx.commit().await?;

let tx = db.begin().await?;
tx.query("CREATE user:two SET name = 'Two';").await?;
let db = tx.cancel().await?;
```

Also verified:
- multiple operations in one tx before commit
- uncommitted changes are invisible across other sessions (`session_isolation.rs`)
- cancelled tx changes are not persisted

HTTP limitation in SDK tests:
- client-side tx path is unavailable on HTTP protocol builds (`basic.rs` has protocol-http unsupported case)

## 3. WS RPC transactions

Transport behavior from server RPC implementations:
- HTTP RPC: `begin`/`commit`/`cancel` => method-not-found (`http.rs`).
- WS RPC: implemented with per-socket tx map (`websocket.rs`).

WS tx flow:
1. `begin` -> returns transaction UUID.
2. Run transactional statements by sending RPC requests with top-level `txn` UUID.
3. `commit` or `cancel` with `params: ["<txn-uuid>"]`.

Example:
```json
{"id":1,"method":"begin","params":[]}
{"id":2,"method":"query","txn":"<uuid>","params":["CREATE t:1 SET v = 1;"]}
{"id":3,"method":"commit","params":["<uuid>"]}
```

## Practical rules

1. Prefer SDK tx API for Rust application code.
2. Prefer WS RPC if you need transaction control over raw RPC.
3. Do not document HTTP RPC transaction support; it is explicitly unsupported in code.
