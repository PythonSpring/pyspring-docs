# Dynamic Query Generation

PySpringModel automatically implements query methods based on their names. The method names follow a specific pattern that allows you to define queries without writing any SQL code.

## Method Naming Conventions

The dynamic query generation follows these naming conventions:

- **Prefixes**: `find_by_`, `get_by_`, `find_all_by_`, `get_all_by_`
- **Single field**: `find_by_name` → `WHERE name = ?`
- **Multiple fields with AND**: `find_by_name_and_email` → `WHERE name = ? AND email = ?`
- **Multiple fields with OR**: `find_by_name_or_email` → `WHERE name = ? OR email = ?`
- **Return types**:
  - `find_by_*` and `get_by_*` return `Optional[Model]`
  - `find_all_by_*` and `get_all_by_*` return `List[Model]`

## Single Result Queries

These methods return `Optional[Model]` and are used when you expect at most one result:

### Find by Single Field

```python
class UserRepository(CrudRepository[int, User]):
    def find_by_name(self, name: str) -> Optional[User]: ...
    def get_by_email(self, email: str) -> Optional[User]: ...
    def find_by_age(self, age: int) -> Optional[User]: ...
    def get_by_status(self, status: str) -> Optional[User]: ...
```

### Find by Multiple Fields with AND Condition

```python
class UserRepository(CrudRepository[int, User]):
    def find_by_name_and_email(self, name: str, email: str) -> Optional[User]: ...
    def get_by_age_and_status(self, age: int, status: str) -> Optional[User]: ...
    def find_by_name_and_age_and_status(self, name: str, age: int, status: str) -> Optional[User]: ...
```

### Find by Multiple Fields with OR Condition

```python
class UserRepository(CrudRepository[int, User]):
    def find_by_name_or_email(self, name: str, email: str) -> Optional[User]: ...
    def get_by_status_or_age(self, status: str, age: int) -> Optional[User]: ...
    def find_by_name_or_email_or_status(self, name: str, email: str, status: str) -> Optional[User]: ...
```

## Multiple Result Queries

These methods return `List[Model]` and are used when you expect multiple results:

### Find All by Single Field

```python
class UserRepository(CrudRepository[int, User]):
    def find_all_by_status(self, status: str) -> List[User]: ...
    def get_all_by_age(self, age: int) -> List[User]: ...
    def find_all_by_name(self, name: str) -> List[User]: ...
    def get_all_by_email(self, email: str) -> List[User]: ...
```

### Find All by Multiple Fields with AND Condition

```python
class UserRepository(CrudRepository[int, User]):
    def find_all_by_age_and_status(self, age: int, status: str) -> List[User]: ...
    def get_all_by_name_and_email(self, name: str, email: str) -> List[User]: ...
    def find_all_by_name_and_age_and_status(self, name: str, age: int, status: str) -> List[User]: ...
```

### Find All by Multiple Fields with OR Condition

```python
class UserRepository(CrudRepository[int, User]):
    def find_all_by_status_or_age(self, status: str, age: int) -> List[User]: ...
    def get_all_by_name_or_email(self, name: str, email: str) -> List[User]: ...
    def find_all_by_name_or_email_or_status(self, name: str, email: str, status: str) -> List[User]: ...
```

## Usage Examples

Here's how to use these dynamic queries in your service layer:

```python
class UserService:
    user_repository: UserRepository
    
    def get_user_by_name(self, name: str) -> Optional[User]:
        return self.user_repository.find_by_name(name)
    
    def get_user_by_email_and_status(self, email: str, status: str) -> Optional[User]:
        return self.user_repository.find_by_email_and_status(email, status)
    
    def get_active_users(self) -> List[User]:
        return self.user_repository.find_all_by_status("active")
    
    def get_users_by_age_and_status(self, age: int, status: str) -> List[User]:
        return self.user_repository.find_all_by_age_and_status(age, status)
    
    def search_users_by_name_or_email(self, name: str, email: str) -> List[User]:
        return self.user_repository.find_all_by_name_or_email(name, email)
```

## Field Name Mapping

PySpringModel automatically maps method parameter names to database column names:

- Method parameter names are converted to lowercase for database column matching
- Underscores in parameter names are preserved
- The framework handles the mapping between Python naming conventions and SQL naming conventions

## Generated SQL Examples

Here are examples of the SQL queries that PySpringModel generates for different method names:

| Method Name | Generated SQL |
|-------------|---------------|
| `find_by_name(name)` | `SELECT * FROM user WHERE name = ?` |
| `find_by_email_and_status(email, status)` | `SELECT * FROM user WHERE email = ? AND status = ?` |
| `find_by_name_or_email(name, email)` | `SELECT * FROM user WHERE name = ? OR email = ?` |
| `find_all_by_status(status)` | `SELECT * FROM user WHERE status = ?` |
| `find_all_by_age_and_status(age, status)` | `SELECT * FROM user WHERE age = ? AND status = ?` |
| `find_all_by_name_or_email_or_status(name, email, status)` | `SELECT * FROM user WHERE name = ? OR email = ? OR status = ?` |

## Best Practices

1. **Use descriptive method names**: Make your method names clear and descriptive
2. **Follow naming conventions**: Stick to the established prefixes and patterns
3. **Consider return types**: Use `find_by_` for single results and `find_all_by_` for multiple results
4. **Use AND for related conditions**: Use `_and_` when conditions should all be true
5. **Use OR for alternatives**: Use `_or_` when any condition can be true
6. **Keep it simple**: For complex queries, consider using custom SQL with the `@Query` decorator

## Limitations

- Method names must follow the exact naming pattern
- Field names in method parameters must match database column names
- Complex queries with joins, aggregations, or subqueries should use custom SQL
- The framework doesn't support dynamic field selection or complex WHERE clauses beyond AND/OR combinations 