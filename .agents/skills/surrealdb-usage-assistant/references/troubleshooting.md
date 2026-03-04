# Troubleshooting (Repo-Verified)

Verification sources:
- `tests/http_integration.rs`
- `tests/ws_integration.rs`
- `surrealdb/tests/api_integration/basic.rs`
- `surrealdb/tests/api_integration/session_isolation.rs`
- `surrealdb/core/src/rpc/protocol.rs`
- `surrealdb/server/src/rpc/http.rs`
- `surrealdb/server/src/rpc/websocket.rs`
- `surrealdb/core/src/api/request.rs`
- `surrealdb/core/src/api/middleware/req.rs`
- `surrealdb/core/src/api/middleware/res.rs`
- `surrealdb/server/src/ntw/api.rs`
- `language-tests/tests/api/**`

## Transport mismatch

| Symptom | Likely cause | Verified fix |
|---|---|---|
| WS connect/query flow fails on `/sql` | wrong endpoint | use `/rpc` for WS RPC |
| RPC tx methods fail over HTTP | HTTP RPC doesn't implement tx methods | use WS for `begin/commit/cancel` |
| live query does not stream | stateless transport | use WS and keep socket open |

## RPC session multiplexing issues

| Symptom | Likely cause | Verified fix |
|---|---|---|
| `attach` fails | missing/invalid request `session` UUID | send top-level `session` UUID in request envelope |
| `sessions` list empty unexpectedly | only default session exists | call `attach` for named sessions first |
| state bleed between clients | using same session unintentionally | assign distinct session UUIDs and use `session` field per request |

## RPC transaction issues

| Symptom | Likely cause | Verified fix |
|---|---|---|
| `method not found` for `begin`/`commit`/`cancel` | HTTP RPC context | switch to WS RPC |
| `Transaction not found` | wrong or already-removed txn UUID | keep txn UUID returned from `begin`; commit/cancel exactly once |
| query not in transaction | missing top-level `txn` field | include `txn` UUID on transactional query requests |

## Token lifecycle/auth errors

| Symptom | Likely cause | Verified fix |
|---|---|---|
| refresh fails | missing/invalid refresh component | authenticate with token pair that includes refresh token |
| refresh fails after revoke | token grant revoked | sign in again to mint new token pair |
| auth appears cleared after invalidate/reset | expected behavior | re-authenticate and re-`use` ns/db |

## SDK session isolation surprises

| Symptom | Likely cause | Verified fix |
|---|---|---|
| clone sees different data/context | cloned client has isolated session state | explicitly set NS/DB and auth per cloned client |
| uncommitted tx not visible in other client | transactional isolation | commit before expecting visibility |
| `set` var not visible in another clone | session-local variables | set variables on each client session |

## `run` method issues

| Symptom | Likely cause | Verified fix |
|---|---|---|
| `run` argument error | wrong `(name, version, args)` shape | send string name, optional version string, optional args array |
| `mod::...` call rejected | surrealism capability disabled | start server with `--allow-experimental surrealism` |
| `silo::...` call rejected | missing/invalid semantic version | provide `major.minor.patch` version string |
| function call mismatch | wrong arg type/count | match function signature (`ws_integration` has failing examples for wrong args) |

## DEFINE API endpoint issues

| Symptom | Likely cause | Verified fix |
|---|---|---|
| HTTP API route returns body-shape error for object/array | final HTTP response body must be None/bytes/string | add `api::res::body("json")` (or another serializer) so body is encoded bytes |
| `api::req::body("json")` fails with 400 | missing/incorrect `Content-Type` or non-bytes request body | send `Content-Type: application/json` and binary JSON payload |
| `api::req::body("auto")` fails | missing/unsupported content type | set supported `Content-Type` explicitly |
| numeric filter/pagination logic fails | query params are strings | cast params, for example `<int>($request.query.limit OR "20")` |
| method-specific behavior not running as expected | request method mismatches handler selection | verify `FOR <method>` coverage and, if needed, add `FOR any` fallback |
| unexpected middleware sequence | incorrect expectations about composition order | account for execution order: DB-level -> route `FOR any` -> method-specific |

## API/file/indexing quick checks

- API route path must begin with `/`; invalid paths are rejected in API tests.
- File operations require `--allow-experimental files`.
- For index expectations, verify syntax and planner behavior with tested index patterns in `language-tests`.
