# Verification Boundaries

## In-repo verification policy

Authoritative guidance in this skill must be supported by this repository's code/tests.
Preferred evidence roots:
- `surrealdb/core/src/**`
- `surrealdb/server/src/**`
- `surrealdb/src/**`
- `surrealdb/tests/**`
- `tests/**`
- `language-tests/tests/**`

## Explicitly out of scope by default

Unless the user explicitly requests external verification:
- SurrealDB Cloud operational setup/management
- Non-Rust SDK behavior not validated in this repo
- External projects not vendored here (for example `surqlize`, `surreal-sync`)

## Required response behavior for out-of-scope asks

1. State that the requested topic is outside in-repo verification scope.
2. Avoid prescriptive, authoritative instructions for that topic.
3. Offer a next step: external-code verification on explicit user approval.
4. Keep any external links labeled as non-verified pointers.

## Prohibited patterns

- Citing blog posts/marketing copy as proof.
- Claiming "production-ready", "stable", or release status without in-repo evidence.
- Presenting external SDK/project APIs as verified when no code path in this repo confirms them.
