# Dynamic Queries

PySpring Model automatically implements query methods based on their names. Declare a method signature in your repository — PySpring generates the SQL at startup.

## How it works

Define methods in your `CrudRepository` subclass with only a signature (no body):

```python
class UserRepository(CrudRepository[int, User]):
    def find_by_name(self, name: str) -> Optional[User]: ...
    def find_all_by_status(self, status: str) -> List[User]: ...
```

At startup, `CrudRepositoryImplementationService` parses each method name, extracts field names and operators, and generates the corresponding SQLAlchemy query. You never write the implementation.

## Naming conventions

### Prefixes

| Prefix | Return Type | Description |
|--------|-------------|-------------|
| `find_by_` | `Optional[Model]` | Returns a single result or `None` |
| `get_by_` | `Optional[Model]` | Same as `find_by_` |
| `find_all_by_` | `List[Model]` | Returns a list of results |
| `get_all_by_` | `List[Model]` | Same as `find_all_by_` |
| `count_by_` | `int` | Returns the count of matching records |
| `exists_by_` | `bool` | Returns `True` if any matching record exists |
| `delete_by_` | `int` | Deletes a single matching record, returns row count |
| `delete_all_by_` | `int` | Deletes all matching records, returns row count |

### Single field

```python
def find_by_name(self, name: str) -> Optional[User]: ...
# → WHERE name = :name

def find_all_by_status(self, status: str) -> List[User]: ...
# → WHERE status = :status
```

### Multiple fields with AND

```python
def find_by_name_and_email(self, name: str, email: str) -> Optional[User]: ...
# → WHERE name = :name AND email = :email

def find_all_by_age_and_status(self, age: int, status: str) -> List[User]: ...
# → WHERE age = :age AND status = :status
```

### Multiple fields with OR

```python
def find_by_name_or_email(self, name: str, email: str) -> Optional[User]: ...
# → WHERE name = :name OR email = :email

def find_all_by_status_or_age(self, status: str, age: int) -> List[User]: ...
# → WHERE status = :status OR age = :age
```

### Count, exists, and delete

```python
class UserRepository(CrudRepository[int, User]):
    # Count matching records
    def count_by_status(self, status: str) -> int: ...
    # → SELECT COUNT(*) FROM user WHERE status = :status

    # Check existence
    def exists_by_email(self, email: str) -> bool: ...
    # → SELECT COUNT(*) FROM user WHERE email = :email > 0

    # Delete single match
    def delete_by_name(self, name: str) -> int: ...
    # → DELETE FROM user WHERE name = :name

    # Delete all matches
    def delete_all_by_status(self, status: str) -> int: ...
    # → DELETE FROM user WHERE status = :status
```

These prefixes work with all the same features — multiple fields, `_and_`/`_or_` connectors, and [field operations](field-operations.md):

```python
class UserRepository(CrudRepository[int, User]):
    def count_by_status_and_age_gt(self, status: str, age: int) -> int: ...
    def exists_by_name_and_email(self, name: str, email: str) -> bool: ...
    def delete_all_by_status_in(self, status: List[str]) -> int: ...
```

## Parameter mapping

Method parameters are matched to field names extracted from the method name.

### Exact matching

Parameters must match the model field names:

```python
# Model has fields: name, age
def find_by_name_and_age(self, name: str, age: int) -> Optional[User]: ...  # OK
def find_by_name_and_age(self, age: int, name: str) -> Optional[User]: ...  # OK — order doesn't matter
```

### Plural parameter support

Parameters can use plural forms for a more natural API:

```python
def find_by_name_and_age(self, names: List[str], ages: List[int]) -> Optional[User]: ...  # OK
def find_by_name_and_age(self, name: str, ages: List[int]) -> Optional[User]: ...  # OK — mixed
```

Plural-to-singular mapping rules:

- **Exact match first** — if the parameter name exists as a field, use it directly
- **Regular plurals** — `names` maps to `name`, `ages` maps to `age`
- **Special cases** — `statuses` maps to `status`, `categories` maps to `category`
- **No ambiguity** — mapping fails if both singular and plural forms exist as fields

### Invalid parameters

```python
# These will raise errors at startup:
def find_by_name_and_age(self, username: str, user_age: int) -> Optional[User]: ...
# Error: parameters 'username' and 'user_age' don't match fields 'name' and 'age'
```

## Return types

The prefix determines the expected return type:

| Prefix pattern | Required return type |
|---------------|---------------------|
| `find_by_*` / `get_by_*` | `Optional[Model]` |
| `find_all_by_*` / `get_all_by_*` | `List[Model]` |
| `count_by_*` | `int` |
| `exists_by_*` | `bool` |
| `delete_by_*` / `delete_all_by_*` | `int` |

## Combining with field operations

Dynamic queries can be combined with [field operations](field-operations.md):

```python
def find_by_age_gt_and_status_in(self, age: int, status: List[str]) -> Optional[User]: ...
# → WHERE age > :age AND status IN :status
```

See the [Field Operations](field-operations.md) page for all available operators.

## Relationship traversal

Dynamic queries can traverse SQLModel `Relationship` fields to query across related models with automatic joins:

```python
def find_all_by_members_status(self, status: str) -> List[Team]: ...
# → SELECT DISTINCT team.* FROM team JOIN user ON ... WHERE user.status = :status
```

See the [Relationship Queries](relationship-queries.md) page for full details.

## SkipAutoImplementation

If you need a custom implementation for a method that matches the naming convention, use `@SkipAutoImplementation`:

```python
from py_spring_model import CrudRepository, SkipAutoImplmentation

class UserRepository(CrudRepository[int, User]):
    # Auto-implemented
    def find_by_name(self, name: str) -> Optional[User]: ...

    # Custom implementation — skipped by auto-implementation
    @SkipAutoImplmentation
    def find_by_email(self, email: str) -> Optional[User]:
        with PySpringModel.create_session() as session:
            return session.exec(
                select(User).where(User.email == email.lower())
            ).first()
```

Use cases for `@SkipAutoImplementation`:

- Custom business logic before or after the query
- Complex queries with joins or subqueries
- Performance-optimized queries
- Integration with external services
