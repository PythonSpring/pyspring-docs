# Component Lifecycle

After this page, you'll have a deeper understanding of how PySpring manages component initialization and destruction.

## Lifecycle overview

Every component goes through these stages:

1. **Discovery** — PySpring scans your project and finds all Component subclasses
2. **Instantiation** — Components are created (but dependencies aren't injected yet)
3. **Dependency injection** — Type hints are resolved and dependencies are set
4. **Post-construction** — `post_construct()` is called on each component
5. **Running** — the application is live
6. **Pre-destruction** — `pre_destroy()` is called during shutdown
7. **Destruction** — Components are garbage collected

## Initialization order

PySpring resolves the dependency graph and initializes components in dependency order. If `ServiceA` depends on `ServiceB`, then `ServiceB` is fully initialized (including its `post_construct()`) before `ServiceA`'s dependencies are injected.

```python
from py_spring_core import Component


class DatabaseService(Component):
    def post_construct(self):
        print("1. Database ready")


class UserRepository(Component):
    database_service: DatabaseService

    def post_construct(self):
        print("2. UserRepository ready (database is already initialized)")


class UserService(Component):
    user_repository: UserRepository

    def post_construct(self):
        print("3. UserService ready (repository is already initialized)")
```

!!! warning
    Circular dependencies will cause an error at startup. If `A` depends on `B` and `B` depends on `A`, PySpring can't resolve the graph. Restructure your code to break the cycle.

## `post_construct` vs `__init__`

Use `post_construct()`, not `__init__()`, for initialization that depends on injected values:

```python
from py_spring_core import Component


class CacheService(Component):
    database_service: DatabaseService

    # Don't do this — database_service isn't injected yet!
    # def __init__(self):
    #     self.cache = self.database_service.load_cache()

    def post_construct(self):
        # Do this — all dependencies are available
        self.cache = self.database_service.load_cache()
```

## `pre_destroy` for cleanup

`pre_destroy()` is called during application shutdown, in reverse initialization order:

```python
from py_spring_core import Component


class ConnectionPool(Component):
    def post_construct(self):
        self.pool = create_pool()

    def pre_destroy(self):
        self.pool.close()
        print("Connection pool closed")
```

## Singleton vs Prototype scope

**Singleton** (default): One instance for the entire application. `post_construct()` and `pre_destroy()` are called exactly once.

**Prototype**: A new instance per injection point. `post_construct()` is called for each instance.

## Recap

Understanding the component lifecycle helps you manage resources correctly.

* Components are initialized in dependency order
* `post_construct()` runs after all dependencies are injected
* `pre_destroy()` runs during shutdown in reverse order
* Use `post_construct()` instead of `__init__()` for DI-dependent setup
* Circular dependencies are detected and rejected at startup
