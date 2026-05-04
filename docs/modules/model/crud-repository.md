# CRUD Repository

`CrudRepository` provides built-in database operations for any `PySpringModel` entity. All methods are transactional by default.

## Setup

```python
from py_spring_model import CrudRepository, PySpringModel, Field
from typing import Optional, List

class User(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    name: str = Field()
    email: str = Field()

class UserRepository(CrudRepository[int, User]):
    pass  # All built-in methods are available
```

The type parameters `CrudRepository[int, User]` define:

- `int` — the primary key type (supports `int` and `UUID`)
- `User` — the model class

## Read operations

### `find_by_id`

Retrieve a single entity by its primary key.

```python
user = user_repository.find_by_id(1)  # Optional[User]
```

Returns `None` if no entity is found.

### `find_all_by_ids`

Retrieve multiple entities by their primary keys.

```python
users = user_repository.find_all_by_ids([1, 2, 3])  # list[User]
```

### `find_all`

Retrieve all entities, with optional pagination and ordering.

```python
# All records
all_users = user_repository.find_all()

# With pagination
page = user_repository.find_all(offset=0, limit=10)

# With ordering
sorted_users = user_repository.find_all(order_by="name", ascending=True)

# Combined
page_sorted = user_repository.find_all(
    offset=20, limit=10, order_by="age", ascending=False
)
```

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `offset` | `Optional[int]` | `None` | Number of records to skip |
| `limit` | `Optional[int]` | `None` | Maximum number of records to return |
| `order_by` | `Optional[str]` | `None` | Column name to sort by |
| `ascending` | `bool` | `True` | Sort direction |

### `count`

Count total records.

```python
total = user_repository.count()  # int
```

### `count_by`

Count records matching a filter.

```python
active_count = user_repository.count_by({"status": "active"})  # int
```

### `exists_by_id`

Check if an entity exists.

```python
exists = user_repository.exists_by_id(1)  # bool
```

## Write operations

### `save`

Save a single entity (insert).

```python
user = User(name="Alice", email="alice@example.com")
saved_user = user_repository.save(user)
```

### `save_all`

Save multiple entities.

```python
users = [
    User(name="Alice", email="alice@example.com"),
    User(name="Bob", email="bob@example.com"),
]
saved = user_repository.save_all(users)
```

### `upsert`

Insert or update based on a query condition.

```python
user = User(name="Alice", email="alice@new.com")
user_repository.upsert(user, {"email": "alice@example.com"})
```

If an entity matching `{"email": "alice@example.com"}` exists, its attributes are updated. Otherwise, a new entity is inserted.

## Delete operations

### `delete`

Delete a single entity.

```python
success = user_repository.delete(user)  # bool
```

### `delete_by_id`

Delete by primary key.

```python
success = user_repository.delete_by_id(1)  # bool
```

### `delete_all`

Delete multiple entities.

```python
success = user_repository.delete_all([user1, user2])  # bool
```

### `delete_all_by_ids`

Delete multiple entities by their primary keys.

```python
success = user_repository.delete_all_by_ids([1, 2, 3])  # bool
```

## Method summary

| Method | Return Type | Description |
|--------|-------------|-------------|
| `find_by_id(id)` | `Optional[T]` | Find by primary key |
| `find_all_by_ids(ids)` | `list[T]` | Find multiple by primary keys |
| `find_all(...)` | `list[T]` | Find all with optional pagination/ordering |
| `count()` | `int` | Count all records |
| `count_by(query)` | `int` | Count records matching filter |
| `exists_by_id(id)` | `bool` | Check existence by primary key |
| `save(entity)` | `T` | Save a single entity |
| `save_all(entities)` | `list[T]` | Save multiple entities |
| `upsert(entity, query)` | `T` | Insert or update |
| `delete(entity)` | `bool` | Delete a single entity |
| `delete_by_id(id)` | `bool` | Delete by primary key |
| `delete_all(entities)` | `bool` | Delete multiple entities |
| `delete_all_by_ids(ids)` | `bool` | Delete multiple by primary keys |

All methods are wrapped with `@Transactional` and use `SessionContextHolder` for session management. See [Transaction Management](transactions.md) for details on propagation behavior.
