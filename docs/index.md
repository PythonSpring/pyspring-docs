<style>
.md-content .md-typeset h1 { display: none; }
</style>

<p align="center">
    <em>PySpring — Spring Boot-inspired Python framework with type-safe dependency injection, ready for production.</em>
</p>
<p align="center">
<a href="https://pypi.org/project/py-spring-core" target="_blank">
    <img src="https://img.shields.io/pypi/v/py-spring-core?color=%2334D058&label=pypi%20package" alt="Package version">
</a>
<a href="https://pypi.org/project/py-spring-core" target="_blank">
    <img src="https://img.shields.io/pypi/pyversions/py-spring-core.svg?color=%2334D058" alt="Supported Python versions">
</a>
</p>

---

**Source Code**: <a href="https://github.com/PythonSpring/pyspring-core" target="_blank">https://github.com/PythonSpring/pyspring-core</a>

---

PySpring is a Python web framework inspired by **Spring Boot**. It gives you a structured, type-safe approach to building scalable web applications — with automatic dependency injection, configuration management, and an ASGI web server built on FastAPI.

The key features are:

* **Type-safe**: Dependency injection resolved entirely from Python type hints. No decorators, no magic strings — your editor knows everything. 🚀
* **Spring Boot-inspired**: Familiar patterns — Components, Properties, Controllers, Bean Collections — for developers who appreciate structured architecture.
* **Built on FastAPI**: Automatic OpenAPI docs, high performance, and async support out of the box.
* **Auto configuration**: Load and validate configuration from JSON/YAML using Pydantic models.
* **Lifecycle management**: `post_construct` and `pre_destroy` hooks for clean resource management.
* **Event-driven**: Built-in thread-safe event system with typed Pydantic events.

## Requirements

Python 3.10+

PySpring builds on top of:

* <a href="https://fastapi.tiangolo.com/" target="_blank">FastAPI</a> — for the web server layer.
* <a href="https://docs.pydantic.dev/" target="_blank">Pydantic</a> — for data validation and configuration.

## Installation

<div class="termy">

```console
$ pip install py-spring-core

---> 100%
```

</div>

## Example

### Create it

Create a file `main.py` with:

```python
from py_spring_core import PySpringApplication


def main():
    app = PySpringApplication("./app-config.json")
    app.run()


if __name__ == "__main__":
    main()
```

Now create a component with dependency injection in `services.py`:

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
        print(f"Connected to {self.database_properties.host}")
```

And a REST controller in `controllers.py`:

```python
from py_spring_core import RestController
from py_spring_core.core.entities.controllers.rest_controller import GetMapping


class UserController(RestController):
    class Config:
        prefix = "/api/users"

    database_service: DatabaseService  # Auto-injected!

    @GetMapping("/")
    def get_users(self):
        return {"users": []}
```

That's the entire application. You declared types — PySpring resolved and injected everything.

### Run it

```console
$ python main.py
```

### Check it

Open your browser at <a href="http://127.0.0.1:8000/docs" target="_blank">http://127.0.0.1:8000/docs</a>.

You get **automatic interactive API documentation** (Swagger UI) with your routes already defined — because PySpring builds on FastAPI. ✨

You also get **ReDoc** at `/redoc`.

## Recap

In that small example, you:

* **Declared dependencies** using type hints — PySpring resolved and injected them.
* **Defined configuration** as a Pydantic model — loaded and validated automatically.
* **Built a REST API** using decorators — with automatic OpenAPI docs.

All with **type safety**, **editor support**, and **minimal boilerplate**.

To learn everything step by step, head to the <a href="tutorial/" target="_blank">Tutorial - User Guide</a>.

## License

This project is licensed under the terms of the MIT license.
