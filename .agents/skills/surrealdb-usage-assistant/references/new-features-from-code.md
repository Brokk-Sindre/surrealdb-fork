# Newer feature patterns validated by repository code

This file captures usage patterns that are explicitly present in this repository's source/tests.

## 1) DEFINE API + middleware + invocation

Verified in `language-tests/tests/api/**`:
- `DEFINE API "/path" FOR <method> THEN { ... }`
- method-specific handlers (`FOR get`, `FOR post`, etc.)
- `MIDDLEWARE` with builtin and custom `DEFINE FUNCTION` middleware
- in-query invocation using `api::invoke("/path", {...})`

Use this when users ask to keep endpoint logic in SurrealQL.

## 2) Computed fields (`COMPUTED`) and record references (`REFERENCE`)

Verified in `language-tests/tests/language/reference/*.surql` and related files:
- computed field definitions: `DEFINE FIELD ... COMPUTED ...`
- reverse traversal/computed reference usage with `<~table`
- reference constraints like `REFERENCE ON DELETE REJECT/UNSET/THEN (...)`

Use this for schema-level derived data and bidirectional link behavior.

## 3) File support with buckets and file pointers

Verified in `language-tests/tests/language/primitive/files/*.surql`:
- feature gate required: `allow-experimental = ["files"]`
- bucket creation: `DEFINE BUCKET <name> BACKEND "memory"`
- file pointer literal: `f"bucket:/path"`
- methods/functions: `.get()`, `.put()`, `.exists()`, `.copy()`, `.rename()`, `file::list(...)`, etc.

If files fail unexpectedly, check experimental capability flags first.

## 4) Transaction semantics (SurrealQL)

Verified in transaction language tests:
- `BEGIN; ... COMMIT;`
- `CANCEL;`
- failure propagation on duplicate keys/errors
- `THROW` within transactions

Use explicit `BEGIN/COMMIT/CANCEL` snippets when guiding SurrealQL-only workflows.

## 5) Client-side transactions (SDK)

Verified in SDK method implementations:
- transaction object with `begin`, `commit`, `cancel`
- transaction-scoped query/CRUD methods (`tx.query`, `tx.create`, ...)

Prefer this for application-managed transactional control flow.

## 6) Search/index patterns

Verified in language tests:
- Full-text indexes with analyzers and `@...@` operators
- HNSW KNN index definitions and vector distance queries

Use these patterns for retrieval-focused use cases; include concrete index definitions and query operators.
