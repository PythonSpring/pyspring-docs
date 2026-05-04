# Starters

After this page, you'll know how to create reusable, self-contained modules that plug into any PySpring application — and how to publish them as installable packages.

A **Starter** is a bundle of components, properties, bean collections, and controllers that you can drop into any PySpring app. Think of it like a Spring Boot starter: a single dependency that brings in everything needed for a specific capability (database support, authentication, metrics, etc.).

## Creating a starter

Extend `PySpringStarter` and register entities in `on_configure`:

```python
from py_spring_core import PySpringStarter, Component, Properties


class CacheProperties(Properties):
    __key__ = "cache"
    ttl_seconds: int = 300
    max_size: int = 1000


class CacheService(Component):
    cache_properties: CacheProperties

    def post_construct(self):
        self.store = {}

    def get(self, key: str):
        return self.store.get(key)

    def set(self, key: str, value: object):
        self.store[key] = value


class CacheStarter(PySpringStarter):
    def on_configure(self):
        self.component_classes.append(CacheService)
        self.properties_classes.append(CacheProperties)
```

When `on_configure` is called, the starter registers its entities with the framework. The host application doesn't need to know about `CacheService` or `CacheProperties` — it just uses the starter.

## Using a starter

Pass starter instances to `PySpringApplication`:

```python
from py_spring_core import PySpringApplication

app = PySpringApplication(
    "./app-config.json",
    starters=[CacheStarter()],
)
app.run()
```

The starter's entities are merged into the application alongside your own components and properties.

## Lifecycle hooks

Starters have two lifecycle hooks:

### `on_configure`

Called **before** the IoC container is built. Use this to register entities:

```python
class MyStarter(PySpringStarter):
    def on_configure(self):
        self.component_classes.append(MyComponent)
        self.bean_collection_classes.append(MyBeans)
        self.properties_classes.append(MyProperties)
        self.rest_controller_classes.append(MyController)
```

You can register any combination of:

| Field                      | Type                     |
|----------------------------|--------------------------|
| `component_classes`        | `list[Type[Component]]`  |
| `bean_collection_classes`  | `list[Type[BeanCollection]]` |
| `properties_classes`       | `list[Type[Properties]]` |
| `rest_controller_classes`  | `list[Type[RestController]]` |

### `on_initialized`

Called **after** the IoC container is built, dependencies are injected, and `app_context` is set. Use this for post-initialization logic:

```python
class MyStarter(PySpringStarter):
    def on_initialized(self):
        # app_context is available here
        service = self.app_context.must_get_component(MyComponent)
        service.do_something()
```

## Lifecycle order

Here's the full sequence during application startup:

1. **Discovery** — auto-discover starters from entry points (see below)
2. **Merge** — combine auto-discovered starters with manually provided ones
3. **`on_configure()`** — called on each starter to register entities
4. **IoC container build** — components, beans, and properties are instantiated
5. **Dependency injection** — all dependencies are resolved
6. **`set_context()`** — each starter receives the `ApplicationContext`
7. **Starter validation** — `depends_on` dependencies are verified
8. **`on_initialized()`** — called on each starter after everything is ready
9. **Component `post_construct()`** — standard component lifecycle continues

## Dependency validation

Starters can declare dependencies on specific entity classes. PySpring validates that these exist in the application context at startup:

```python
from py_spring_core import PySpringStarter, Component


class RequiredDatabase(Component):
    ...


class MyStarter(PySpringStarter):
    def on_configure(self):
        self.depends_on = [RequiredDatabase]
        self.component_classes.append(SomeService)
```

If `RequiredDatabase` is not registered by the host application or another starter, PySpring raises an `InvalidDependencyError` at startup — not a mystery failure at runtime.

## Dataclass-style registration

You can also register entities directly via the constructor, without overriding `on_configure`:

```python
starter = PySpringStarter(
    component_classes=[CacheService],
    properties_classes=[CacheProperties],
)
```

This is useful for simple cases or testing. For reusable starters, prefer the subclass approach with `on_configure`.

## Auto-discovery with entry points

Starters can be automatically discovered from installed packages using Python's `entry_points` mechanism. This means users of your starter just `pip install` it — no manual wiring required.

### Publishing a starter

In your starter package's `pyproject.toml`, register the starter class under the `pyspring.starters` group:

```toml
[project.entry-points."pyspring.starters"]
my-cache-starter = "my_cache_package:CacheStarter"
```

When a PySpring application starts, it scans this entry point group and instantiates any discovered starters automatically.

### How discovery works

`StarterDiscovery.from_entry_points()` scans the `pyspring.starters` group and:

- Loads each entry point
- Validates it's a concrete `PySpringStarter` subclass (not the base class)
- Deduplicates by class identity
- Logs warnings for invalid entries and load failures

Manual starters take priority — if you pass a starter class explicitly and the same class is also auto-discovered, the auto-discovered duplicate is skipped.

### Package scanning

For development or non-entry-point scenarios, you can discover starters by scanning packages directly:

```python
from py_spring_core.core.starter import StarterDiscovery

starters = StarterDiscovery.from_packages(["my_starter_package"])
```

This recursively walks the package's sub-modules and collects all `PySpringStarter` subclasses.

## Recap

Starters let you package reusable functionality into self-contained modules.

* Extend `PySpringStarter` and override `on_configure` to register entities
* Use `on_initialized` for post-IoC-container setup
* Declare `depends_on` to validate required dependencies at startup
* Publish as a pip package with `pyspring.starters` entry points for zero-config auto-discovery
* Manual starters and auto-discovered starters merge seamlessly — manual takes priority on duplicates
