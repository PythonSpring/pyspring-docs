# Features

PySpring gives you a Spring Boot-inspired architecture for Python, with all the developer experience you'd expect from a modern framework.

## Type-safe dependency injection

You declare dependencies as type hints. PySpring resolves and injects them automatically — no decorators, no service locators, no magic strings.

```python
class UserService(Component):
    user_repository: UserRepository  # Injected automatically

    def post_construct(self):
        print("Ready to serve users!")
```

Your editor sees the types, gives you autocomplete, and catches mistakes before you run anything.

## Spring Boot patterns in Python

If you've used Spring Boot, you'll feel at home. PySpring provides:

* **Components** — the building blocks of your application
* **Properties** — type-safe configuration loaded from JSON/YAML
* **REST Controllers** — class-based controllers with route decorators
* **Bean Collections** — integration point for third-party code
* **Event System** — publish-subscribe for decoupled communication
* **Middleware** — request/response processing pipeline

## Built on FastAPI

PySpring uses FastAPI as its web layer. That means you get:

* **Automatic OpenAPI docs** — Swagger UI and ReDoc, for free
* **High performance** — one of the fastest Python frameworks available
* **Async support** — use `async def` wherever you need it
* **Pydantic validation** — request and response validation built in

## Auto configuration

Define your configuration as Pydantic models. PySpring loads and validates them from JSON or YAML at startup.

```python
class DatabaseProperties(Properties):
    __key__ = "database"
    host: str
    port: int
    name: str
```

If the config file is missing a field or has the wrong type, you get a clear error at startup — not a mystery crash in production.

## Component lifecycle

Every component has `post_construct` and `pre_destroy` hooks for clean resource management.

```python
class CacheService(Component):
    def post_construct(self):
        self.cache = {}
        print("Cache initialized")

    def pre_destroy(self):
        self.cache.clear()
        print("Cache cleaned up")
```

## Event-driven architecture

A built-in, thread-safe event system lets components communicate without direct coupling.

```python
class UserCreatedEvent(ApplicationEvent):
    user_id: str
    username: str
```

Publish from one component, handle in another. No imports between them.

## Starters

Package reusable functionality into self-contained modules. A starter bundles components, properties, controllers, and beans into a single unit that plugs into any PySpring application.

```python
class CacheStarter(PySpringStarter):
    def on_configure(self):
        self.component_classes.append(CacheService)
        self.properties_classes.append(CacheProperties)
```

Publish starters as pip packages with entry point auto-discovery — users just `pip install` and it works.

## Graceful shutdown

Handle SIGINT and SIGTERM cleanly. Close database connections, flush queues, finish in-flight requests — all with clear lifecycle hooks.

## Scheduling

Built-in support for scheduled tasks with cron expressions, intervals, and complex trigger combinations — powered by APScheduler and fully integrated with dependency injection.

See the [Scheduler module docs](modules/scheduler/) for the full reference.

## ORM and data access

[PySpring Model](modules/model/) brings Spring Data JPA-style repository patterns to Python:

```python
class UserRepository(CrudRepository[int, User]):
    def find_by_name(self, name: str) -> Optional[User]: ...
    def find_all_by_status_in(self, status: List[str]) -> List[User]: ...
    def find_by_age_gt_and_status(self, age: int, status: str) -> Optional[User]: ...
```

Declare method signatures — PySpring generates the SQL automatically. Built on SQLModel and SQLAlchemy.

## Transaction management

Declarative transactions with 7 propagation types, modeled after Spring's `@Transactional`:

```python
from py_spring_model import Transactional, Propagation

@Transactional(propagation=Propagation.REQUIRES_NEW)
def write_audit_log(self, message: str) -> None:
    # Always runs in a new, independent transaction
    ...
```

Supports REQUIRED, REQUIRES_NEW, SUPPORTS, MANDATORY, NOT_SUPPORTED, NEVER, and NESTED propagation.

See the [Model module docs](modules/model/) for the full reference.
