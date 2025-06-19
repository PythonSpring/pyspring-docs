# Custom SQL Queries

For complex queries that can't be expressed through method names, PySpringModel supports custom SQL queries using the `@Query` decorator. This allows you to write raw SQL while still benefiting from PySpringModel's type safety and parameter handling.

## Basic Custom Queries

### Simple WHERE Clauses

```python
from py_spring_model import CrudRepository, Query
from typing import List

class UserRepository(CrudRepository[int, User]):
    @Query("SELECT * FROM user WHERE age > {min_age}")
    def find_users_older_than(self, min_age: int) -> List[User]: ...
    
    @Query("SELECT * FROM user WHERE age < {max_age}")
    def find_users_younger_than(self, max_age: int) -> List[User]: ...
    
    @Query("SELECT * FROM user WHERE email LIKE '%{domain}%'")
    def find_users_by_email_domain(self, domain: str) -> List[User]: ...
```

### Single Result Queries

For queries that should return at most one result, use `Optional[Model]` as the return type:

```python
class UserRepository(CrudRepository[int, User]):
    @Query("SELECT * FROM user WHERE email = {email} LIMIT 1")
    def get_user_by_email(self, email: str) -> Optional[User]: ...
    
    @Query("SELECT * FROM user WHERE name = {name} AND status = {status} LIMIT 1")
    def get_user_by_name_and_status(self, name: str, status: str) -> Optional[User]: ...
```

## Complex Custom Queries

### Multiple Conditions

```python
class UserRepository(CrudRepository[int, User]):
    @Query("SELECT * FROM user WHERE age BETWEEN {min_age} AND {max_age} AND status = {status}")
    def find_users_by_age_range_and_status(self, min_age: int, max_age: int, status: str) -> List[User]: ...
    
    @Query("SELECT * FROM user WHERE name LIKE %{name_pattern}% OR email LIKE %{email_pattern}%")
    def search_users_by_name_or_email(self, name_pattern: str, email_pattern: str) -> List[User]: ...
```

### Ordering and Limiting

```python
class UserRepository(CrudRepository[int, User]):
    @Query("SELECT * FROM user ORDER BY age DESC LIMIT {limit}")
    def find_oldest_users(self, limit: int) -> List[User]: ...
    
    @Query("SELECT * FROM user WHERE status = {status} ORDER BY name ASC LIMIT {limit}")
    def find_users_by_status_ordered(self, status: str, limit: int) -> List[User]: ...
```

### Aggregation Queries

```python
class UserRepository(CrudRepository[int, User]):
    @Query("SELECT COUNT(*) as count FROM user WHERE status = {status}")
    def count_users_by_status(self, status: str) -> int: ...
    
    @Query("SELECT AVG(age) as average_age FROM user WHERE status = {status}")
    def get_average_age_by_status(self, status: str) -> float: ...
    
    @Query("SELECT status, COUNT(*) as count FROM user GROUP BY status")
    def get_user_count_by_status(self) -> List[dict]: ...
```

## Parameter Substitution

The `@Query` decorator supports parameter substitution using `{parameter_name}` syntax:

### Basic Parameter Substitution

```python
class UserRepository(CrudRepository[int, User]):
    @Query("SELECT * FROM user WHERE name = {name}")
    def find_by_name(self, name: str) -> List[User]: ...
    
    @Query("SELECT * FROM user WHERE age > {min_age} AND age < {max_age}")
    def find_by_age_range(self, min_age: int, max_age: int) -> List[User]: ...
```

### String Pattern Matching

```python
class UserRepository(CrudRepository[int, User]):
    @Query("SELECT * FROM user WHERE name LIKE %{name_pattern}%")
    def search_by_name_pattern(self, name_pattern: str) -> List[User]: ...
    
    @Query("SELECT * FROM user WHERE email LIKE '{domain}%'")
    def find_users_by_email_prefix(self, domain: str) -> List[User]: ...
```

### Multiple Parameter Types

```python
class UserRepository(CrudRepository[int, User]):
    @Query("SELECT * FROM user WHERE age >= {min_age} AND age <= {max_age} AND status IN ({statuses})")
    def find_users_by_criteria(self, min_age: int, max_age: int, statuses: List[str]) -> List[User]: ...
```

## Advanced Query Features

### Joins

```python
class UserRepository(CrudRepository[int, User]):
    @Query("""
        SELECT u.* FROM user u 
        JOIN user_profile p ON u.id = p.user_id 
        WHERE p.city = {city}
    """)
    def find_users_by_city(self, city: str) -> List[User]: ...
```

### Subqueries

```python
class UserRepository(CrudRepository[int, User]):
    @Query("""
        SELECT * FROM user 
        WHERE age > (SELECT AVG(age) FROM user WHERE status = {status})
    """)
    def find_users_older_than_average(self, status: str) -> List[User]: ...
```

### Complex WHERE Clauses

```python
class UserRepository(CrudRepository[int, User]):
    @Query("""
        SELECT * FROM user 
        WHERE (age BETWEEN {min_age} AND {max_age}) 
        AND (status = {status} OR status = {alt_status})
        AND name IS NOT NULL
    """)
    def find_users_complex_criteria(self, min_age: int, max_age: int, status: str, alt_status: str) -> List[User]: ...
```

## Query Decorator Features

The `@Query` decorator provides several features:

### Type Safety

- Method parameters must match SQL parameters
- Return type inference for `Optional[Model]` and `List[Model]`
- Parameter validation at runtime

### Error Handling

- Validates required parameters are provided
- Checks parameter types match expected types
- Provides clear error messages for missing or invalid parameters

### SQL Injection Protection

- Parameters are properly escaped and bound
- Prevents SQL injection attacks
- Safe handling of user input

## Best Practices

### 1. Use Descriptive Method Names

```python
# Good
@Query("SELECT * FROM user WHERE age > {min_age}")
def find_users_older_than(self, min_age: int) -> List[User]: ...

# Avoid
@Query("SELECT * FROM user WHERE age > {age}")
def query1(self, age: int) -> List[User]: ...
```

### 2. Keep Queries Readable

```python
# Good - Multi-line for complex queries
@Query("""
    SELECT u.* FROM user u 
    JOIN user_profile p ON u.id = p.user_id 
    WHERE p.city = {city} AND u.status = {status}
""")
def find_users_by_city_and_status(self, city: str, status: str) -> List[User]: ...

# Avoid - Single line for complex queries
@Query("SELECT u.* FROM user u JOIN user_profile p ON u.id = p.user_id WHERE p.city = {city} AND u.status = {status}")
def find_users_by_city_and_status(self, city: str, status: str) -> List[User]: ...
```

### 3. Use Appropriate Return Types

```python
# Single result
@Query("SELECT * FROM user WHERE email = {email} LIMIT 1")
def get_user_by_email(self, email: str) -> Optional[User]: ...

# Multiple results
@Query("SELECT * FROM user WHERE status = {status}")
def find_users_by_status(self, status: str) -> List[User]: ...

# Aggregation result
@Query("SELECT COUNT(*) as count FROM user WHERE status = {status}")
def count_users_by_status(self, status: str) -> int: ...
```

### 4. Handle Edge Cases

```python
class UserRepository(CrudRepository[int, User]):
    @Query("SELECT * FROM user WHERE name LIKE %{name_pattern}% OR name IS NULL")
    def search_users_by_name_pattern(self, name_pattern: str) -> List[User]: ...
    
    @Query("SELECT * FROM user WHERE age >= {min_age} OR age IS NULL")
    def find_users_by_min_age(self, min_age: int) -> List[User]: ...
```

## Limitations

- Complex database-specific features may require raw SQLAlchemy usage
- Some advanced SQL features might not be supported
- Performance optimization for very complex queries may require manual tuning
- Database-specific SQL dialects may not be fully supported 