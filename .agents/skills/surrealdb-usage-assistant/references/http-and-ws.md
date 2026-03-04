# HTTP / WS / RPC (Repo-Verified)

Verification sources:
- `surrealdb/core/src/rpc/method.rs`
- `surrealdb/core/src/rpc/protocol.rs`
- `surrealdb/core/src/rpc/request.rs`
- `surrealdb/server/src/rpc/http.rs`
- `surrealdb/server/src/rpc/websocket.rs`
- `tests/http_integration.rs`
- `tests/ws_integration.rs`

## HTTP endpoints

| Method | Path | Notes |
|---|---|---|
| `GET` | `/health` | Liveness |
| `POST` | `/sql` | SurrealQL over HTTP |
| `POST` | `/import` | Import script |
| `GET` | `/export` | Export script |
| `POST` | `/signin` | Signin/auth token |
| `POST` | `/signup` | Record-access signup |
| `POST` | `/rpc` | JSON-RPC over HTTP |
| `GET/WS` | `/rpc` | WebSocket RPC upgrade |
| `*` | `/api/{ns}/{db}/{path}` | `DEFINE API` routes |
| `GET/POST` | `/graphql` | GraphQL endpoint |

## RPC request envelope

From `request.rs`:
- `id`: optional
- `method`: required string
- `params`: optional array (defaults to empty)
- `session`: optional UUID/string (session multiplexing)
- `txn`: optional UUID/string (query execution in existing transaction)

Example:
```json
{"id":1,"method":"query","params":["RETURN 1"],"session":"11111111-1111-1111-1111-111111111111"}
```

## RPC method surface (complete)

Parsed by `Method::parse` in `method.rs`.

### Core/session/auth methods
- `ping`
- `info`
- `use`
- `signup`
- `signin`
- `authenticate`
- `refresh`
- `invalidate`
- `revoke`
- `reset`
- `set` (alias: `let`)
- `unset`
- `version`

### Query/realtime methods
- `query`
- `live`
- `kill`
- `run`

### Session multiplexing methods
- `attach`
- `sessions`
- `detach`

### Transaction methods
- `begin`
- `commit`
- `cancel`

### Deprecated CRUD-style RPC methods (still dispatched)
`protocol.rs` marks these as deprecated:
- `select`, `insert`, `create`, `upsert`, `update`, `merge`, `patch`, `delete`, `relate`, `insert_relation`

## HTTP vs WS transaction behavior

From transport-specific protocol impls:
- HTTP RPC (`server/src/rpc/http.rs`): `begin`, `commit`, `cancel` return method-not-found.
- WebSocket RPC (`server/src/rpc/websocket.rs`): transaction methods are implemented.

WS transaction flow:
1. `begin` -> returns txn UUID.
2. Use txn UUID either:
   - as top-level `txn` on query-bearing requests (`query`, deprecated CRUD methods, `run` path where applicable through dispatcher), or
   - for lifecycle methods `commit`/`cancel` via `params: ["<txn-uuid>"]`.
3. `commit`/`cancel` removes tx from WS tx map.

Minimal WS tx example:
```json
{"id":1,"method":"begin","params":[]}
{"id":2,"method":"query","txn":"<uuid>","params":["CREATE t:1 SET v = 1;"]}
{"id":3,"method":"commit","params":["<uuid>"]}
```

## Session multiplexing (`attach` / `sessions` / `detach`)

From `protocol.rs` + `ws_integration.rs`:
- `attach` registers a named session keyed by request `session` UUID.
- `sessions` lists non-default session UUIDs.
- `detach` removes named session and cleans up its live queries.

Pattern:
```json
{"id":1,"method":"attach","session":"11111111-1111-1111-1111-111111111111","params":[]}
{"id":2,"method":"signin","session":"11111111-1111-1111-1111-111111111111","params":[{"user":"root","pass":"root"}]}
{"id":3,"method":"use","session":"11111111-1111-1111-1111-111111111111","params":["ns","db"]}
{"id":4,"method":"sessions","params":[]}
{"id":5,"method":"detach","session":"11111111-1111-1111-1111-111111111111","params":[]}
```

## Token lifecycle methods

From `protocol.rs`:
- `refresh(token)` validates refresh token, revokes old refresh token, issues new pair.
- `revoke(token)` revokes refresh-token grant without clearing whole session.
- `reset()` resets session and cleans up its live queries.
- `invalidate()` clears auth/session state.

## `run` method behavior

From `protocol.rs` and tests:
- Signature: `(name: string, version: string|null, args: array|null)`.
- Supports receiver patterns:
  - `fn::...` custom SurrealQL function
  - `mod::...` module call (requires experimental `surrealism` capability)
  - `silo::org::pkg::<sub>` style receiver with required semantic version string (`major.minor.patch`)
  - `ml::...` model receiver, version required
  - unprefixed/builtin functions (for example `math::abs`)
- `tests/ws_integration.rs` verifies custom `fn::...` and builtin `math::...` calls.

## Live queries

- WS supports realtime (`LQ_SUPPORT = true`); HTTP does not.
- `LIVE SELECT ...` emits async push notifications on the socket.
- `kill` removes live query registration.

## Capability gating

RPC method execution is capability-gated (`allows_rpc_method`).
`tests/ws_integration.rs` verifies deny/allow behavior such as:
- `--deny-rpc --allow-rpc version,use,attach`

## Practical rules

1. Use WS for realtime, session multiplexing, and RPC transactions.
2. Use HTTP `/sql` for stateless query execution.
3. Prefer `query` + SurrealQL over deprecated CRUD RPC methods for new flows.
