# Auto CRUD Routes

PySpring Model can automatically generate RESTful CRUD endpoints for all your models. When enabled, every `PySpringModel` entity gets a full set of HTTP routes — no controllers or boilerplate required.

## Enable auto CRUD routes

Set `create_crud_routes` to `true` in your `application-properties.json`:

```json
{
    "py_spring_model": {
        "sqlalchemy_database_uri": "sqlite:///./app.db",
        "create_all_tables": true,
        "create_crud_routes": true
    }
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `create_crud_routes` | `bool` | `false` | Automatically expose CRUD REST endpoints for all models |

That's it. No additional code is needed. When the application starts, PySpring registers routes for every model that extends `PySpringModel` with `table=True`.

## How it works

1. On startup, `PySpringModelStarter` registers a `PySpringModelRestController`.
2. The controller discovers all `PySpringModel` subclasses via `PySpringModel.get_model_lookup()`.
3. For each model, it derives a **resource name** from the table name (the snake_case version of the class name) and registers CRUD routes under that name.

For example, given this model:

```python
from py_spring_model import PySpringModel, Field

class UserProfile(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    name: str = Field()
    email: str = Field()
    age: int = Field()
```

The table name is `user_profile` (auto-converted from `UserProfile`), and the following endpoints are registered:

## Generated endpoints

### Get by ID

```
GET /{resource_name}/{id}
```

Retrieve a single record by its primary key.

```
GET /user_profile/1
```

```json
{
    "id": 1,
    "name": "Alice",
    "email": "alice@example.com",
    "age": 30
}
```

### Get all (paginated)

```
GET /{resource_name}/{limit}/{offset}
```

Retrieve a paginated list of records.

```
GET /user_profile/10/0
```

```json
[
    {"id": 1, "name": "Alice", "email": "alice@example.com", "age": 30},
    {"id": 2, "name": "Bob", "email": "bob@example.com", "age": 25}
]
```

### Count

```
GET /{resource_name}/count
```

Get the total number of records.

```
GET /user_profile/count
```

```json
42
```

### Get multiple by IDs

```
POST /{resource_name}/ids
```

Retrieve multiple records by a list of IDs.

```
POST /user_profile/ids
Content-Type: application/json

{
    "ids": [1, 2, 3]
}
```

### Create

```
POST /{resource_name}
```

Create a new record.

```
POST /user_profile
Content-Type: application/json

{
    "name": "Charlie",
    "email": "charlie@example.com",
    "age": 28
}
```

### Batch create

```
POST /{resource_name}/batch
```

Create multiple records at once.

```
POST /user_profile/batch
Content-Type: application/json

[
    {"name": "Dave", "email": "dave@example.com", "age": 35},
    {"name": "Eve", "email": "eve@example.com", "age": 22}
]
```

### Update by ID

```
PUT /{resource_name}/{id}
```

Update an existing record by its primary key. The request body should contain the full model data.

```
PUT /user_profile/1
Content-Type: application/json

{
    "name": "Alice Updated",
    "email": "alice@new.com",
    "age": 31
}
```

### Delete by ID

```
DELETE /{resource_name}/{id}
```

Delete a single record. Returns `204 No Content` on success.

```
DELETE /user_profile/1
```

### Batch delete

```
DELETE /{resource_name}/batch
```

Delete multiple records by their IDs. Returns `204 No Content` on success.

```
DELETE /user_profile/batch
Content-Type: application/json

{
    "ids": [1, 2, 3]
}
```

## Endpoint summary

| Method | Path | Description |
|--------|------|-------------|
| `GET` | `/{resource}/count` | Count all records |
| `GET` | `/{resource}/{id}` | Get one by ID |
| `GET` | `/{resource}/{limit}/{offset}` | Get paginated list |
| `POST` | `/{resource}/ids` | Get multiple by IDs |
| `POST` | `/{resource}` | Create one |
| `POST` | `/{resource}/batch` | Create multiple |
| `PUT` | `/{resource}/{id}` | Update one by ID |
| `DELETE` | `/{resource}/{id}` | Delete one by ID |
| `DELETE` | `/{resource}/batch` | Delete multiple by IDs |

## Resource naming

The resource name in the URL is derived from the model's table name, which is the snake_case conversion of the class name:

| Class name | Resource name (URL) |
|------------|---------------------|
| `User` | `/user` |
| `UserProfile` | `/user_profile` |
| `OrderItem` | `/order_item` |

## ID type support

Auto CRUD routes support both `int` and `UUID` primary keys. The ID type is automatically detected from the model's `id` field annotation:

```python
from uuid import UUID, uuid4

class Article(PySpringModel, table=True):
    id: UUID = Field(default_factory=uuid4, primary_key=True)
    title: str = Field()
```

## Swagger UI

All generated routes include OpenAPI metadata (summary, description, and tags). When your application is running, you can explore the auto-generated endpoints in the interactive Swagger UI at:

```
http://localhost:8000/docs
```

Each model's endpoints are grouped under a tag matching the resource name, making it easy to browse and test.

## Complete example

```python
# models.py
from py_spring_model import PySpringModel, Field

class Product(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    name: str = Field()
    price: float = Field()
    category: str = Field()

class Order(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    product_id: int = Field()
    quantity: int = Field()

# main.py
from py_spring_core import PySpringApplication
from py_spring_model import PySpringModelStarter

def main():
    app = PySpringApplication(
        "./app-config.json",
        starters=[PySpringModelStarter()],
    )
    app.run()

if __name__ == "__main__":
    main()
```

```json
// application-properties.json
{
    "py_spring_model": {
        "sqlalchemy_database_uri": "sqlite:///./shop.db",
        "create_all_tables": true,
        "create_crud_routes": true
    }
}
```

With this setup, the application exposes full CRUD routes for both `/product` and `/order` automatically.

!!! tip
    Auto CRUD routes work alongside custom `CrudRepository` classes and `RestController` classes. You can use auto-generated routes for quick prototyping and add custom repositories or controllers for more complex business logic.

!!! note
    Auto CRUD routes use the same `@Transactional` session management as `CrudRepository`. Each request gets its own database session that is automatically cleaned up via the session middleware.
