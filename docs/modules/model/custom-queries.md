# Custom Queries

When [dynamic queries](dynamic-queries.md) aren't expressive enough, use the `@Query` decorator to write raw SQL.

## Basic usage

```python
from py_spring_model import CrudRepository, Query
from typing import List, Optional

class UserRepository(CrudRepository[int, User]):
    @Query("SELECT * FROM user WHERE age > :min_age")
    def find_users_older_than(self, min_age: int) -> List[User]: ...

    @Query("SELECT * FROM user WHERE email LIKE '%' || :domain || '%'")
    def find_users_by_email_domain(self, domain: str) -> List[User]: ...
```

The `@Query` decorator:

1. Registers the method to be skipped by auto-implementation
2. At runtime, substitutes parameters into the SQL template using SQLAlchemy's `text()` bind parameters
3. Maps the result rows to your model class using Pydantic's `model_validate`

## Parameter substitution

Use `:parameter_name` syntax (SQLAlchemy bind parameters) in your SQL:

```python
@Query("SELECT * FROM user WHERE age BETWEEN :min_age AND :max_age")
def find_users_by_age_range(self, min_age: int, max_age: int) -> List[User]: ...

@Query("SELECT * FROM user WHERE name = :name AND status = :status LIMIT 1")
def get_user_by_name_and_status(self, name: str, status: str) -> Optional[User]: ...
```

Method parameters are validated against the SQL template at runtime — missing or mistyped parameters raise clear errors.

## Return types

### List of models

```python
@Query("SELECT * FROM user WHERE age > :min_age")
def find_older_users(self, min_age: int) -> List[User]: ...
```

### Optional single model

```python
@Query("SELECT * FROM user WHERE email = :email LIMIT 1")
def get_user_by_email(self, email: str) -> Optional[User]: ...
```

Returns `None` if no row matches.

### Scalar types

```python
@Query("SELECT COUNT(*) FROM user WHERE status = :status")
def count_by_status(self, status: str) -> int: ...
```

Supports `int`, `float`, `str`, and `bool` return types.

### None (fire-and-forget)

```python
@Query("DELETE FROM user WHERE status = :status", is_modifying=True)
def purge_by_status(self, status: str) -> None: ...
```

## Modifying queries

For INSERT, UPDATE, and DELETE operations, set `is_modifying=True`. This ensures the session commits after execution:

```python
@Query(
    "UPDATE user SET status = :new_status WHERE age < :max_age",
    is_modifying=True,
)
def deactivate_young_users(self, new_status: str, max_age: int) -> List[User]: ...

@Query(
    "DELETE FROM user WHERE status = :status",
    is_modifying=True,
)
def purge_users_by_status(self, status: str) -> List[User]: ...
```

!!! warning
    Without `is_modifying=True`, write operations will not be committed to the database.

## When to use @Query vs dynamic methods

| Scenario | Use |
|----------|-----|
| Simple equality, comparison, or membership filters | Dynamic methods |
| Joins, subqueries, aggregations | `@Query` |
| `BETWEEN`, `ORDER BY`, `LIMIT` in combination | `@Query` |
| INSERT/UPDATE/DELETE operations | `@Query` with `is_modifying=True` |
| Complex `WHERE` clauses | `@Query` |

## Combining with dynamic methods

A repository can mix dynamic methods and `@Query` methods freely:

```python
class UserRepository(CrudRepository[int, User]):
    # Dynamic — auto-implemented
    def find_by_name(self, name: str) -> Optional[User]: ...
    def find_all_by_status(self, status: str) -> List[User]: ...

    # Custom SQL
    @Query("SELECT * FROM user WHERE age BETWEEN :min AND :max ORDER BY age")
    def find_in_age_range(self, min: int, max: int) -> List[User]: ...

    @Query("UPDATE user SET status = 'archived' WHERE age > :age", is_modifying=True)
    def archive_old_users(self, age: int) -> None: ...
```
