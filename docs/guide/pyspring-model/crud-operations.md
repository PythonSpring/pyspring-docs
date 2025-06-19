# CrudRepository Class Documentation

## Overview
The `CrudRepository` class is a generic CRUD (Create, Read, Update, Delete) repository implementation that provides common database operations for a single SQLModel entity. It is designed to handle basic CRUD operations on a single database table using the SQLModel library, which simplifies session management and transaction handling.

## Methods

### Read Operations

#### `find_by_id(id: ID) -> Optional[T]`
Retrieves a single entity by its ID.

```python
# Example usage
user_repository = UserRepository()

# Find user with ID 1
user = user_repository.find_by_id(1)
if user:
    print(f"Found user: {user.name}")
else:
    print("User not found")

# Find user with ID 999 (doesn't exist)
missing_user = user_repository.find_by_id(999)  # Returns None
```

#### `find_all_by_ids(ids: list[ID]) -> list[T]`
Retrieves a list of entities by their IDs.

```python
# Example usage
user_repository = UserRepository()

# Find multiple users by their IDs
user_ids = [1, 2, 3, 5]
users = user_repository.find_all_by_ids(user_ids)
print(f"Found {len(users)} users")

# Empty list if no IDs provided
no_users = user_repository.find_all_by_ids([])  # Returns []

# Mix of existing and non-existing IDs
mixed_ids = [1, 999, 2, 888]  # Only returns users with IDs 1 and 2
found_users = user_repository.find_all_by_ids(mixed_ids)
```

#### `find_all() -> list[T]`
Retrieves all entities from the table.

```python
# Example usage
user_repository = UserRepository()

# Get all users
all_users = user_repository.find_all()
print(f"Total users in database: {len(all_users)}")

# Iterate through all users
for user in all_users:
    print(f"User: {user.name} ({user.email})")
```

### Write Operations

#### `save(entity: T) -> T`
Saves a single entity to the database (insert or update).

```python
# Example usage
user_repository = UserRepository()

# Create and save a new user
new_user = User(name="John Doe", email="john@example.com", age=30)
saved_user = user_repository.save(new_user)
print(f"Saved user with ID: {saved_user.id}")

# Update an existing user
existing_user = user_repository.find_by_id(1)
existing_user.age = 31
updated_user = user_repository.save(existing_user)
print(f"Updated user: {updated_user.name}, new age: {updated_user.age}")
```

#### `save_all(entities: Iterable[T]) -> bool`
Saves multiple entities to the database in a single transaction.

```python
# Example usage
user_repository = UserRepository()

# Create multiple users
users_to_save = [
    User(name="Alice", email="alice@example.com", age=25),
    User(name="Bob", email="bob@example.com", age=28),
    User(name="Charlie", email="charlie@example.com", age=32)
]

# Save all users at once
success = user_repository.save_all(users_to_save)
if success:
    print("All users saved successfully")
else:
    print("Failed to save users")

# Can also use with existing entities for bulk updates
existing_users = user_repository.find_all_by_ids([1, 2, 3])
for user in existing_users:
    user.status = "updated"
bulk_update_success = user_repository.save_all(existing_users)
```

#### `delete(entity: T) -> bool`
Deletes a single entity from the database.

```python
# Example usage
user_repository = UserRepository()

# Find and delete a user
user_to_delete = user_repository.find_by_id(1)
if user_to_delete:
    success = user_repository.delete(user_to_delete)
    if success:
        print("User deleted successfully")
    else:
        print("Failed to delete user")

# Delete a user object directly
user = User(id=2, name="Test User", email="test@example.com", age=25)
deleted = user_repository.delete(user)  # Returns True if successful
```

#### `delete_all(entities: Iterable[T]) -> bool`
Deletes multiple entities from the database.

```python
# Example usage
user_repository = UserRepository()

# Delete multiple users
users_to_delete = user_repository.find_all_by_ids([1, 2, 3])
success = user_repository.delete_all(users_to_delete)
if success:
    print(f"Deleted {len(users_to_delete)} users")

# Delete users based on criteria
inactive_users = user_repository.find_all_by_status("inactive")
deletion_success = user_repository.delete_all(inactive_users)
```

#### `delete_by_id(_id: ID) -> bool`
Deletes an entity by its ID.

```python
# Example usage
user_repository = UserRepository()

# Delete user by ID
user_id = 1
success = user_repository.delete_by_id(user_id)
if success:
    print(f"User with ID {user_id} deleted")
else:
    print(f"User with ID {user_id} not found or couldn't be deleted")

# Delete multiple users by ID (one at a time)
user_ids_to_delete = [2, 3, 4]
for user_id in user_ids_to_delete:
    if user_repository.delete_by_id(user_id):
        print(f"Deleted user {user_id}")
```

#### `delete_all_by_ids(ids: list[ID]) -> bool`
Deletes multiple entities by their IDs.

```python
# Example usage
user_repository = UserRepository()

# Delete multiple users by their IDs
ids_to_delete = [1, 2, 3, 4, 5]
success = user_repository.delete_all_by_ids(ids_to_delete)
if success:
    print(f"Successfully deleted users with IDs: {ids_to_delete}")

# Handle empty list
empty_deletion = user_repository.delete_all_by_ids([])  # Returns True (no-op)

# Mix of existing and non-existing IDs
mixed_ids = [1, 999, 2, 888]
result = user_repository.delete_all_by_ids(mixed_ids)  # Deletes existing ones
```

#### `upsert(entity: T, query_by: dict[str, Any]) -> T`
Performs an upsert operation (insert if not exists, update if exists).

```python
# Example usage
user_repository = UserRepository()

# Upsert by email - create new user if email doesn't exist
user_data = User(name="Jane Doe", email="jane@example.com", age=27)
saved_user = user_repository.upsert(user_data, query_by={"email": "jane@example.com"})
print(f"Upserted user: {saved_user.name} with ID: {saved_user.id}")

# Upsert by email - update existing user if email exists
updated_data = User(name="Jane Smith", email="jane@example.com", age=28)
updated_user = user_repository.upsert(updated_data, query_by={"email": "jane@example.com"})
print(f"Updated user name to: {updated_user.name}")

# Upsert by multiple fields
user = User(name="Bob Johnson", email="bob@company.com", age=35, department="IT")
result = user_repository.upsert(
    user, 
    query_by={"email": "bob@company.com", "department": "IT"}
)

# Upsert with complex query criteria
existing_user = User(name="Admin User", email="admin@system.com", role="admin")
admin_user = user_repository.upsert(
    existing_user,
    query_by={"role": "admin", "email": "admin@system.com"}
)
```

## Usage

The `CrudRepository` class provides a powerful foundation for database operations in PySpring applications. Here's a comprehensive guide on how to use it effectively in different scenarios.

### Basic Repository Setup

First, define your entity model and create a repository:

```python
from py_spring_model import PySpringModel, CrudRepository
from sqlmodel import Field
from typing import Optional

# Define your entity model
class User(PySpringModel, table=True):
    id: Optional[int] = Field(default=None, primary_key=True)
    name: str = Field()
    email: str = Field(unique=True)
    age: int = Field()
    status: str = Field(default="active")

# Create a repository for the User entity
class UserRepository(CrudRepository[int, User]):
    pass  # Inherits all CRUD operations
```

### Service Layer Integration

Use the repository in your service layer for business logic:

```python
from typing import List, Optional
from py_spring_core import Component

class UserService(Component):
    user_repository: UserRepository
    def create_user(self, name: str, email: str, age: int) -> User:
        """Create a new user"""
        user = User(name=name, email=email, age=age)
        return self.user_repository.save(user)
    
    def get_user_by_id(self, user_id: int) -> Optional[User]:
        """Retrieve a user by ID"""
        return self.user_repository.find_by_id(user_id)
    
    def get_all_users(self) -> List[User]:
        """Get all users"""
        return self.user_repository.find_all()
    
    def update_user_status(self, user_id: int, new_status: str) -> Optional[User]:
        """Update a user's status"""
        user = self.user_repository.find_by_id(user_id)
        if user:
            user.status = new_status
            return self.user_repository.save(user)
        return None
    
    def delete_user(self, user_id: int) -> bool:
        """Delete a user by ID"""
        return self.user_repository.delete_by_id(user_id)
```

### Batch Operations

Efficiently handle multiple entities with batch operations:

```python
from py_spring_core import Component

class UserService(Component):
    user_repository: UserRepository
    
    def create_multiple_users(self, user_data: List[dict]) -> bool:
        """Create multiple users in a single transaction"""
        users = [User(**data) for data in user_data]
        return self.user_repository.save_all(users)
    
    def get_users_by_ids(self, user_ids: List[int]) -> List[User]:
        """Get multiple users by their IDs"""
        return self.user_repository.find_all_by_ids(user_ids)
    
    def delete_multiple_users(self, user_ids: List[int]) -> bool:
        """Delete multiple users by their IDs"""
        return self.user_repository.delete_all_by_ids(user_ids)
    
    def bulk_status_update(self, user_ids: List[int], new_status: str) -> bool:
        """Update status for multiple users"""
        users = self.user_repository.find_all_by_ids(user_ids)
        for user in users:
            user.status = new_status
        return self.user_repository.save_all(users)
```

### Upsert Operations

Handle scenarios where you need to insert or update based on existing data:

```python
from py_spring_core import Component

class UserService(Component):
    user_repository: UserRepository
    
    def create_or_update_user_by_email(self, name: str, email: str, age: int) -> User:
        """Create a new user or update existing one based on email"""
        user = User(name=name, email=email, age=age)
        return self.user_repository.upsert(user, query_by={"email": email})
    
    def sync_user_data(self, external_users: List[dict]) -> List[User]:
        """Sync users from external source"""
        synced_users = []
        for user_data in external_users:
            user = User(**user_data)
            synced_user = self.user_repository.upsert(
                user, 
                query_by={"email": user_data["email"]}
            )
            synced_users.append(synced_user)
        return synced_users
```

### Error Handling and Validation

Implement proper error handling in your service methods:

```python
import logging
from typing import Union, Dict, Any

class UserService:
    def __init__(self, user_repository: UserRepository):
        self.user_repository = user_repository
        self.logger = logging.getLogger(__name__)
    
    def safe_create_user(self, name: str, email: str, age: int) -> Union[User, Dict[str, Any]]:
        """Safely create a user with error handling"""
        try:
            # Validate input
            if not name or not email:
                return {"error": "Name and email are required"}
            
            if age < 0:
                return {"error": "Age must be non-negative"}
            
            # Create user
            user = User(name=name, email=email, age=age)
            saved_user = self.user_repository.save(user)
            
            self.logger.info(f"Created user: {saved_user.id}")
            return saved_user
            
        except Exception as e:
            self.logger.error(f"Error creating user: {str(e)}")
            return {"error": f"Failed to create user: {str(e)}"}
    
    def safe_delete_user(self, user_id: int) -> Dict[str, Any]:
        """Safely delete a user with proper error handling"""
        try:
            # Check if user exists
            user = self.user_repository.find_by_id(user_id)
            if not user:
                return {"success": False, "message": "User not found"}
            
            # Delete user
            if self.user_repository.delete_by_id(user_id):
                self.logger.info(f"Deleted user: {user_id}")
                return {"success": True, "message": "User deleted successfully"}
            else:
                return {"success": False, "message": "Failed to delete user"}
                
        except Exception as e:
            self.logger.error(f"Error deleting user {user_id}: {str(e)}")
            return {"success": False, "message": f"Error: {str(e)}"}
```

### Advanced Usage Patterns

#### Repository Inheritance

Extend `CrudRepository` for domain-specific operations:

```python
from py_spring_model import CrudRepository
from typing import List
from models import User

class UserRepository(CrudRepository[int, User]):
    def find_active_users(self) -> List[User]:
        """Custom method to find active users"""
        # This would be implemented using dynamic queries
        return self.find_all_by_status("active")
    
    def count_users_by_status(self, status: str) -> int:
        """Count users by status"""
        users = self.find_all_by_status(status)
        return len(users)
```

#### Multiple Repository Coordination

Coordinate multiple repositories in complex operations:

```python
from py_spring_core import Component

class UserManagementService(Component):
    user_repository: UserRepository
    profile_repository: ProfileRepository
    audit_repository: AuditRepository
        
    def create_user_with_profile(self, user_data: dict, profile_data: dict) -> User:
        """Create user and associated profile"""
        try:
            # Create user
            user = User(**user_data)
            saved_user = self.user_repository.save(user)
            
            # Create profile
            profile_data["user_id"] = saved_user.id
            profile = Profile(**profile_data)
            self.profile_repository.save(profile)
            
            # Log audit
            audit = AuditLog(
                action="user_created",
                entity_id=saved_user.id,
                timestamp=datetime.now()
            )
            self.audit_repository.save(audit)
            
            return saved_user
            
        except Exception as e:
            # Handle rollback or cleanup
            self.logger.error(f"Error in user creation: {str(e)}")
            raise
```

#### Selective Loading

```python
from py_spring_core import Component

class UserService(Component):
    user_repository: UserRepository
    
    def get_user_summaries(self) -> List[Dict[str, Any]]:
        """Get lightweight user summaries"""
        users = self.user_repository.find_all()
        return [
            {
                "id": user.id,
                "name": user.name,
                "status": user.status
            }
            for user in users
        ]
```

## Usage Example

Here is a basic example of how to use the `CrudRepository` class with a `HeroRepository`:

```python
from src.repository import HeroRepository
from models import Hero  # Assuming Hero is a defined SQLModel

# Create a HeroRepository instance
hero_repository = HeroRepository()

# Create a new hero
new_hero = Hero(id=1, name="Superman")

# Save the hero
hero_repository.save(new_hero)

# Find a hero by ID
found_hero = hero_repository.find_by_id(1)

# Delete a hero
hero_repository.delete(new_hero)
```

## Best Practices

### 1. Repository Design
- Keep repositories focused on data access operations
- Use specific repository classes for each entity type
- Avoid business logic in repository methods

### 2. Service Layer
- Implement business logic in service classes
- Use repositories within services for data operations
- Handle validation and error cases in the service layer

### 3. Error Handling
- Always handle potential database exceptions
- Provide meaningful error messages to users
- Log errors for debugging and monitoring

### 4. Performance
- Use batch operations for multiple entities
- Consider pagination for large datasets
- Implement appropriate caching strategies

### 5. Transaction Management
- Let PySpring handle session management
- Use upsert operations when appropriate
- Consider transaction boundaries for multi-step operations

## Notes
- The `CrudRepository` class is designed for simple CRUD operations on a single table. For more complex scenarios involving multiple tables, consider using the Unit of Work pattern provided by SQLModel.
- The class automatically handles session management, so users do not need to manually manage database sessions.
- For complex queries beyond basic CRUD operations, use the dynamic query methods or custom query implementations. 