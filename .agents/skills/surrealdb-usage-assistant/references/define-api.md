# DEFINE API (Repo-Verified)

Verification sources:
- `surrealdb/core/src/sql/statements/define/api.rs`
- `surrealdb/core/src/sql/statements/define/config/api.rs`
- `surrealdb/core/src/api/request.rs`
- `surrealdb/core/src/api/invocation.rs`
- `surrealdb/core/src/api/middleware/req.rs`
- `surrealdb/core/src/api/middleware/res.rs`
- `surrealdb/server/src/ntw/api.rs`
- `language-tests/tests/api/**`
- `language-tests/tests/language/statements/define/api/*`
- `tests/http_integration.rs`

## Core shape

```sql
DEFINE API "/path"
  FOR get THEN {
    { status: 200, body: { ok: true } };
  };
```

Supported in test coverage:
- method blocks (`get`, `post`, `put`, `patch`, `delete`, `trace`, `any`)
- path variants (static, dynamic, wildcard/rest)
- middleware blocks
- permission clauses
- `OVERWRITE` / `IF NOT EXISTS`

## Config-level API middleware and permissions

From config grammar/tests:

```sql
DEFINE CONFIG API MIDDLEWARE fn::db_middleware();
DEFINE CONFIG API PERMISSIONS FULL;
```

## Route and request object examples

```sql
DEFINE API "/users/:id" FOR get THEN {
  { status: 200, body: { id: $request.params.id } };
};
```

```sql
DEFINE API "/custom" FOR any
  MIDDLEWARE fn::my_middleware("arg")
  PERMISSIONS FULL
  THEN {
    { status: 200, body: { ok: true } };
  };
```

Builtin middleware/function coverage appears in tests under `language-tests/tests/api/middleware/*`.

`$request` fields are implemented in `core/src/api/request.rs`:
- `$request.method`
- `$request.body`
- `$request.headers`
- `$request.params`
- `$request.query`
- `$request.context`
- `$request.request_id`

## Response body shape over HTTP routes

For external HTTP route execution (`/api/{ns}/{db}/{path}`), the final response body must be
`None`, bytes, or string. This is enforced in `server/src/ntw/api.rs`.

Practical rule:
- if your handler returns object/array values, use response serialization middleware
  (`api::res::body("json")`, `api::res::body("cbor")`, etc.) so the body becomes bytes.
- for fully manual responses, use `raw: true` and return bytes/string explicitly
  (covered in `language-tests/tests/api/response/raw_response.surql`).

## Query params are string-typed

`$request.query` is `BTreeMap<String, String>` in `core/src/api/request.rs`.

Practical rule:
- cast query params when numeric semantics are needed.
- example:

```sql
LIMIT <int>($request.query.limit OR "20")
START <int>($request.query.offset OR "0")
```

## Request body parsing preconditions

`api::req::body(...)` behavior is enforced by `core/src/api/middleware/req.rs`:
- `api::req::body("json")` validates `Content-Type: application/json`.
- the request body must be binary input for parsing middleware.
- `api::req::body("auto")` requires content-type and rejects unsupported types.

See tests:
- `language-tests/tests/api/body/req_serialization.surql`
- `language-tests/tests/api/errors/validation.surql`

## Method dispatch and middleware order

Method-specific handlers and middleware are supported and tested:
- one route can define both `FOR get` and `FOR post` handlers.
- middleware order is DB-level -> route `FOR any` -> method-specific `FOR <method>`.

Evidence:
- `language-tests/tests/api/methods/method_specific.surql`
- `language-tests/tests/api/middleware/execution_order.surql`
- `language-tests/tests/api/methods/fallback.surql`

## Invocation

In-SQL invocation is covered in API test suites:

```sql
RETURN api::invoke("/path", { method: "get" });
```

External invocation goes through:
- `/api/{namespace}/{database}/{path}`

## Troubleshooting signals from tests

- invalid path forms fail (for example missing leading slash)
- missing handlers/middleware validation errors are explicit
- permission ordering/override behavior is tested (`api/permissions/setup.surql`)

## Claims intentionally not adopted

The following claims are intentionally excluded from this skill because they are contradicted by
or not established in this repository:
- "method-specific middleware is unreliable, always use only `FOR any`"
- "`/import` splits purely on semicolons, so `LET` inside API handlers must be avoided"
- parser-wide blanket claims like "IF ... THEN ... ELSE in API handler objects is unsupported"
