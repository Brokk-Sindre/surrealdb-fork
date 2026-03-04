# GraphQL (Repo-Verified)

Verification sources:
- `tests/graphql_integration.rs`
- `surrealdb/core/src/sql/statements/define/config/graphql.rs`
- `language-tests/tests/language/statements/define/config/graphql/*`

## Endpoint and context

GraphQL is served from `/graphql`.
Tests use standard Surreal headers for context:
- `surreal-ns`
- `surreal-db`

## Required configuration

Integration tests show GraphQL requires config before query execution:

```sql
DEFINE CONFIG GRAPHQL AUTO;
```

Without config, GraphQL returns `NotConfigured` in tested flows.

## Verified query capabilities

From `tests/graphql_integration.rs`:
- table querying
- `_get_<table>` style lookup
- `limit`, `start`, `order`, `filter`
- auth-aware behavior with root/basic auth and bearer token
- geometry fields and geometry union fragments

Example:

```graphql
query {
  foo(filter: { val: { eq: 42 } }, order: { desc: val }, limit: 1) {
    id
    val
  }
}
```

## Config surface (SQL)

From SQL config type + language tests:
- `DEFINE CONFIG GRAPHQL AUTO|NONE`
- table controls: `TABLES INCLUDE ...` / `TABLES EXCLUDE ...`
- function controls: `FUNCTIONS AUTO|NONE`
- limits: `DEPTH <n> COMPLEXITY <n>`
- introspection toggle: `INTROSPECTION NONE`

Example:

```sql
DEFINE CONFIG OVERWRITE GRAPHQL TABLES INCLUDE foo FUNCTIONS AUTO DEPTH 20 COMPLEXITY 1000;
```

## Troubleshooting

- `NotConfigured`: define GraphQL config first.
- empty schema/query errors: ensure tables/fields are defined in selected NS/DB.
- auth errors: validate credentials/token and NS/DB context headers.
