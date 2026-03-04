---
name: surrealdb-usage-assistant
description: Use this skill when the task is to use SurrealDB itself (querying data, connecting clients, authenticating, selecting namespace/database, using CLI/HTTP/WebSocket/Rust SDK, custom API routes, computed fields, record references, file buckets, transactions, search, and troubleshooting interaction flows). Do not use this skill for modifying SurrealDB internals.
---

# SurrealDB Usage Assistant

This skill guides an AI agent on how to interact with SurrealDB using behaviors verified in this repository's code/tests.

## 1) Interface selection

Choose one primary interface:

1. **CLI shell** (`surreal sql`) for interactive exploration.
2. **HTTP SQL** (`POST /sql`) for stateless calls.
3. **WebSocket RPC** (`/rpc`) for session workflows and live updates.
4. **HTTP API routes** (`/api/{ns}/{db}/{path}`) for `DEFINE API` endpoints.
5. **Rust SDK** (`surrealdb` crate) for embedded/app integration.
6. **GraphQL** (`/graphql`) when typed GraphQL access is requested.

## 2) Required execution sequence

For most tasks:
1. Connect/start server.
2. Authenticate (user/pass, token, or auth flow per interface).
3. Set namespace/database context.
4. Execute SurrealQL/API operation.
5. For realtime or multi-step stateful flows, keep session and use WS or SDK transactions.

## 3) High-signal references (load only what is needed)

- `references/cli-and-server.md` → startup/auth/capability flags.
- `references/http-and-ws.md` → `/sql`, `/rpc`, `/api/...`, `/graphql` usage.
- `references/rust-sdk.md` → connect/auth/query plus client-side transactions.
- `references/new-features-from-code.md` → 3.x features proven by tests (DEFINE API, COMPUTED, REFERENCE, file buckets, vector/full-text, tx semantics).
- `references/troubleshooting.md` → common failures and fixes.

## 4) Response rules

- Prefer examples that are directly represented in repository tests or source.
- Make namespace/database explicit.
- For experimental features, mention required capability flags.
- For permissions/capabilities errors, provide concrete startup flag fixes.
- When uncertain, say what is verified by code and what is not.

## 5) Guardrails

- Do not invent unsupported flags/endpoints/auth levels.
- Keep defaults aligned with code (`ws://localhost:8000`, `127.0.0.1:8000`, `/sql`, `/rpc`).
- Distinguish local-engine behavior from remote-server behavior.
