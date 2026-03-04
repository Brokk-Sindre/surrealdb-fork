# Troubleshooting SurrealDB interaction

## Connection/path mismatches

- SQL over HTTP must use `/sql`.
- RPC over WS must use `/rpc`.
- Custom API routes must use `/api/{ns}/{db}/{path}`.
- GraphQL uses `/graphql`.

## Auth/context mismatches

- If auth succeeds but queries fail, ensure namespace/database are set.
- Namespace/database-level auth requires matching `--auth-level` and supplied ns/db.

## Capability/permission issues

- If file pointers/buckets fail, ensure files experimental capability is enabled (`--allow-experimental files`).
- If guest/record users cannot execute ad-hoc queries, inspect `--deny-arbitrary-query` / `--allow-arbitrary-query` startup flags.
- If route-level requests fail, inspect HTTP/RPC capability allow/deny flags.

## API endpoint issues

- Validate `DEFINE API` path starts with `/`.
- Confirm handler exists for requested method (or `FOR any` fallback).
- Check middleware function signatures match expected API middleware calling convention.

## Realtime / transaction issues

- Keep WS sessions open for live query notifications.
- For SDK transactions, ensure every transaction is explicitly committed or cancelled.
