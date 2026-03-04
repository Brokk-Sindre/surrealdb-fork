# Rust SDK (Repo-Verified)

Verification sources:
- `surrealdb/tests/api_integration/basic.rs`
- `surrealdb/tests/api_integration/run.rs`
- `surrealdb/tests/api_integration/live.rs`
- `surrealdb/tests/api_integration/session_isolation.rs`
- `surrealdb/tests/api_integration/mod.rs`
- `surrealdb/src/method/authenticate.rs`
- `surrealdb/src/method/invalidate.rs`
- `surrealdb/src/method/begin.rs`
- `surrealdb/src/method/commit.rs`
- `surrealdb/src/method/cancel.rs`

## Baseline remote flow

```rust
use surrealdb::engine::any::connect;
use surrealdb::opt::auth::Root;

let db = connect("ws://localhost:8000").await?;
db.signin(Root {
    username: "root",
    password: "root",
}).await?;
db.use_ns("myns").use_db("mydb").await?;
```

`use_ns(...).use_db(...)` is required context for most operations in integration tests.

## Query, variables, and extraction

From `basic.rs`:

```rust
let mut res = db.query("CREATE person SET name = $name RETURN *;")
    .bind(("name", "Alice"))
    .await?;

let rows: Vec<surrealdb::types::Value> = res.take(0)?;
```

Session variables are supported and isolated per client session:

```rust
db.set("my_var", 111).await?;
let mut res = db.query("RETURN $my_var").await?;
let value: Option<i32> = res.take(0)?;
db.unset("my_var").await?;
```

## `run` API

From `run.rs` and SDK method docs:

```rust
let answer: i64 = db.run("fn::answer").await?;
let doubled: i64 = db.run("fn::double").args(21).await?;
let two: f64 = db.run("math::log").args((100, 10)).await?;
```

Argument behavior (as tested/docs in repo):
- no `.args()` -> zero args
- single value -> one arg
- tuple -> multiple args
- array value -> single array arg

## Live query streams

From `live.rs`:

```rust
use futures::StreamExt;
use surrealdb::Notification;

let mut stream = db.select("person").live().await?;
while let Some(event) = stream.next().await {
    let event: Notification<surrealdb::types::Value> = event?;
    // event.action => Create/Update/Delete
}
```

Also covered:
- live on record ids and ranges
- `LIVE SELECT ... FETCH ...`
- `LIVE SELECT` returning UUID via query `take()`

## Client-side transactions

From `basic.rs`, `begin.rs`, `commit.rs`, `cancel.rs`:

```rust
let tx = db.begin().await?;
tx.query("CREATE user:one SET name = 'One';").await?;
let db = tx.commit().await?;   // returns Surreal<C>

let tx = db.begin().await?;
tx.query("CREATE user:two SET name = 'Two';").await?;
let db = tx.cancel().await?;   // rollback, returns Surreal<C>
```

Important transport note (verified in tests):
- client-side transaction API is not supported on HTTP protocol builds (`basic.rs` has protocol-http skip case).

## Token refresh and revocation flow

From `basic.rs`, `authenticate.rs`, `invalidate.rs`:

```rust
// Signin/signup with refresh support configured in DEFINE ACCESS ... WITH REFRESH
let token = db.signin(record_access).await?;

// Refresh token pair
let token = db.authenticate(token).refresh().await?;

// Revoke refresh token explicitly
db.invalidate().refresh(token).await?;
```

Observed behavior in tests:
- refreshed token rotates access token
- revoked refresh token cannot be reused for refresh
- access token may still authenticate until expiry/revocation policy applies

## Session isolation behavior

From `session_isolation.rs`:
- `db.clone()` creates isolated client session state.
- NS/DB selection is isolated per clone.
- `set`/`unset` variables are isolated per clone.
- transaction visibility is isolated until commit.
- authentication state is isolated; invalidating one client does not invalidate others.

## Practical rules

1. Prefer WS transport for live queries and transaction-heavy flows.
2. Always set NS/DB explicitly per client session.
3. For token lifecycle, use SDK refresh/revoke helpers (`authenticate(...).refresh()`, `invalidate().refresh(...)`).
