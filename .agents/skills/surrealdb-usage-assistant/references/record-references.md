# Record References (Repo-Verified)

Verification sources:
- `language-tests/tests/language/reference/*.surql`
- `language-tests/tests/language/statements/define/field/value_reference.surql`
- `language-tests/tests/language/statements/define/field/value_reference_with_computed.surql`

## Reference field definitions

```sql
DEFINE FIELD owner ON license TYPE record<person> REFERENCE;
```

Array references are also covered:

```sql
DEFINE FIELD comics ON person TYPE option<array<record<comic_book>>> REFERENCE;
```

## Reverse traversal

Language tests cover reverse lookup syntax using `<~`:

```sql
SELECT *, <~person AS owners FROM comic_book;
SELECT *, <~person.{ id, name } AS owners FROM comic_book;
```

## Reference + computed pattern

```sql
DEFINE FIELD owner ON license TYPE record<person> REFERENCE;
DEFINE FIELD licenses ON person COMPUTED <~license;
```

## Deletion constraints

Reference tests include on-delete policy scenarios (reject/unset/cleanup).
If deletes fail, inspect reference constraints and inbound links first.
