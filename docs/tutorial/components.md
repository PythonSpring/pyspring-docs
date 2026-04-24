# Components

After this page, you'll know how to create components — the fundamental building blocks of a PySpring application.

A **Component** is a class that PySpring manages for you. It gets instantiated, initialized, and destroyed by the framework — and it can have dependencies injected automatically.

## Create a component

Let's create a simple service:

```python
from py_spring_core import Component


class GreetingService(Component):
    def greet(self, name: str) -> str:
        return f"Hello, {name}!"
```

That's it. By extending `Component`, this class is now managed by PySpring. The framework will:

* Discover it automatically
* Create an instance
* Make it available for injection into other components

## Lifecycle hooks

Components have two lifecycle hooks you can override:

```python
from py_spring_core import Component


class GreetingService(Component):
    def post_construct(self):
        print("GreetingService is ready!")

    def greet(self, name: str) -> str:
        return f"Hello, {name}!"

    def pre_destroy(self):
        print("GreetingService shutting down...")
```

* **`post_construct()`** — called after the component is fully initialized and all dependencies are injected. Use this for setup logic.
* **`pre_destroy()`** — called before the component is destroyed. Use this for cleanup.

!!! tip
    Use `post_construct()` instead of `__init__()` for initialization that depends on injected values. At `__init__` time, dependencies haven't been injected yet.

## Component scope

By default, components are **singletons** — one instance shared across the entire application. You can change this:

```python
from py_spring_core import Component
from py_spring_core.core.entities.component import ComponentScope


class GreetingService(Component):
    class Config:
        scope = ComponentScope.Singleton  # default

    def greet(self, name: str) -> str:
        return f"Hello, {name}!"
```

Available scopes:

* `ComponentScope.Singleton` — one shared instance (default)
* `ComponentScope.Prototype` — a new instance created each time

!!! note
    Most components should be singletons. Use `Prototype` only when you genuinely need a fresh instance per usage.

## Recap

Components are the core building blocks in PySpring.

* Extend `Component` to create a managed class
* Use `post_construct()` for initialization after DI
* Use `pre_destroy()` for cleanup
* Default scope is `Singleton`
* Components are auto-discovered — no manual registration needed

Next, let's learn about **Properties** — how to manage configuration in a type-safe way.
