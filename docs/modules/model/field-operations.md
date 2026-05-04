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

- Add suffixes (`_gt`, `_in`, `_like`, etc.) to field names in method signatures
- PySpring parses the suffix and generates the correct SQL operator
- Combine with `_and_` / `_or_` for multi-condition queries
- Use `List` parameter types for `_in` and `_not_in` operations
- Use SQL wildcards with `_like`
