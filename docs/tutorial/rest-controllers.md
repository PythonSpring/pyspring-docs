# REST Controllers

After this page, you'll know how to build REST APIs using PySpring's class-based controllers.

PySpring controllers build on FastAPI, so you get automatic OpenAPI docs, request validation, and high performance — with a class-based structure on top.

## Create a controller

```python
from py_spring_core import RestController
from py_spring_core.core.entities.controllers.rest_controller import GetMapping


class ItemController(RestController):
    class Config:
        prefix = "/api/items"

    @GetMapping("/")
    def get_items(self):
        return {"items": []}
```

The `Config.prefix` sets the base URL for all routes in this controller. The `@GetMapping("/")` decorator registers a GET endpoint at `/api/items/`.

!!! tip
    PySpring automatically discovers and registers your controllers. No need to manually add them to a router.

## Route decorators

PySpring provides decorators for all common HTTP methods:

```python
from py_spring_core import RestController
from py_spring_core.core.entities.controllers.rest_controller import (
    GetMapping,
    PostMapping,
    PutMapping,
    DeleteMapping,
    PatchMapping,
)
from pydantic import BaseModel


class Item(BaseModel):
    name: str
    price: float


class ItemController(RestController):
    class Config:
        prefix = "/api/items"

    @GetMapping("/")
    def get_items(self):
        return {"items": []}

    @GetMapping("/{item_id}")
    def get_item(self, item_id: int):
        return {"item_id": item_id}

    @PostMapping("/", status_code=201)
    def create_item(self, item: Item):
        return item

    @PutMapping("/{item_id}")
    def update_item(self, item_id: int, item: Item):
        return {"item_id": item_id, **item.model_dump()}

    @DeleteMapping("/{item_id}")
    def delete_item(self, item_id: int):
        return {"message": f"Item {item_id} deleted"}
```

Because PySpring uses FastAPI underneath, you get:

* Path parameters (`{item_id}`) with automatic type conversion
* Request body validation via Pydantic models
* Response serialization
* Status code customization

## Inject services into controllers

Controllers support dependency injection, just like any other component:

```python
from py_spring_core import RestController, Component
from py_spring_core.core.entities.controllers.rest_controller import GetMapping


class UserService(Component):
    def get_all_users(self):
        return [{"id": 1, "name": "Alice"}]


class UserController(RestController):
    class Config:
        prefix = "/api/users"

    user_service: UserService  # Auto-injected!

    @GetMapping("/")
    def get_users(self):
        return {"users": self.user_service.get_all_users()}
```

This is where PySpring's DI shines — your controllers stay thin and delegate to services.

## Check the docs

After starting your application, open <a href="http://127.0.0.1:8000/docs" target="_blank">http://127.0.0.1:8000/docs</a> for Swagger UI, or `/redoc` for ReDoc. All your controller routes are documented automatically. ✨

## Recap

REST Controllers give you class-based API routing built on FastAPI.

* Extend `RestController` and set `Config.prefix`
* Use `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping`
* Inject services via type hints
* Automatic OpenAPI documentation
* Full request/response validation via Pydantic

Next, let's learn about **Bean Collections** — how to integrate third-party code into PySpring's DI system.
