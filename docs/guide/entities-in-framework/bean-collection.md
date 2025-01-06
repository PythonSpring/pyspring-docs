The `BeanCollection` class in PySpring is designed to help integrate third-party code, especially when developers cannot modify the code directly. Below is a breakdown of its key features and functionality.

### **Purpose and Key Features**

-   **Bean Management**: The `BeanCollection` helps organize and manage a collection of beans (components) within a class. It provides a way to access and control these beans efficiently.

-   **Scanning for Beans**: The `scan_beans()` method scans the current class for methods that create beans. These methods are identified by their names, which by default start with the identifier `create` (e.g., `createMyBean`).

-   **Bean Creation**: Beans are created using methods whose return types are annotated. The return type of a bean creation method indicates the type of the bean being created.

-   **BeanView**: Each bean is represented by a `BeanView` object, which holds:

    -   The bean itself
    -   The bean's name
    -   The bean creation function
    -   A validation method (`is_valid_bean()`) to check if the bean's name matches the name of the class it returns.
-   **Error Handling**: Common errors are handled through exceptions, such as:
        -   `BeanConflictError`: Raised when a bean with the same name already exists.
        -   `InvalidBeanError`: Raised when the bean's name doesn't match the class name.

* * * * *

### **Properties Integration**

`BeanCollection` can also work with properties, which are loaded **before** the beans are created. This is a key aspect of how `BeanCollection` integrates with the broader PySpring framework.

#### **How Properties Interact with BeanCollection**:

-   **Properties Loading**:
    PySpring loads properties from a designated properties file using the `Properties` class and `_PropertiesLoader`. This happens before the initialization of the Inversion of Control (IoC) container.

-   **Accessing Properties**:
    Once loaded, properties are accessible via the `get_properties()` method in the `ApplicationContext`. The properties are stored in the `singleton_properties_instance_container`.

-   **Dependency Injection**:
    PySpring's dependency injection mechanism ensures that any required properties are injected into a bean before it is created. This is done by inspecting the type annotations of the bean creation function.

#### **How it Works**:

1.  **Configuration**:
    The application is configured to load properties from a specific file as defined in `app-config.json`.

2.  **Loading Properties**:
    During initialization, `PySpringApplication` uses the `_PropertiesLoader` to load properties into a dictionary. The keys in the dictionary match the keys defined in the `Properties` class. The `load_properties()` method of `ApplicationContext` then makes these properties available.

3.  **Bean Creation with Properties**:
    When a bean is created via a method in `BeanCollection`, PySpring checks for any dependencies, including properties. If a property is declared as a dependency, it will be injected into the bean before the bean is created.

#### **Example**:
Suppose you have a properties class and a bean collection:
```py
from py_spring_core import Properties, BeanCollection

class MyProperties(Properties):
    __key__ = "my_properties"
    my_config_value: str

class MyBeanCollection(BeanCollection):
    properties: MyProperties
    @classmethod
    def create_my_bean(cls) -> MyBean:
        return MyBean(cls.properties.my_config_value)

class MyBean:
    def __init__(self, config_value: str):
        self.config_value = config_value`
```
In this example:

-   `MyBeanCollection` creates a `MyBean` instance and depends on `MyProperties` to fetch a configuration value.
-   When the `ApplicationContext` initializes the bean, it first loads the properties from the properties file and injects the `MyProperties` instance into the `createMyBean` method.

* * * * *

### **How BeanCollection Works**

1.  **Identifying Bean Creation Functions**:
    The `scan_beans()` method identifies bean creation methods by checking if their names start with a specific identifier, which is `create` by default. For example, a method named `createMyBean` would be recognized as a bean creation method.

2.  **Creating `BeanView` Instances**:
    For each method found, `scan_beans()` creates a `BeanView` instance. This object stores information about the bean, such as the creation function and the bean's class type.

3.  **Bean Validation**:
    The `is_valid_bean()` method within `BeanView` checks whether the name of the bean matches the name of the class it is supposed to create. For instance, if a method is annotated to return an object of type `MyBean`, the bean's name must also be `MyBean`.

4.  **Dependency Injection**:
    While `BeanCollection` is responsible for creating bean instances, dependency injection ensures that any required dependencies (including properties) are automatically injected into the beans.

* * * * *

### **Integration with PySpring**

-   **Registration**:
    `BeanCollection` classes are registered within the `ApplicationContext`. This allows PySpring to manage and inject these beans into other components of the application.

-   **Singleton Management**:
    Beans created by `BeanCollection` are treated as singletons by the `ApplicationContext`, meaning that only one instance of each bean is created and reused.

-   **Usage**:
    Beans managed by `BeanCollection` can be accessed through the `ApplicationContext` using the `get_bean()` method.

* * * * *

### **Benefits of Using BeanCollection**

-   **Loose Coupling**:
    The application code does not need to know how third-party objects are created, promoting separation of concerns and flexibility.

-   **Centralized Management**:
    Beans are managed in one central location, making it easier to find and maintain them.

-   **Simplified Integration**:
    `BeanCollection` provides a standardized approach to integrating third-party or external code without needing to modify it.

-   **Dependency Injection**:
    Beans can take advantage of PySpring's dependency injection, allowing them to access other registered components, controllers, or beans.

-   **Properties Support**:
    `BeanCollection` can load properties before bean creation, making it possible to configure beans dynamically using values from external properties files.

* * * * *

### **Beans Interacting with Other Components**

Beans created within a `BeanCollection` can interact with other components and properties defined within the project, not just the properties used during their initial creation. PySpring's dependency injection mechanism allows these beans to access other registered entities (components, beans, and properties) after they are instantiated.

#### **How This Interaction Works**:

-   **Dependency Injection**:
    When a bean is created, PySpring examines the type hints of the bean creation method, as well as the class definitions of other components, for type annotations. It uses these annotations to inject the necessary dependencies into the bean, including other beans, properties, and components.

-   **Component Interaction**:
    In your example, `ExampleService` has type hints for `ExampleProperties` and `MyBean`. When `ExampleService` is instantiated, PySpring uses dependency injection to inject the correct instances of these dependencies. Additionally, `AnotherExampleService` depends on `ExampleService`, which allows it to access properties via `ExampleService`.

-   **Lifecycle Methods**:
    Components like `ExampleService` and `AnotherExampleService` have lifecycle methods (`post_construct` and `pre_destroy`) that PySpring automatically calls. The `post_construct` method is executed after a component is initialized, meaning all its dependencies have been injected. At this point, the component can begin using the injected objects and properties.

```py
class MyProperties(Properties):
    key = "my_properties"
    my_config_value: str

# this may be a third party class that you cannot modify
class MyBean: 
    def __init__(self, config_value: str):
        self.config_value = config_value


class MyBeanCollection(BeanCollection):
    properties: MyProperties

    @classmethod
    def createMyBean(cls) -> MyBean:
        return MyBean(cls.properties.my_config_value)




class ExampleProperties(Properties):
    key = "example"
    value: str


class ExampleService(Component):
    example_properties: ExampleProperties
    my_bean: MyBean

    def post_construct(self) -> None:
        logger.info(f"Example value: {self.example_properties.value}")

    def pre_destroy(self) -> None:
        logger.info("Pre destroy method called")


class AnotherExampleService(Component):
    example_service: ExampleService

    def post_construct(self) -> None:
        logger.info("AnotherExampleService post construct called")
        logger.info(f"Example value: {self.example_service.example_properties.value}")
```

#### **How It Works in This Example**:

1.  **Properties Loading**:
    The application loads properties for `MyProperties` and `ExampleProperties` from the properties file.

2.  **Bean Creation**:
    `MyBeanCollection` creates an instance of `MyBean`, injecting `MyProperties` during its creation.

3.  **Component Instantiation**:
    PySpring initializes `ExampleService` and `AnotherExampleService`, injecting dependencies via type annotations.
4.  **Dependency Injection**:
    -   `ExampleService` receives an instance of `ExampleProperties` and an instance of `MyBean`.
    -   `AnotherExampleService` receives an instance of `ExampleService`.
5.  **Lifecycle Hook**:
    -   The `post_construct` method in `ExampleService` logs the value of `example_properties.value`, demonstrating that injected properties are available. It also shows that the injected `MyBean` is accessible.
    -   The `post_construct` method in `AnotherExampleService` logs the value of the same `example_properties.value` via `example_service`, demonstrating access to an injected component's properties.


#### **Key Takeaways**:
-   **Beans Interact with Components**:
    Beans created by a `BeanCollection` can interact with any component registered within the application context. This is because all these classes (beans, components, and controllers) are managed by the application context.

-   **Properties Sharing**:
    Properties loaded from a properties file are not only used for bean creation but can also be injected into components. This enables dynamic configuration of the application.

-   **Dependency IoC Container**:
    PySpring builds a dependency IoC container for all registered entities. This allows the framework to inject objects into other entities (properties, beans, components) that depend on them.

-   **Flexibility**:
    This approach provides a flexible architecture where components and beans can depend on each other, allowing the creation of modular and interconnected applications.

### **Summary**

The `BeanCollection` class provides a structured method to integrate external code into a PySpring application by treating them as managed beans. This approach enhances modularity, maintainability, and reduces tight coupling with third-party code. Additionally, it supports properties loading and dependency injection, allowing for more flexible and configurable bean creation.