# Built-in Read Operations

**Note**: PySpringModel currently only supports **read operations**. Write operations (Create, Update, Delete) are not yet implemented.

The `CrudRepository` provides a comprehensive set of built-in methods for common database read operations. These methods handle data retrieval with automatic session management and error handling.

## Read Operations

### Find by Primary Key

```python
class UserRepository(CrudRepository[int, User]):
    # Find a single entity by its primary key
    def find_by_id(self, id: int) -> Optional[User]: ...
    
    # Find multiple entities by their primary keys
    def find_all_by_ids(self, ids: List[int]) -> List[User]: ...
```

### Find All Records

```python
class UserRepository(CrudRepository[int, User]):
    # Find all entities in the table
    def find_all(self) -> List[User]: ...
```

### Usage Examples

```python
class UserService:
    user_repository: UserRepository
    
    def get_user_by_id(self, user_id: int) -> Optional[User]:
        return self.user_repository.find_by_id(user_id)
    
    def get_users_by_ids(self, user_ids: List[int]) -> List[User]:
        return self.user_repository.find_all_by_ids(user_ids)
    
    def get_all_users(self) -> List[User]:
        return self.user_repository.find_all()
```

## Write Operations (Not Yet Supported)

The following operations are planned but not yet implemented in PySpringModel:

### Save Single Entity
```python
# This functionality is not yet available
# def save(self, entity: User) -> User: ...
```

### Save Multiple Entities
```python
# This functionality is not yet available
# def save_all(self, entities: List[User]) -> List[User]: ...
```

### Delete Operations
```python
# This functionality is not yet available
# def delete(self, entity: User) -> None: ...
# def delete_by_id(self, id: int) -> None: ...
```

## Complete Read Operations Example

Here's a complete example showing all available read operations:

```python
from py_spring_model import PySpringModel, CrudRepository
from sqlmodel import Field
from typing import Optional, List

# Model definition
class User(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    name: str = Field()
    email: str = Field()
    age: int = Field()
    status: str = Field()

# Repository with read operations only
class UserRepository(CrudRepository[int, User]):
    # Dynamic queries
    def find_by_name(self, name: str) -> Optional[User]: ...
    def find_all_by_status(self, status: str) -> List[User]: ...

# Service using read operations only
class UserService:
    user_repository: UserRepository
    
    # Read operations
    def get_user_by_id(self, user_id: int) -> Optional[User]:
        return self.user_repository.find_by_id(user_id)
    
    def get_user_by_name(self, name: str) -> Optional[User]:
        return self.user_repository.find_by_name(name)
    
    def get_all_users(self) -> List[User]:
        return self.user_repository.find_all()
    
    def get_active_users(self) -> List[User]:
        return self.user_repository.find_all_by_status("active")
    
    def get_users_by_ids(self, user_ids: List[int]) -> List[User]:
        return self.user_repository.find_all_by_ids(user_ids)
```

## Session Management

PySpringModel automatically handles database sessions for read operations:

### Automatic Session Handling

```python
class UserService:
    user_repository: UserRepository
    
    def get_user_data(self, user_id: int) -> Optional[dict]:
        # Session is automatically managed for read operations
        user = self.user_repository.find_by_id(user_id)
        if user:
            return {
                "id": user.id,
                "name": user.name,
                "email": user.email,
                "age": user.age,
                "status": user.status
            }
        return None
```

## Error Handling

PySpringModel provides robust error handling for read operations:

### Common Error Scenarios

```python
class UserService:
    user_repository: UserRepository
    
    def safe_get_user(self, user_id: int) -> Optional[User]:
        try:
            return self.user_repository.find_by_id(user_id)
        except Exception as e:
            # Handle database connection errors, etc.
            print(f"Error retrieving user: {e}")
            return None
    
    def safe_get_users_by_status(self, status: str) -> List[User]:
        try:
            return self.user_repository.find_all_by_status(status)
        except Exception as e:
            # Handle database errors
            print(f"Error retrieving users: {e}")
            return []
```

## Performance Considerations

### Batch Reading

```python
class UserService:
    user_repository: UserRepository
    
    def get_multiple_users(self, user_ids: List[int]) -> List[User]:
        # Efficient batch reading
        return self.user_repository.find_all_by_ids(user_ids)
```

### Selective Loading

```python
class UserService:
    user_repository: UserRepository
    
    def get_user_names_only(self) -> List[str]:
        # Use custom queries for selective loading
        users = self.user_repository.find_all()
        return [user.name for user in users]
```

## Best Practices

1. **Use appropriate read methods**: Choose the right read method for your use case
2. **Handle errors gracefully**: Always handle potential exceptions in your service layer
3. **Use batch operations**: Prefer batch operations for multiple entities
4. **Monitor performance**: Use custom queries for complex operations that can't be handled by built-in methods
5. **Plan for future write operations**: Design your models and repositories with future write operations in mind
6. **Use SQLAlchemy for writes**: For now, use SQLAlchemy directly for any write operations you need 