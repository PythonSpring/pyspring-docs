# Properties

The `Properties` class in PySpring provides a way to manage application-specific configurations. It allows you to load properties from a file and inject them into components.

## Key Features

*   **Configuration Management**:  `Properties` classes provide a structured way to manage application settings. They are loaded from a properties file, typically in JSON or YAML format.
*  **Unique Identifiers**: Each `Properties` class has a unique key (`__key__`) that is used to identify it within the application. This key must be set as a class-level attribute in your `Properties` subclass.
*   **Automatic Loading and Injection**: PySpring automatically loads properties during the application initialization process and injects them into components where they are needed.
*   **File Format Support**: Properties can be loaded from JSON or YAML files. The `_PropertiesLoader` class handles the parsing of these files.
*   **Validation**: The `Properties` class uses `Pydantic` for data validation. This ensures that the loaded properties conform to the expected schema.
*  **Singleton Scope**: Properties are typically loaded as singletons within the application context and are shared across the application.

## How to Define Properties

To define your properties, you need to create a class that inherits from `Properties` and set the `__key__` class variable . This key must be a unique string that will be used to identify this `Properties` class.

```py
from py_spring_core import Properties
from pydantic import Field

class AppConfigProperties(Properties):
    __key__ = "app_config" # Unique key for this properties class
    app_name: str
    version: str = Field(default="0.1.0")
    log_level: str = "INFO"
```

In the above example:

-   `AppConfigProperties` inherits from the base `Properties` class.
-   `__key__` is set to `"app_config"`, which is used to identify this specific set of properties.
-   `app_name`, `version`, and `log_level` are properties defined using Pydantic's field syntax.

How Properties are Loaded
-------------------------

The `_PropertiesLoader` class is used to load properties from a file.

1.  During application startup, the `PySpringApplication` initializes an `ApplicationContext` that loads the properties.
2.  The application scans for all classes that inherit from `Properties`.
3.  The properties file path is defined in the application config (e.g., `./application-properties.json` or `./application-properties.yaml`).
4.  The loader attempts to read and parse the file based on its extension (`.json`, `.yaml`, or `.yml`).
5.  The loader validates the properties against the classes that inherit from `Properties`, ensuring the keys match what has been defined in the application, and raises an error if validation fails.
6.  Once loaded, the properties are stored in the `ApplicationContext` and are accessible for dependency injection.

How to Use Properties
---------------------

### Dependency Injection

Once properties are loaded, you can inject them into your components using type annotations.

```py
from py_spring_core import Component, Properties
from .app_config_properties import AppConfigProperties

class MyComponent(Component):
    app_config: AppConfigProperties

    def post_construct(self):
        print(f"App Name: {self.app_config.app_name}")
        print(f"Log Level: {self.app_config.log_level}")`
```
In this example:

-   The `MyComponent` class has a type annotation `app_config: AppConfigProperties`, where `AppConfigProperties` is the properties class defined above.
-   The application context automatically injects an instance of `AppConfigProperties` into the `app_config` attribute of the `MyComponent` instance.
-   The properties can then be accessed within the `MyComponent`'s methods.

### Accessing Properties Directly

You can also access properties directly from the `_PropertiesLoader` if needed. Note, this is typically not necessary if you are using dependency injection.

```py
from py_spring_core.core.entities.properties.properties_loader import _PropertiesLoader

app_config = _PropertiesLoader.get_properties("app_config")

if app_config:
    print(f"Direct access: App Name: {app_config.app_name}")`

```
## Configuration File Examples
---------------------------

### JSON

A JSON properties file might look like this. The key `app_config` must match the key defined in the `Properties` subclass.


```json 
{
  "app_config": {
    "app_name": "My Cool App",
    "version": "1.2.3",
    "log_level": "DEBUG"
  }
}
```

### YAML

A YAML properties file might look like this. The key `app_config` must match the key defined in the `Properties` subclass.
```yaml
app_config:
  app_name: My Cool App
  version: 1.2.3
  log_level: DEBUG`
```

## Important Considerations
------------------------

-   **Key Uniqueness**: Ensure that each `Properties` class has a unique `__key__` to avoid conflicts.
-   **File Path**: The properties file path must be correctly configured in the application's configuration file.
-   **File Format**: PySpring supports both JSON and YAML formats for properties files.
-   **Validation**: Ensure that your properties files match the structure defined in your `Properties` classes. Pydantic will enforce data types and raise errors if there are mismatches.
-   **Dependency Injection**: For loose coupling, use dependency injection to make your properties available to components, which is typically preferred over directly accessing properties from `_PropertiesLoader`.
-   **Error Handling**: Be aware of the exceptions that can be raised during properties loading, such as `InvalidPropertiesKeyError`, and `TypeError` if the properties are not found or are not properly configured.

By using the `Properties` class, you can effectively manage your application's configurations in a structured and maintainable way.