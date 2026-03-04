# Computed Fields (Repo-Verified)

Verification sources:
- `surrealdb/core/src/sql/statements/define/field.rs`
- `language-tests/tests/language/statements/define/field/nested_computed_fields.surql`
- `language-tests/tests/language/statements/define/field/value_reference_with_computed.surql`
- `language-tests/tests/language/statements/define/field/value_reference.surql`

## Syntax

```sql
DEFINE FIELD <field> ON <table> COMPUTED <expr>;
DEFINE FIELD <field> ON <table> TYPE <type> COMPUTED <expr>;
```

## Typical patterns

```sql
DEFINE FIELD since ON license TYPE datetime DEFAULT time::now();
DEFINE FIELD valid ON license COMPUTED time::now() - since < 2y;
```

```sql
DEFINE FIELD full_name ON person COMPUTED string::concat(first, " ", last);
```

## Notes from language tests

- computed expressions are exercised in field-definition coverage
- computed + reference interactions are covered (`value_reference_with_computed.surql`)
- nested computed field behavior is covered (`nested_computed_fields.surql`)

## Migration-oriented note

Use `COMPUTED` field definitions for derived-read expressions in this codebase.
