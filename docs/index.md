# **PySpring** Framework


### Overview

**PySpring** is a Python web framework inspired by **Spring Boot**. It provides a structured approach to building scalable web applications with key features like:

- **Auto Dependency Injection**
- **Auto Configuration Management**
- **ASGI Web Server** for hosting your application

### Technologies Used in PySpring

- **FastAPI**: For the web server layer.
- **Pydantic**: For data validation.

**PySpring** combines these technologies to deliver a seamless development experience for building modern, scalable applications.
### Key Features

- **Application Initialization**: **PySpringApplication** class serves as the main entry point for the **PySpring** application. It initializes the application from a configuration file, scans the application source directory for Python files, and groups them into class files and model files

- **Application Context Management**: **PySpring** manages the application context and dependency injection. It registers application entities such as components, controllers, bean collections, and properties. It also initializes the application context and injects dependencies.

- **REST Controllers**: **PySpring** supports RESTful API development using the RestController class. It allows you to define routes, handle HTTP requests, and register middlewares easily.

- **Component-based Architecture**: **PySpring** encourages a component-based architecture, where components are reusable and modular building blocks of the application. Components can have their own lifecycle and can be registered and managed by the application context.

- **Properties Management**: Properties classes provide a convenient way to manage application-specific configurations. **PySpring** supports loading properties from a properties file and injecting them into components.

- **Framework Modules**: **PySpring** allows the integration of additional framework modules to extend the functionality of the application. Modules can provide additional routes, middlewares, or any other custom functionality required by the application.

- **Builtin FastAPI Integration**: **PySpring** integrates with `FastAPI`, a modern, fast (high-performance), web framework for building APIs with Python. It leverages FastAPI's features for routing, request handling, and server configuration.

- **OpenAPI Generation**: Since **PySpring** leverages `FastAPI`, it automatically generates [OpenAPI](https://fastapi.tiangolo.com/features/#based-on-open-standards) documentation for the application. The API routes, endpoints, and data models are used to create interactive, self-updating OpenAPI documentation, which can be easily accessed via FastAPI's built-in web interface.

- **Type-Safety**: The framework is type-safe when used properly. All dependency injection (DI) is determined based on Python type hints, ensuring that dependencies are injected in a consistent and reliable manner. This feature enables better development practices by reducing runtime errors and improving code clarity.

- **Qualifier Support**: PySpring supports qualifiers for dependency injection, allowing you to specify which implementation to inject when multiple implementations of the same interface exist. This is achieved using Python's `Annotated` type hints, making it easy to manage complex dependency scenarios.

- **Component Registration Validation**: The framework includes robust validation to prevent duplicate component registration and provides clear error messages for developers. This ensures that your application maintains a clean and consistent component structure.

- **Decorator-Based Route Mapping**: PySpring now supports a more declarative approach to route definition using decorators, similar to Spring's annotation-based routing. This provides a cleaner and more type-safe way to define API endpoints with better IDE support and code organization.

- **Event System**: PySpring provides a powerful event system that enables event-driven architecture in your applications. It features thread-safe event publishing, decorator-based event listeners, and asynchronous event processing. The system is fully integrated with the component system and provides type-safe events using Pydantic models.

### Getting Started
To get started with **PySpring**, follow these steps:

1. Install the **PySpring** framework by running:

```bash
pip3 install py-spring-core
```

2. Create a new Python project and navigate to its directory

-  Implement your application properties, components, controllers, using **PySpring** conventions inside declared source code folder (whcih can be modified the key `app_src_target_dir` inside app-config.json), this controls what folder will be scanned by the framework.

- Instantiate a `PySpringApplication` object in your main script, passing the path to your application configuration file.

- Optionally, define and enable any framework modules you want to use.

- Run the application by calling the `run()` method on the `PySpringApplication` object, as shown in the example code below:

```py
from py_spring_core import PySpringApplication

def main():
    app = PySpringApplication("./app-config.json")
    app.run()

if __name__ == "__main__":
    main()
```

- For example project, please refer to this [github repo](https://github.com/NFUChen/PySpring-Example-Project).

# Contributing

Contributions to **PySpring** are welcome! If you find any issues or have suggestions for improvements, please submit a pull request or open an issue on GitHub.