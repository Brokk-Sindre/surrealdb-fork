# File Support (Repo-Verified, Experimental)

Verification sources:
- `language-tests/tests/language/primitive/files/*.surql`
- `surrealdb/core/src/sql/statements/define/bucket.rs`
- `surrealdb/core/src/sql/statements/remove/bucket.rs`
- `surrealdb/core/src/dbs/capabilities.rs` (experimental target `files`)

## Capability requirement

File features are capability-gated:
- enable with `--allow-experimental files`

## Bucket definition

```sql
DEFINE BUCKET test BACKEND "memory";
```

Also supported by SQL surface:
- `REMOVE BUCKET <name>`

## File pointer shape

Use `file` literals with bucket path:

```sql
f"test:/a.txt"
```

## Verified file operations

Covered in language tests:
- `file::get`
- `file::head`
- `file::exists`
- `file::put`
- `file::put_if_not_exists`
- `file::copy`
- `file::copy_if_not_exists`
- `file::rename`
- `file::rename_if_not_exists`
- `file::delete`
- `file::list`

Example sequence:

```sql
DEFINE BUCKET test BACKEND "memory";
file::put(f"test:/a.txt", "abc");
file::get(f"test:/a.txt").?.to_string();
file::copy(f"test:/a.txt", "b.txt");
file::rename(f"test:/b.txt", "c.txt");
file::list("test");
file::delete(f"test:/c.txt");
```

## Troubleshooting

- `method not allowed`/capability error: start server with `--allow-experimental files`.
- `NONE` from `file::get(...)`: object is missing or path differs from expected bucket/key.
