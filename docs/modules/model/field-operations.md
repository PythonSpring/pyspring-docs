# Field Operations

Field operations extend [dynamic queries](dynamic-queries.md) with comparison, membership, and pattern-matching operators. Add a suffix to any field name in a method signature to change the generated SQL condition.

## Supported operations

| Operation | Suffix | SQL | Example Method |
|-----------|--------|-----|----------------|
| Equals | *(default)* | `field = ?` | `find_by_name` |
| Greater than | `_gt` | `field > ?` | `find_by_age_gt` |
| Greater or equal | `_gte` | `field >= ?` | `find_by_age_gte` |
| Less than | `_lt` | `field < ?` | `find_by_age_lt` |
| Less or equal | `_lte` | `field <= ?` | `find_by_age_lte` |
| Not equals | `_ne` | `field != ?` | `find_by_status_ne` |
| In | `_in` | `field IN (?)` | `find_by_status_in` |
| Not in | `_not_in` | `field NOT IN (?)` | `find_by_category_not_in` |
| Like | `_like` | `field LIKE ?` | `find_by_name_like` |
| Not like | `_not_like` | `field NOT LIKE ?` | `find_by_name_not_like` |
| Between | `_between` | `field BETWEEN ? AND ?` | `find_all_by_age_between` |
| Is null | `_is_null` | `field IS NULL` | `find_all_by_email_is_null` |
| Is not null | `_is_not_null` | `field IS NOT NULL` | `find_all_by_email_is_not_null` |
| Starts with | `_starts_with` | `field LIKE 'val%'` | `find_by_name_starts_with` |
| Ends with | `_ends_with` | `field LIKE '%val'` | `find_by_name_ends_with` |
| Contains | `_contains` | `field LIKE '%val%'` | `find_by_name_contains` |

## Comparison operations

```python
class UserRepository(CrudRepository[int, User]):
    # Greater than
    def find_by_age_gt(self, age: int) -> Optional[User]: ...
    def find_all_by_age_gt(self, age: int) -> List[User]: ...

    # Greater or equal
    def find_all_by_age_gte(self, age: int) -> List[User]: ...

    # Less than
    def find_by_age_lt(self, age: int) -> Optional[User]: ...

    # Less or equal
    def find_by_age_lte(self, age: int) -> Optional[User]: ...

    # Not equals
    def find_by_status_ne(self, status: str) -> Optional[User]: ...
```

Usage:

```python
adults = user_repo.find_all_by_age_gte(age=18)
young_user = user_repo.find_by_age_lt(age=25)
non_active = user_repo.find_by_status_ne(status="active")
```

## Membership operations

### IN

The parameter type should be `List`:

```python
class UserRepository(CrudRepository[int, User]):
    def find_all_by_status_in(self, status: List[str]) -> List[User]: ...
    def find_all_by_id_in(self, id: List[int]) -> List[User]: ...
```

Usage:

```python
users = user_repo.find_all_by_status_in(status=["active", "pending"])
users = user_repo.find_all_by_id_in(id=[1, 2, 3, 5])
```

### NOT IN

```python
class UserRepository(CrudRepository[int, User]):
    def find_all_by_category_not_in(self, category: List[str]) -> List[User]: ...
```

Usage:

```python
non_employees = user_repo.find_all_by_category_not_in(
    category=["employee", "intern"]
)
```

## Pattern matching

### LIKE

```python
class UserRepository(CrudRepository[int, User]):
    def find_by_name_like(self, name: str) -> Optional[User]: ...
    def find_all_by_name_like(self, name: str) -> List[User]: ...
```

Usage:

```python
johns = user_repo.find_all_by_name_like(name="%John%")
gmail_users = user_repo.find_all_by_email_like(email="%@gmail.com")
```

Use SQL wildcard characters (`%` for any sequence, `_` for single character) in the parameter value.

### NOT LIKE

```python
class UserRepository(CrudRepository[int, User]):
    def find_all_by_name_not_like(self, name: str) -> List[User]: ...
```

Usage:

```python
non_johns = user_repo.find_all_by_name_not_like(name="%John%")
```

### STARTS WITH

Automatically appends `%` to the parameter value:

```python
class UserRepository(CrudRepository[int, User]):
    def find_all_by_name_starts_with(self, name: str) -> List[User]: ...
```

Usage:

```python
users = user_repo.find_all_by_name_starts_with(name="Jo")
# → WHERE name LIKE 'Jo%'
```

### ENDS WITH

Automatically prepends `%` to the parameter value:

```python
class UserRepository(CrudRepository[int, User]):
    def find_all_by_email_ends_with(self, email: str) -> List[User]: ...
```

Usage:

```python
gmail_users = user_repo.find_all_by_email_ends_with(email="@gmail.com")
# → WHERE email LIKE '%@gmail.com'
```

### CONTAINS

Automatically wraps the parameter value with `%`:

```python
class UserRepository(CrudRepository[int, User]):
    def find_all_by_name_contains(self, name: str) -> List[User]: ...
```

Usage:

```python
users = user_repo.find_all_by_name_contains(name="oh")
# → WHERE name LIKE '%oh%'
```

!!! tip
    `_starts_with`, `_ends_with`, and `_contains` handle the `%` wildcards for you — unlike `_like`, you pass plain strings without wildcards.

## Range operations

### BETWEEN

Requires two parameters — `min_{field}` and `max_{field}`:

```python
class UserRepository(CrudRepository[int, User]):
    def find_all_by_age_between(self, min_age: int, max_age: int) -> List[User]: ...
```

Usage:

```python
users = user_repo.find_all_by_age_between(min_age=18, max_age=30)
# → WHERE age BETWEEN 18 AND 30
```

The parameter names must follow the `min_` / `max_` prefix convention matching the field name.

## Null checks

### IS NULL

No parameter required — the field is checked for `NULL`:

```python
class UserRepository(CrudRepository[int, User]):
    def find_all_by_email_is_null(self) -> List[User]: ...
```

Usage:

```python
users_without_email = user_repo.find_all_by_email_is_null()
# → WHERE email IS NULL
```

### IS NOT NULL

```python
class UserRepository(CrudRepository[int, User]):
    def find_all_by_email_is_not_null(self) -> List[User]: ...
```

Usage:

```python
users_with_email = user_repo.find_all_by_email_is_not_null()
# → WHERE email IS NOT NULL
```

!!! note
    `_is_null` and `_is_not_null` operations do not take any parameters — the method signature should not include the corresponding field as a parameter.

## Combining operations

Field operations can be combined with `and` / `or` connectors:

```python
class UserRepository(CrudRepository[int, User]):
    # AND combination
    def find_by_age_gt_and_status_in(
        self, age: int, status: List[str]
    ) -> Optional[User]: ...

    # OR combination
    def find_by_salary_gte_or_category_in(
        self, salary: float, category: List[str]
    ) -> Optional[User]: ...

    # Multiple comparisons
    def find_all_by_age_gte_and_age_lte(
        self, age: int, age: int
    ) -> List[User]: ...
```

Usage:

```python
target_users = user_repo.find_by_age_gt_and_status_in(
    age=30,
    status=["active", "pending"]
)
```

## Summary

- Add suffixes (`_gt`, `_in`, `_like`, `_between`, `_is_null`, etc.) to field names in method signatures
- PySpring parses the suffix and generates the correct SQL operator
- Combine with `_and_` / `_or_` for multi-condition queries
- Use `List` parameter types for `_in` and `_not_in` operations
- Use SQL wildcards with `_like` and `_not_like`
- Use `_starts_with`, `_ends_with`, `_contains` for automatic wildcard handling
- Use `min_` / `max_` prefixed parameters for `_between` operations
- `_is_null` and `_is_not_null` require no parameters
