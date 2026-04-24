# Properties

After this page, you'll know how to define type-safe configuration for your application using `Properties`.

In PySpring, configuration isn't just a dictionary — it's a validated Pydantic model. You define the shape of your config, and PySpring loads and validates it automatically.

## Define a Properties class

```python
from py_spring_core import Properties


class DatabaseProperties(Properties):
    __key__ = "database"
    host: str
    port: int
    name: str
```

The `__key__` tells PySpring which section of your config file maps to this class. So with this config file:

```json
{
  "database": {
    "host": "localhost",
    "port": 5432,
    "name": "mydb"
  }
}
```

PySpring will load the `"database"` section and create a validated `DatabaseProperties` instance.

!!! tip
    Because `Properties` extends Pydantic's `BaseModel`, you get full validation for free. If `port` is a string instead of an int, you'll get a clear error at startup.

## Use default values

You can provide defaults for optional fields:

```python
from py_spring_core import Properties
from pydantic import Field


class AppConfigProperties(Properties):
    __key__ = "app_config"
    app_name: str
    version: str = Field(default="0.1.0")
    log_level: str = "INFO"
```

If `version` or `log_level` aren't in the config file, the defaults are used.

## Inject Properties into a Component

Properties are injected the same way as any other dependency — declare the type:

```python
from py_spring_core import Component, Properties


class DatabaseProperties(Properties):
    __key__ = "database"
    host: str
    port: int
    name: str


class DatabaseService(Component):
    database_properties: DatabaseProperties  # Auto-injected!

    def post_construct(self):
        print(f"Connecting to {self.database_properties.host}:{self.database_properties.port}")
```

PySpring sees the type hint, finds the matching `Properties` instance, and injects it. No manual wiring needed.

## Supported file formats

PySpring supports both JSON and YAML for configuration files:

=== "JSON"

    ```json
    {
      "database": {
        "host": "localhost",
        "port": 5432,
        "name": "mydb"
      }
    }
    ```

=== "YAML"

    ```yaml
    database:
      host: localhost
      port: 5432
      name: mydb
    ```

!!! note
    The file format is detected from the file extension (`.json`, `.yaml`, or `.yml`).

## Recap

Properties give you type-safe, validated configuration.

* Define a `Properties` class with `__key__` pointing to a config section
* Fields are validated by Pydantic at startup
* Inject into components using type hints
* Supports JSON and YAML
* Use `Field(default=...)` for optional values

Next, let's learn about **Dependency Injection** — the mechanism that makes all this wiring work automatically.
