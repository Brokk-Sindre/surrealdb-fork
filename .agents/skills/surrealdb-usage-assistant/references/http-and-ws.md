# HTTP / WS / API / GraphQL patterns

## HTTP SQL endpoint

Use `POST /sql` with:
- `NS` header
- `DB` header
- Basic auth (common pattern in integration tests)

```bash
curl -sS -X POST 'http://127.0.0.1:8000/sql' \
  -u 'root:root' \
  -H 'NS: app' -H 'DB: main' \
  -H 'Accept: application/json' \
  --data-binary 'SELECT * FROM person;'
```

## WebSocket RPC endpoint

Use `ws://<addr>/rpc`.

Common RPC flow:
1. `signin`
2. `use` (ns/db)
3. `query`
4. optional `live` (or `LIVE SELECT ...` via `query`)

This is session-oriented; do not treat it as stateless request/response.

## Custom API endpoint surface

Server route is:
- `/api/{ns}/{db}/{*path}`

`DEFINE API` definitions are exercised extensively in language tests, and can be invoked in-query with `api::invoke("/path")`.

External usage pattern:

```text
http://localhost:8000/api/<namespace>/<database>/<endpoint-path>
```

## GraphQL endpoint

GraphQL route is `/graphql` (server feature-gated at compile time; enabled in default feature sets used by this repo).

Operational requirement from server code path:
- session must resolve namespace + database before request execution.
