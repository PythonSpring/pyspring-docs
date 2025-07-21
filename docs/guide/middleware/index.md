# Middleware System

PySpring provides a robust middleware system that allows you to intercept and process HTTP requests and responses. This guide covers how to create, register, and manage middlewares in your PySpring application.

## Overview

The middleware system in PySpring is built on top of FastAPI's middleware system, providing additional features like:

- Centralized middleware management through `MiddlewareRegistry`
- Automatic dependency injection for middlewares
- Simplified middleware creation with the `Middleware` base class
- Flexible execution order control

## Creating a Middleware

To create a middleware, extend the `Middleware` base class:

```python
from py_spring_core import Middleware
from fastapi import Request, Response

class LoggingMiddleware(Middleware):
    async def process_request(self, request: Request) -> Response | None:
        # Pre-processing logic
        print(f"Incoming request to: {request.url.path}")
        
        # Return None to continue to the next middleware
        return None
```

The `process_request` method is where you implement your middleware logic. You can:
- Return `None` to continue to the next middleware
- Return a `Response` to short-circuit the request and return immediately
- Access and modify the request object
- Perform any necessary pre-processing

### Optional Features

Middlewares can implement additional features:

```python
class CustomMiddleware(Middleware):
    def should_skip(self, request: Request) -> bool:
        """Optional method to determine if middleware should be skipped"""
        return request.url.path.startswith("/health")

    async def process_request(self, request: Request) -> Response | None:
        # Your middleware logic here
        return None
```

## Registering Middlewares

To register middlewares globally, create a `MiddlewareRegistry` implementation:

```python
from py_spring_core import MiddlewareRegistry, Middleware
from typing import Type

class AppMiddlewareRegistry(MiddlewareRegistry):
    def get_middleware_classes(self) -> list[Type[Middleware]]:
        return [
            AuthMiddleware,
            LoggingMiddleware,
            CORSMiddleware
        ]
```

The framework will:

1. Automatically discover your `MiddlewareRegistry` implementation

2. Register all middlewares in the order specified

3. Inject any required dependencies into your middlewares

## Execution Order

Middlewares follow a Last-In-First-Out (LIFO) execution order:

```python
# If get_middleware_classes() returns: [MiddlewareA, MiddlewareB]

# Request flow:
# MiddlewareB → MiddlewareA → Route Handler

# Response flow:
# Route Handler → MiddlewareA → MiddlewareB
```

Consider this when ordering your middlewares. For example:
- Authentication middleware should run early
- Logging middleware might want to wrap the entire request
- CORS middleware typically needs to run first

## Best Practices

1. **Keep Middlewares Focused**: Each middleware should have a single responsibility

2. **Use Dependency Injection**: Leverage PySpring's dependency injection:
   ```python
   class AuthMiddleware(Middleware):
       auth_service: AuthService
       async def process_request(self, request: Request) -> Response | None:
           token = request.headers.get("Authorization")
           if not self.auth_service.validate_token(token):
               return Response(status_code=401)
           return None
   ```

3. **Error Handling**: Always handle exceptions in your middleware:
   ```python
   async def process_request(self, request: Request) -> Response | None:
       try:
           # Your middleware logic
           return None
       except Exception as e:
           logger.error(f"Middleware error: {e}")
           return Response(
               content={"error": "Internal Server Error"},
               status_code=500
           )
   ```

4. **Performance**: Keep middleware processing light and efficient:
   - Avoid heavy computations
   - Don't block the event loop
   - Cache results when possible

5. **Logging**: Add appropriate logging for debugging:
   ```python
   async def process_request(self, request: Request) -> Response | None:
       logger.debug(f"Processing request in {self.__class__.__name__}")
       # ... middleware logic
       logger.debug(f"Completed processing in {self.__class__.__name__}")
       return None
   ```

## Common Use Cases

Here are some common middleware use cases:

### Authentication
```python
class AuthMiddleware(Middleware):
    async def process_request(self, request: Request) -> Response | None:
        if not self._is_authenticated(request):
            return Response(status_code=401)
        return None
```

### Request Logging
```python
class LoggingMiddleware(Middleware):
    async def process_request(self, request: Request) -> Response | None:
        logger.info(f"Request: {request.method} {request.url}")
        return None
```

### CORS
```python
class CORSMiddleware(Middleware):
    async def process_request(self, request: Request) -> Response | None:
        if request.method == "OPTIONS":
            return Response(
                headers={
                    "Access-Control-Allow-Origin": "*",
                    "Access-Control-Allow-Methods": "GET, POST, PUT, DELETE, OPTIONS",
                    "Access-Control-Allow-Headers": "Content-Type, Authorization",
                }
            )
        return None
```

### Performance Monitoring
```python
class PerformanceMiddleware(Middleware):
    async def process_request(self, request: Request) -> Response | None:
        start_time = time.time()
        response = await self.call_next(request)
        duration = time.time() - start_time
        logger.info(f"Request took {duration:.2f} seconds")
        return response
```

## Project Structure

Recommended project structure for middleware organization:

```
src/
  middlewares/
    __init__.py
    auth_middleware.py
    logging_middleware.py
    cors_middleware.py
    middleware_registry.py
```

This structure keeps your middleware-related code organized and maintainable. 