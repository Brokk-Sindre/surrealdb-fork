# CLI/server usage aligned to code

## Commands surfaced by CLI

The CLI subcommands include `start`, `sql`, `import`, `export`, `version`, `upgrade`, `ml`, `isready`, `validate`, and `fix`.

## Start defaults

`start` uses:
- datastore path default: `memory`
- bind default: `127.0.0.1:8000`
- initial credentials via `--username/--password`

Examples:

```bash
surreal start --username root --password root memory
surreal start --username root --password root --bind 0.0.0.0:8000 memory
```

## SQL shell defaults/options

`surreal sql` supports:
- `--endpoint` (default `ws://localhost:8000`)
- auth: `--username/--password` or `--token`
- `--auth-level root|namespace|database`
- `--namespace` + `--database`
- output flags: `--pretty`, `--json`, `--multi`

Example:

```bash
surreal sql \
  --endpoint ws://localhost:8000 \
  --username root --password root --auth-level root \
  --namespace app --database main --pretty
```

## Capability flags relevant to 3.x usage

Capabilities are exposed as startup flags in `start` options. High-value flags for newer features:

- Experimental gates:
  - `--allow-experimental files`
  - `--deny-experimental files`
- Arbitrary query control by user group (`guest`, `record`, `system`):
  - `--allow-arbitrary-query ...`
  - `--deny-arbitrary-query ...`

Practical example for API-only guest access style setups:

```bash
surreal start --username root --password root \
  --deny-arbitrary-query guest \
  memory
```
