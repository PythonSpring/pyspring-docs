Dependency Injection with Entities in PySpring
================================================

PySpring's **dependency injection (DI)** system simplifies dependency management, improves code modularity, and promotes loose coupling between application entities. This guide covers how to use PySpring's DI system to integrate components and other application entities.

* * * * *

Key Features of PySpring's DI System
------------------------------------

### 1\. Declaring Dependencies

To inject a dependency into a component, declare an attribute with a type hint for the required dependency. PySpring automatically resolves and injects the required dependency during application initialization.

**Example:**

```py
from py_spring_core import Component
class ExampleService(Component):
    example_properties: ExampleProperties
```

Here, the `ExampleService` component declares a dependency on `ExampleProperties` by defining the `example_properties` attribute with the appropriate type annotation.

* * * * *

### 2\. Automatic Resolution

During initialization, PySpring's `ApplicationContext` automatically scans declared dependencies, resolves them, and injects the corresponding instances.

#### How It Works:

-   When a component is instantiated, the `_inject_entity_dependencies` method of the `ApplicationContext` is called.

-   If the dependency is a `Properties` class, the system retrieves the properties from the application context using the key defined in the `Properties` class.

-   If the dependency is not a `Properties` class, the system retrieves it from the component or bean collection using `get_component` or `get_bean`.

-   If the dependency cannot be resolved, an error is raised.

**Error Handling:** If a dependency cannot be resolved, `_inject_entity_dependencies` raises a `ValueError` with a descriptive error message. This ensures early detection of misconfigured dependencies.

* * * * *

### 3\. Accessing Injected Dependencies

Once injected, dependencies can be accessed directly using the declared attribute name.

**Example:**

```py
from py_spring_core import Component
class ExampleService(Component):
    example_properties: ExampleProperties

    def display_property(self):
        print(self.example_properties.value)
```

Here, `example_properties` is automatically injected, and its value is accessed using `self.example_properties.value`.

* * * * *

### 4\. Component Lifecycle Methods

PySpring supports lifecycle methods to manage component behavior:

-   `post_construct()`: Called after dependencies are injected, allowing initialization with injected resources.

-   `pre_destroy()`: Called before the component is destroyed, enabling cleanup actions.

**Example:**

```py
class ExampleService(Component):
    example_properties: ExampleProperties

    def post_construct(self):
        print("Component initialized with", self.example_properties)

    def pre_destroy(self):
        print("Cleaning up ExampleService")
```

* * * * *

### 5\. Supported Dependency Types

PySpring's DI system supports injecting the following:

-   **Component classes**

-   **Properties classes**

-   **Beans** defined within a `BeanCollection`

* * * * *

### 6\. Qualifier Support

PySpring supports qualifiers for dependency injection, allowing you to specify which implementation to inject when multiple implementations of the same interface exist.

#### Using Qualifiers:

```py
from typing import Annotated
from py_spring_core import Component

class ServiceConsumer(Component):
    # Will inject the specifically qualified implementation
    service_a: Annotated[AbstractService, "ServiceA"]
    service_b: Annotated[AbstractService, "ServiceB"]

    def post_construct(self) -> None:
        print(self.service_a.process())  # Uses ServiceA
        print(self.service_b.process())  # Uses ServiceB
```

#### Complete Example with Qualifiers:

```py
from abc import ABC, abstractmethod
from typing import Annotated
from py_spring_core import Component
from py_spring_core.core.entities.component import ComponentScope

# Abstract base class
class AbstractExample(Component, ABC):
    class Config:
        scope = ComponentScope.Singleton

    @abstractmethod
    def say_hello(self) -> str: ...

# First implementation
class ExampleA(AbstractExample):
    class Config:
        name = "ExampleA"
        scope = ComponentScope.Singleton

    def say_hello(self) -> str:
        return "Hello from Example A!"

# Second implementation
class ExampleB(AbstractExample):
    class Config:
        name = "ExampleB"
        scope = ComponentScope.Singleton

    def say_hello(self) -> str:
        return "Hello from Example B!"

# Service using both implementations
class TestService(Component):
    example_a: Annotated[AbstractExample, "ExampleA"]
    example_b: Annotated[AbstractExample, "ExampleB"]

    def post_construct(self) -> None:
        print(self.example_a.say_hello())  # Output: "Hello from Example A!"
        print(self.example_b.say_hello())  # Output: "Hello from Example B!"
```

### 7\. Component Registration Validation

PySpring now includes validation to prevent duplicate component registration and provides clear error messages for developers.

#### Preventing Duplicate Registration:

```py
from py_spring_core import Component
from py_spring_core.core.entities.component import ComponentScope

# First registration of MyService
class MyService(Component):
    class Config:
        name = "MyService"
        scope = ComponentScope.Singleton

    def do_something(self) -> str:
        return "Service is working!"

# Attempting to register the same component again will raise ValueError
try:
    class MyService(Component):  # This will raise ValueError
        class Config:
            name = "MyService"
            scope = ComponentScope.Singleton

        def do_something(self) -> str:
            return "This will never be registered!"
except ValueError as e:
    print(f"Error: {e}")  # Output: "Error: [COMPONENT REGISTRATION ERROR] Component: MyService already registered"
```

#### Key Points for Component Registration:

1. Each component must have a unique name (either explicitly set in Config or derived from the class name)
2. Attempting to register a component with an existing name will raise a ValueError
3. The name in Config can be used to override the default class name
4. All components must have a valid implementation of any abstract methods they inherit

* * * * *

Practical Example
-----------------

Below is an example showcasing the DI system in action:

```py
from py_spring_core import Component, Properties
class ExampleProperties(Properties):
    key: str
    value: str

class ExampleService(Component):
    example_properties: ExampleProperties

    def post_construct(self):
        print(f"Initialized with property: {self.example_properties.value}")

    def pre_destroy(self):
        print("ExampleService is being cleaned up")

class AnotherExampleService(Component):
    example_service: ExampleService

    def post_construct(self):
        print("AnotherExampleService initialized")
```

#### Steps in Dependency Injection:

1.  **Load Properties:** `ExampleProperties` values are loaded from the application's properties file.

2.  **Component Instantiation:** An instance of `ExampleService` is created.

3.  **Dependency Injection:** The `example_properties` attribute of `ExampleService` is injected with the `ExampleProperties` instance.

4.  **Another Component Instantiation:** An instance of `AnotherExampleService` is created.

5.  **Cross-Component Injection:** The `example_service` attribute of `AnotherExampleService` is injected with the `ExampleService` instance.

6.  **Lifecycle Methods:** `post_construct` methods are called for both components.

7.  **Cleanup:** On application shutdown, the `pre_destroy` method of `ExampleService` is called.

* * * * *

Summary
-------

PySpring's DI system provides:

-   **Seamless Dependency Management**: Automatically resolves and injects dependencies.

-   **Error Handling**: Ensures misconfigurations are detected early.

-   **Lifecycle Support**: Enables initialization and cleanup logic.

-   **Flexibility**: Supports a wide range of dependency types, including components, properties, and beans.

By simply declaring dependencies using type hints, PySpring handles the resolution and injection, letting you focus on building modular, testable, and maintainable applications.