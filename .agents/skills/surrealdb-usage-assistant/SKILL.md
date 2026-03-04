---
name: surrealdb-usage-assistant
description: Use this skill when working with SurrealDB usage and integrations that can be verified from this repository: SurrealQL statements/functions, CLI/server usage, HTTP/WS RPC, Rust SDK usage, DEFINE API, computed fields, references, files, indexing/search, transactions, GraphQL config surface, and Surrealism module flows. Do not use it for SurrealDB engine internals.
---

# SurrealDB Usage Assistant (Repo-Verified)

This skill is strictly grounded in this repository's source and tests.

Verification-first contract:
- Only present behavior that can be traced to in-repo code/tests.
- If something is not verifiable in this repo, say so explicitly.
- Do not use blog posts or external marketing docs as evidence.

Primary evidence roots:
- `surrealdb/core/src/**`
- `surrealdb/server/src/**`
- `surrealdb/tests/api_integration/**`
- `tests/**`
- `language-tests/tests/**`

## Scope boundaries

Out of scope for authoritative guidance unless external code verification is explicitly requested:
- SurrealDB Cloud operations and account/project management.
- Non-Rust SDK specifics not backed by in-repo code/tests.
- External ecosystem projects not vendored in this repo (for example `surqlize`, `surreal-sync`).

For boundary handling rules, read `references/verification-boundaries.md`.

## Step 1: Identify interface

| Interface | Use when |
|---|---|
| CLI (`surreal start/sql/import/export/...`) | Server operation, shell workflows, scripting |
| HTTP SQL (`POST /sql`) | Stateless query execution |
| RPC (`/rpc` via HTTP/WS) | JSON-RPC methods, live/session/multiplexing flows |
| Custom API (`/api/{ns}/{db}/{path}`) | `DEFINE API` route execution |
| Rust SDK (`surrealdb` crate) | Application integration in Rust |
| GraphQL endpoint/config | GraphQL endpoint behavior + `DEFINE CONFIG GRAPHQL` surface |
| Surrealism modules | Module packaging and `DEFINE MODULE` runtime calls |

## Step 2: Establish context (always)

Always make these explicit before operational queries:
1. Connect to endpoint/engine.
2. Authenticate (if remote).
3. Select namespace/database (`USE` or headers / SDK selectors).

## Step 3: Route to minimal references

Read only files needed for the user request.

### Core references
- CLI/server behavior: `references/cli-and-server.md`
- HTTP/WS/RPC behavior: `references/http-and-ws.md`
- Rust SDK behavior: `references/rust-sdk.md`
- Troubleshooting: `references/troubleshooting.md`
- Verification boundaries: `references/verification-boundaries.md`

### SurrealQL and feature references
- Statement/function coverage map: `references/surrealql-surface.md`
- Computed fields: `references/computed-fields.md`
- Record references: `references/record-references.md`
- DEFINE API: `references/define-api.md`
- File support: `references/file-support.md`
- Indexing/search/vector: `references/indexing.md`
- Client-side transactions: `references/client-side-transactions.md`
- GraphQL endpoint/config: `references/graphql.md`
- Surrealism modules: `references/surrealism.md`

### External boundary stubs
- `references/surqlize.md`
- `references/surreal-sync.md`

## Response rules

1. State what is verified and cite file paths.
2. Name transport-specific limits (for example WS-only RPC tx methods).
3. Mark deprecated RPC methods when relevant.
4. Avoid implicit NS/DB context.
5. For out-of-scope areas, follow `references/verification-boundaries.md` exactly.
