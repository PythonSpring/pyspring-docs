# Environment Variables

After this page, you'll know how to use environment variables in your properties files so that secrets and environment-specific values aren't hardcoded.

## The problem

In the [Properties](../tutorial/properties.md) tutorial, all values are static:

```yaml
database:
  host: localhost
  port: 5432
  name: mydb
```

This works for local development, but in production you don't want to commit database credentials or environment-specific hosts into your config files.

## The `${VAR}` syntax

PySpring supports `${VAR_NAME}` placeholders in properties files. At startup, placeholders are resolved from environment variables **after** YAML/JSON parsing but **before** Pydantic validation.

```yaml
database:
  host: ${DB_HOST}
  port: ${DB_PORT}
  name: ${DB_NAME}
```

If the environment variable is set, its value replaces the placeholder. If it's not set and no default is provided, PySpring raises an `EnvVarNotFoundError` at startup — so you catch missing config immediately.

## Default values

Use `${VAR_NAME:default}` to provide a fallback when an environment variable isn't set:

```yaml
database:
  host: ${DB_HOST:localhost}
  port: ${DB_PORT:5432}
  name: ${DB_NAME}
```

| Pattern | Behavior |
|---|---|
| `${VAR}` | Use the env var value. Raise `EnvVarNotFoundError` if not set. |
| `${VAR:default}` | Use the env var value if set, otherwise use `"default"`. |
| `${VAR:}` | Use the env var value if set, otherwise use an empty string. |

!!! tip
    Use defaults for values that have sensible fallbacks (hosts, ports) and omit defaults for values that **must** be provided (API keys, secrets). This way, a missing secret fails fast at startup instead of silently using a wrong value.

## Partial substitution

You can mix placeholders with static text, and use multiple placeholders in a single value:

```yaml
database:
  url: postgresql://${DB_HOST:localhost}:${DB_PORT:5432}/${DB_NAME}
```

With `DB_HOST=prod-db`, `DB_PORT=3306`, and `DB_NAME=myapp`:

```
postgresql://prod-db:3306/myapp
```

## Full example

### Properties class

```python
from py_spring_core import Properties


class DatabaseProperties(Properties):
    __key__ = "database"
    host: str
    port: int
    name: str
```

### Config file

=== "YAML"

    ```yaml
    database:
      host: ${DB_HOST:localhost}
      port: ${DB_PORT:5432}
      name: ${DB_NAME}
    ```

=== "JSON"

    ```json
    {
      "database": {
        "host": "${DB_HOST:localhost}",
        "port": "${DB_PORT:5432}",
        "name": "${DB_NAME}"
      }
    }
    ```

### Running the application

```console
$ DB_NAME=myapp python main.py
```

Since `DB_HOST` and `DB_PORT` have defaults, only `DB_NAME` is required. If you omit `DB_NAME`, the application fails at startup with:

```
EnvVarNotFoundError: [ENV VAR NOT FOUND] Environment variable 'DB_NAME' is not set and no default was provided
```

!!! note "Type coercion"
    All resolved values are **strings**, because `EnvVarResolver` performs string substitution. Pydantic validation runs after resolution and will coerce types automatically in the default lax mode — for example, `"5432"` becomes `int 5432`, and `"true"` becomes `bool True`. This works because `Properties` inherits from `BaseModel` without strict mode enabled.

## Nested and list values

Environment variable resolution works recursively through nested dicts and lists:

```yaml
app:
  servers:
    - host: ${PRIMARY_HOST}
      port: 8080
    - host: ${SECONDARY_HOST:backup.local}
      port: 8081
  database:
    connection:
      url: ${DB_URL}
```

Non-string values (integers, booleans, floats, `null`) pass through unchanged — only string values containing `${...}` are resolved.

## How it works

The `EnvVarResolver` class handles all placeholder resolution. It is called internally by the properties loader — you don't need to use it directly.

The resolution pipeline is:

1. Properties file is parsed as YAML/JSON into a Python dict
2. `EnvVarResolver.resolve_dict()` recursively walks the dict and resolves all `${VAR}` placeholders in string values
3. The resolved dict is validated by Pydantic against your `Properties` class

## Recap

Environment variable resolution lets you externalize configuration.

* Use `${VAR}` for required environment variables
* Use `${VAR:default}` for optional ones with fallbacks
* Use `${VAR:}` for an empty string default
* Multiple placeholders and partial substitution are supported
* Resolution happens automatically — no code changes needed
* Missing required variables fail fast at startup with `EnvVarNotFoundError`
