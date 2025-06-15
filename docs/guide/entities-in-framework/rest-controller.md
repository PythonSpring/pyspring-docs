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

Extend the `RestController` class and use the route mapping decorators to define your API endpoints. Optionally, override `register_middlewares()` to add middleware.

### Example:

```py
from py_spring_core import RestController
from py_spring_core import GetMapping, PostMapping
from fastapi import HTTPException
from pydantic import BaseModel

class Item(BaseModel):
    name: str
    price: float

class MyController(RestController):
    class Config:
        prefix = "/api/items"  # Base URL prefix for this controller

    @GetMapping("/")
    def read_items(self):
        return {"message": "List of items"}

    @GetMapping("/{item_id}")
    def read_item(self, item_id: int):
        return {"message": f"Details for item {item_id}"}

    @PostMapping("/", status_code=201)
    def create_item(self, item: Item):
        return item
```

### Key Points:

-   Use the `Config` inner class to set the route prefix.
-   Define routes using the route mapping decorators (`@GetMapping`, `@PostMapping`, etc.).

* * * * *

### Automatic Registration Process
1.  During application startup, PySpring scans for classes inheriting from `RestController`.
2.  The `_handle_register_rest_controller()` method:
    -   Registers the controller with the application context.
    -   Initializes an `APIRouter` instance.
    -   Automatically registers routes defined with decorators.
    -   Includes the controller's router in the main FastAPI application.
    -   Calls `register_middlewares()` for middleware registration.

* * * * *

### Defining Routes
PySpring provides a declarative approach to route definition using decorators, similar to Spring's annotation-based routing. This provides a cleaner and more type-safe way to define API endpoints.

#### Available Decorators:
- `@GetMapping`: For HTTP GET requests
- `@PostMapping`: For HTTP POST requests
- `@PutMapping`: For HTTP PUT requests
- `@DeleteMapping`: For HTTP DELETE requests
- `@PatchMapping`: For HTTP PATCH requests

#### Example Usage:
```python
from py_spring_core import RestController
from py_spring_core import GetMapping, PostMapping, PutMapping, DeleteMapping
from pydantic import BaseModel

class User(BaseModel):
    name: str
    email: str

class UserController(RestController):
    class Config:
        prefix = "/api/users"

    @GetMapping("/")
    def get_users(self):
        return {"users": []}
        
    @GetMapping("/{user_id}")
    def get_user(self, user_id: int):
        return {"user_id": user_id}
        
    @PostMapping("/")
    def create_user(self, user: User):
        return {"message": "User created", "user": user}
        
    @PutMapping("/{user_id}")
    def update_user(self, user_id: int, user: User):
        return {"message": f"User {user_id} updated", "user": user}
        
    @DeleteMapping("/{user_id}")
    def delete_user(self, user_id: int):
        return {"message": f"User {user_id} deleted"}
```

#### Key Benefits:
1. **Type Safety**: Decorators provide better type checking and IDE support
2. **Cleaner Code**: Route definitions are more concise and readable
3. **Automatic Registration**: Routes are automatically registered during application initialization
4. **Class-Level Prefixing**: Still supports the `Config.prefix` for base URL prefixing
5. **Preserved Metadata**: Function metadata is preserved using `functools.wraps`

#### Technical Details:
- Routes are stored in a static `RouteMapping.routes` dictionary
- Each route is registered with its HTTP method, path, and handler function
- Route registration happens during application initialization
- Decorators preserve function metadata for better introspection

### Defining Middlewares
Override `register_middlewares()`