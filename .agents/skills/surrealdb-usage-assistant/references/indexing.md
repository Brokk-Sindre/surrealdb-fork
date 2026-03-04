# Indexing / Search / Vector (Repo-Verified)

Verification sources:
- `language-tests/tests/language/statements/define/index/*.surql`
- `language-tests/tests/language/functions/search/*.surql`
- `surrealdb/core/src/sql/statements/define/index.rs`
- `surrealdb/core/src/sql/statements/rebuild.rs`

## Index definition surface

Covered by statement tests:
- standard indexes (`FIELDS` / `COLUMNS`)
- `UNIQUE` indexes
- multi-field indexes
- `IF NOT EXISTS` / redefinition behavior
- `CONCURRENTLY`
- HNSW vector indexes

Examples:

```sql
DEFINE INDEX idx_email ON user FIELDS email UNIQUE;
DEFINE INDEX idx_acct_email ON user FIELDS account, email UNIQUE;
DEFINE INDEX idx_conc ON user FIELDS email CONCURRENTLY;
```

## Vector/HNSW indexes

From index/search tests:

```sql
DEFINE INDEX hnsw_pts ON pts
  FIELDS point
  HNSW DIMENSION 4 DIST EUCLIDEAN TYPE F32 EFC 500 M 12;
```

Query pattern in tests:

```sql
SELECT id, vector::distance::knn() AS distance
FROM test
WHERE embedding <|2,100|> $qvec;
```

## Full-text + ranking surface

Search function tests exercise:
- `FULLTEXT ... BM25`
- ranking via `search::score(...)`
- score fusion workflows (`search::rrf`, `search::linear`) in function-level tests

Example setup:

```sql
DEFINE ANALYZER simple TOKENIZERS blank,class;
DEFINE INDEX idx_text ON TABLE test FIELDS text FULLTEXT ANALYZER simple BM25;
```

## Rebuild support

From core statement surface:
- `REBUILD INDEX ... ON ...`

Use when index metadata exists and reindexing is required.

## Practical notes

1. Prefer explicit index names and avoid relying on implicit planner behavior.
2. For vector search, keep `DIMENSION` aligned with embedding size.
3. Use language-test-backed syntax variants (`FIELDS`/`COLUMNS`, `CONCURRENTLY`, HNSW params).
