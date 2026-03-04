# SurrealQL Surface Map (Repo-Verified)

Verification sources:
- `surrealdb/core/src/sql/statements/mod.rs`
- `surrealdb/core/src/sql/statements/define/mod.rs`
- `surrealdb/core/src/sql/statements/remove/mod.rs`
- `surrealdb/core/src/sql/statements/show.rs`
- `surrealdb/core/src/sql/statements/access.rs`
- `language-tests/tests/language/statements/**`
- `language-tests/tests/language/functions/**`
- `surrealdb/tests/api_integration/basic.rs`

## Statement family map

From core statement modules:
- data/manipulation: `SELECT`, `CREATE`, `INSERT`, `UPSERT`, `UPDATE`, `DELETE`, `RELATE`
- flow/control: `IF`, `FOR`, `RETURN`, `SLEEP`, `OPTION`, `LET`, `USE`
- realtime/change: `LIVE`, `KILL`, `SHOW CHANGES`
- transactions: `BEGIN`, `COMMIT`, `CANCEL`
- metadata/schema families:
  - `DEFINE ...`
  - `REMOVE ...`
  - `ALTER ...`
  - `REBUILD ...`
  - `ACCESS ...` (grant/show/revoke/purge)
  - `INFO ...`

## `DEFINE` / `REMOVE` sub-surface

From `define/mod.rs` and `remove/mod.rs`:
- `NAMESPACE`, `DATABASE`
- `TABLE`, `FIELD`, `INDEX`, `EVENT`
- `FUNCTION`, `PARAM`, `ANALYZER`
- `USER`, `ACCESS`
- `CONFIG` (including GraphQL/API config forms)
- `API`
- `BUCKET`
- `SEQUENCE`
- `MODEL`
- `MODULE`

## High-signal advanced features in tests

### GraphQL config controls
Evidence:
- `language-tests/tests/language/statements/define/config/graphql/*`

Covered patterns include:
- `DEFINE CONFIG GRAPHQL AUTO|NONE`
- `TABLES INCLUDE/EXCLUDE ...`
- `FUNCTIONS AUTO|NONE`
- `DEPTH` and `COMPLEXITY`
- `INTROSPECTION NONE`

### API config and route middleware/permissions
Evidence:
- `language-tests/tests/language/statements/define/api/*`
- `language-tests/tests/api/**`

Covered patterns include:
- `DEFINE CONFIG API MIDDLEWARE ... PERMISSIONS ...`
- `DEFINE API ... FOR <method> ... MIDDLEWARE ... PERMISSIONS ... THEN ...`
- route overwrite/if-not-exists, path matching variants, serialization/validation/error cases

### Sequence support
Evidence:
- `language-tests/tests/language/statements/define/sequence/*`
- `language-tests/tests/language/statements/remove/sequence.surql`
- `language-tests/tests/language/functions/sequence/nextval.surql`

Covered patterns:
- `DEFINE SEQUENCE ...`
- `REMOVE SEQUENCE ...`
- `sequence::nextval(...)`

### ACCESS grant/show/revoke/purge
Evidence:
- `surrealdb/core/src/sql/statements/access.rs`
- `language-tests/tests/language/statements/access/**`

Covered operations:
- `ACCESS ... GRANT ...`
- `ACCESS ... SHOW ...`
- `ACCESS ... REVOKE ...`
- `ACCESS ... PURGE ...`

### SHOW CHANGES / changefeed
Evidence:
- `surrealdb/core/src/sql/statements/show.rs`
- `surrealdb/tests/api_integration/basic.rs` (`SHOW CHANGES FOR TABLE ...`)

Covered pattern:
- `SHOW CHANGES FOR TABLE|DATABASE ... SINCE ... [LIMIT ...]`

### Related feature cross-links
- Computed fields: `computed-fields.md`
- Record references: `record-references.md`
- File support: `file-support.md`
- Indexing/search/vector: `indexing.md`
- DEFINE API detail: `define-api.md`

## Function namespace map from language tests

Top-level namespaces observed under `language-tests/tests/language/functions`:
- `array`
- `bytes`
- `crypto`
- `duration`
- `encoding`
- `geo`
- `math`
- `ml`
- `module`
- `object`
- `rand`
- `record`
- `search`
- `sequence`
- `set`
- `string`
- `type`
- `value`
- plus general function coverage files (`count.surql`, `not.surql`, `general/*`, `path_hints/*`)

## Practical use

Use this file as the first routing layer when the request asks:
- "is X statement supported?"
- "which namespace/function should I use?"
- "is this new syntax in this repo?"

Then open the focused reference file for examples and constraints.
