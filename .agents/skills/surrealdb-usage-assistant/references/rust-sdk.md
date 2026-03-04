# Rust SDK usage (code-grounded)

## Connect options

`surrealdb::engine::any::connect(...)` supports remote and local engines:
- remote: `ws://`, `wss://`, `http://`, `https://`
- local: `mem://` (and `memory` alias), `rocksdb://...`, `surrealkv://...`, `tikv://...`

## Baseline remote flow

```rust
use surrealdb::engine::any::connect;
use surrealdb::opt::auth::Root;

let db = connect("ws://localhost:8000").await?;
db.signin(Root { username: "root", password: "root" }).await?;
db.use_ns("app").use_db("main").await?;

let mut res = db.query("CREATE person:alice SET name = $name;")
    .bind(("name", "Alice"))
    .await?;
```

## Client-side transaction flow (implemented in SDK)

The SDK exposes explicit transaction lifecycle methods:
- `db.begin().await?`
- perform ops via transaction handle (`tx.query(...)`, `tx.create(...)`, etc.)
- `tx.commit().await?` or `tx.cancel().await?`

```rust
let tx = db.begin().await?;
tx.query("CREATE person:one;").await?;
tx.query("CREATE person:two;").await?;
let db = tx.commit().await?;
```

Use this for multi-step app logic where commit/cancel decisions happen in application code.
