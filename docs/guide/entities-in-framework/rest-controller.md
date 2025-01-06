# RestController

The `RestController` class in PySpring serves as a base class for building RESTful API controllers. It provides a structured way to define API endpoints, handle HTTP requests, and manage middleware.

---

## Key Features

1. **Base Class**: Acts as a base class that should be extended for creating specific controllers.
2. **Routing**: Provides a mechanism to register routes using FastAPI's routing capabilities.
3. **Middleware Support**: Allows registration of middleware for pre- and post-request processing.
4. **FastAPI Integration**: Leverages FastAPI's features for building high-performance APIs.
5. **Configuration**: Supports configuration of the base URL prefix for controller routes.
6. **Automatic Registration**: Automatically integrates controllers with the application context and FastAPI during initialization.

---

## How to Define a RestController

### Create a Controller Class

Extend the `RestController` class and override the `register_routes()` method to define routes. Optionally, override `register_middlewares()` to add middleware.

### Example:

```py
from py_spring_core import RestController
from fastapi import HTTPException
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

class MyController(RestController):
    class Config:
        prefix = "/api/items"  # Base URL prefix for this controller

    def register_routes(self):
        @self.router.get("/")
        def read_items():
            return {"message": "List of items"}

        @self.router.get("/{item_id}")
        def read_item(item_id: int):
            return {"message": f"Details for item {item_id}"}

        @self.router.post("/", status_code=201)
        def create_item(item: Item):
            return item
```

### Key Points:

-   Use the `Config` inner class to set the route prefix.
-   Define routes using the `@self.router` decorator.

* * * * *

### Automatic Registration Process
1.  During application startup, PySpring scans for classes inheriting from `RestController`.
2.  The `_handle_register_rest_controller()` method:
    -   Registers the controller with the application context.
    -   Initializes an `APIRouter` instance.
    -   Calls `register_routes()` to add routes.
    -   Includes the controller's router in the main FastAPI application.
    -   Calls `register_middlewares()` for middleware registration.

* * * * *

### Defining Routes
Use `@self.router` decorators to define API endpoints inside `register_routes()`:

```py
@self.router.get("/")
def read_items():
    return {"message": "List of items"}

@self.router.post("/", status_code=201)
def create_item(item: Item):
    return item
```
### Example:

-   `GET /` maps to the `read_items` method.
-   `POST /` maps to the `create_item` method.

### FastAPI Features:

-   Dependency Injection
-   Request body handling
-   Path parameter parsing

* * * * *

### Defining Middlewares
Override `register_middlewares()` to add middleware. Use `app.middleware()` to define custom middleware functions.

### Example:

```py
def register_middlewares(self):
    @self.app.middleware("http")
    async def add_process_time_header(request, call_next):
        start_time = time.time()
        response = await call_next(request)
        process_time = time.time() - start_time
        response.headers["X-Process-Time"] = str(process_time)
        return response
```
### Common Middleware Use Cases:
-   Authentication
-   Authorization
-   Logging
-   Performance metrics

### Accessing FastAPI Components
-   **`self.app`**: The main FastAPI application instance.
-   **`self.router`**: The APIRouter instance for the controller.


### Important Considerations
1.  **Class Configuration**: Use the `Config` class to define the URL prefix.
2.  **Route Registration**: Ensure all routes are defined in `register_routes()` using `@self.router`.
3.  **Middleware Registration**: Add middleware in `register_middlewares()` using `app.middleware()`.
4.  **Dependency Injection**: In this class, you can leverage both PySpring's and FastAPI's dependency injection systems to manage dependencies efficiently.
5.  **Error Handling**: Use `HTTPException` for proper error responses and integrate with PySpring's error handling mechanisms.
6.  **Code Organization**: Keep controllers focused on request handling and delegate business logic to services or components.

* * * * *

### Summary
The `RestController` class helps build well-structured, maintainable, and scalable RESTful APIs in PySpring. By utilizing its features like routing, middleware support, and integration with FastAPI, developers can focus on creating efficient APIs while maintaining code clarity and modularity.