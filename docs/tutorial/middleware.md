# Middleware

After this page, you'll know how to use PySpring's middleware system to process requests before they reach your controllers.

Middleware lets you run logic on every incoming request — authentication checks, logging, rate limiting, and more — without touching your controllers.

## Create a middleware

Extend `Middleware` and implement `process_request`:

```python
from fastapi import Request, Response

from py_spring_core.core.entities.middlewares.middleware import Middleware


class LoggingMiddleware(Middleware):
    async def process_request(self, request: Request) -> Response | None:
        print(f"Incoming request: {request.method} {request.url}")
        return None  # Continue to next middleware or route handler
```

Return `None` to let the request continue. Return a `Response` to short-circuit and respond immediately:

```python
from fastapi import Request, Response

from py_spring_core.core.entities.middlewares.middleware import Middleware


class ApiKeyMiddleware(Middleware):
    async def process_request(self, request: Request) -> Response | None:
        api_key = request.headers.get("X-API-Key")
        if not api_key:
            return Response(content="Missing API key", status_code=401)
        return None  # Key present, continue
```

## Skip requests conditionally

Override `should_skip` to bypass middleware for certain requests:

```python
from fastapi import Request, Response

from py_spring_core.core.entities.middlewares.middleware import Middleware


class AuthMiddleware(Middleware):
    def should_skip(self, request: Request) -> bool:
        # Don't require auth for health checks
        return str(request.url).endswith("/health")

    async def process_request(self, request: Request) -> Response | None:
        token = request.headers.get("Authorization")
        if not token:
            return Response(content="Unauthorized", status_code=401)
        return None
```

By default, `should_skip` returns `False`, so every request is processed.

## Register middleware

Use `MiddlewareConfiguration` to register your middleware classes:

```python
from py_spring_core.core.entities.middlewares.middleware import Middleware
from py_spring_core.core.entities.middlewares.middleware_registry import (
    MiddlewareConfiguration,
    MiddlewareRegistry,
)
from fastapi import Request, Response


class LoggingMiddleware(Middleware):
    async def process_request(self, request: Request) -> Response | None:
        print(f"{request.method} {request.url}")
        return None


class AuthMiddleware(Middleware):
    async def process_request(self, request: Request) -> Response | None:
        if not request.headers.get("Authorization"):
            return Response(content="Unauthorized", status_code=401)
        return None


class AppMiddlewareConfiguration(MiddlewareConfiguration):
    def configure_middlewares(self, registry: MiddlewareRegistry) -> None:
        registry.add_middleware(LoggingMiddleware)
        registry.add_middleware(AuthMiddleware)
```

!!! tip
    PySpring automatically discovers your `MiddlewareConfiguration` subclass. No manual wiring required.

## Control execution order

The registry gives you fine-grained control over middleware ordering:

```python
class AppMiddlewareConfiguration(MiddlewareConfiguration):
    def configure_middlewares(self, registry: MiddlewareRegistry) -> None:
        registry.add_middleware(LoggingMiddleware)
        registry.add_middleware(AuthMiddleware)

        # Insert rate limiting before auth
        registry.add_before(AuthMiddleware, RateLimitMiddleware)

        # Insert CORS handling after logging
        registry.add_after(LoggingMiddleware, CorsMiddleware)
```

Available ordering methods:

| Method | Description |
|---|---|
| `add_middleware(cls)` | Append to the end |
| `add_at_index(i, cls)` | Insert at position `i` |
| `add_before(target, cls)` | Insert before `target` |
| `add_after(target, cls)` | Insert after `target` |

!!! note
    Middleware forms a stack. The **last** middleware added is the outermost and runs first on the request path. On the response path, the order is reversed. For example, if you register `[A, B]`, the request flows `B -> A -> route` and the response flows `route -> A -> B`.

## Recap

Middleware lets you intercept and process requests before they hit your controllers.

* Extend `Middleware` and implement `process_request`
* Return `None` to continue, or a `Response` to short-circuit
* Override `should_skip` to bypass middleware for certain requests
* Use `MiddlewareConfiguration` to register and order middleware
* Control ordering with `add_before`, `add_after`, and `add_at_index`

Next, let's learn about **Scheduling** — how to run tasks on a schedule.
