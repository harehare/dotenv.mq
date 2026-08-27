<h1 align="center">dotenv.mq</h1>

A [dotenv](https://github.com/motdotla/dotenv#rules) (`.env`) parser and serializer implemented as an [mq](https://github.com/harehare/mq) module.

## Features

- `KEY=value`, `KEY="value"`, and `KEY='value'`
- Optional leading `export ` per line (shell-style)
- Blank lines and full-line `#` comments
- Inline `# comment` after unquoted values (a `#` only starts a comment when it's the first character of the value or preceded by whitespace, so `URL=http://example.com/#frag` is preserved)
- `\n`, `\t`, `\r`, `\"`, `\\` escapes and literal embedded newlines inside double-quoted values
- Single-quoted values are taken literally — no escape processing
- Later definitions of the same key overwrite earlier ones
- `dotenv_stringify` to serialize a dict back to `.env` format

## Installation

Copy `dotenv.mq` to your mq module directory, or place it anywhere and reference it with `-L`.

```sh
cp dotenv.mq ~/.local/mq/config/
```

### HTTP Import

```sh
mq --allow-http-import -I raw 'import "github.com/harehare/dotenv.mq" | dotenv::dotenv_parse(.)' .env
```

## API

| Function | Description |
|---|---|
| `dotenv_parse(input)` | Parses the full contents of a `.env` file into a dict |
| `dotenv_parse_line(input)` | Parses a single `KEY=VALUE` line, returns `[key, value]` |
| `dotenv_stringify(dict)` | Serializes a dict into `.env` format |

Raises an error if a line is missing `=` after the key, or if a quoted value is unterminated.

## Example

Given `.env`:

```sh
# app config
export APP_NAME=myapp
PORT=8080
DEBUG=false

DB_URL="postgres://user:pass@localhost/db?sslmode=disable"
GREETING="line1\nline2"
URL=http://example.com/path#fragment
INLINE=foo # trailing comment
```

```sh
mq -I raw 'import "dotenv" | dotenv::dotenv_parse(.)' -F json .env
# => {
# =>   "APP_NAME": "myapp",
# =>   "PORT": "8080",
# =>   "DEBUG": "false",
# =>   "DB_URL": "postgres://user:pass@localhost/db?sslmode=disable",
# =>   "GREETING": "line1\nline2",
# =>   "URL": "http://example.com/path#fragment",
# =>   "INLINE": "foo"
# => }

# Extract a single value
mq -I raw 'import "dotenv" | dotenv::dotenv_parse(.) | ."PORT"' .env
# => "8080"

# Parse a single line
mq -I null 'import "dotenv" | dotenv::dotenv_parse_line("export KEY=value")'
# => ["KEY", "value"]

# Round-trip: parse then re-stringify
mq -I raw 'import "dotenv" | dotenv::dotenv_stringify(dotenv::dotenv_parse(.))' .env
```

## Compatibility

Requires [mq](https://github.com/harehare/mq) v0.6 or later.

## License

MIT
