# Modifiable Queries with Commit Control

## Overview

PySpringModel provides fine-grained control over database transaction commits through the `is_modifying` parameter in the `@Query` decorator. This feature allows developers to specify whether a query operation should automatically commit changes to the database, enabling:

- **Performance Optimization**: Avoid unnecessary commits for read-only operations
- **Transaction Control**: Explicit control over when database changes are persisted
- **Batch Processing**: Efficient handling of multiple operations within transactions
- **Error Recovery**: Better rollback control for complex operations

## Basic Usage

### Read-Only Operations (Default)

By default, all queries have `is_modifying=False`, which means no automatic commit:

```python
from py_spring_model import CrudRepository, Query
from typing import Optional, List

class UserRepository(CrudRepository[int, User]):
    # Read operations - no commit needed (default behavior)
    @Query("SELECT * FROM user WHERE email = {email}")
    def find_by_email(self, email: str) -> Optional[User]: ...
    
    @Query("SELECT * FROM user WHERE age > {min_age}")
    def find_users_older_than(self, min_age: int) -> List[User]: ...
    
    @Query("SELECT COUNT(*) as count FROM user WHERE status = {status}")
    def count_users_by_status(self, status: str) -> int: ...
```

### Modifying Operations

For operations that modify data, set `is_modifying=True` to enable automatic commits:

```python
class UserRepository(CrudRepository[int, User]):
    # INSERT with automatic commit
    @Query("INSERT INTO user (name, email, age) VALUES ({name}, {email}, {age}) RETURNING *", is_modifying=True)
    def create_user(self, name: str, email: str, age: int) -> User: ...
    
    # UPDATE with automatic commit
    @Query("UPDATE user SET name = {name}, age = {age} WHERE email = {email} RETURNING *", is_modifying=True)
    def update_user(self, name: str, email: str, age: int) -> User: ...
    
    # DELETE with automatic commit
    @Query("DELETE FROM user WHERE id = {user_id}", is_modifying=True)
    def delete_user(self, user_id: int) -> None: ...
```

## Advanced Usage Patterns

### Batch Operations with Custom Transaction Control

For complex batch operations, you can use `is_modifying=False` and manage transactions manually:

```python
class UserRepository(CrudRepository[int, User]):
    @Query("INSERT INTO user (name, email, age) VALUES ({name}, {email}, {age})", is_modifying=False)
    def insert_user_no_commit(self, name: str, email: str, age: int) -> None: ...
    
    @Query("UPDATE user SET status = {status} WHERE id = {user_id}", is_modifying=False)
    def update_user_status_no_commit(self, user_id: int, status: str) -> None: ...
    
    def create_users_batch(self, users: List[dict]) -> bool:
        """Create multiple users in a single transaction"""
        try:
            with PySpringModel.create_managed_session(should_commit=True) as session:
                for user_data in users:
                    self.insert_user_no_commit(**user_data)
                return True
        except Exception as e:
            logger.error(f"Batch user creation failed: {e}")
            return False
    
    def bulk_update_user_statuses(self, user_updates: List[dict]) -> bool:
        """Update multiple user statuses in a single transaction"""
        try:
            with PySpringModel.create_managed_session(should_commit=True) as session:
                for update in user_updates:
                    self.update_user_status_no_commit(update['user_id'], update['status'])
                return True
        except Exception as e:
            logger.error(f"Bulk status update failed: {e}")
            return False
```

### Conditional Operations

Combine read and write operations with conditional logic:

```python
class UserRepository(CrudRepository[int, User]):
    @Query("SELECT * FROM user WHERE email = {email}")
    def find_by_email(self, email: str) -> Optional[User]: ...
    
    @Query("INSERT INTO user (name, email, age) VALUES ({name}, {email}, {age}) RETURNING *", is_modifying=True)
    def create_user(self, name: str, email: str, age: int) -> User: ...
    
    @Query("UPDATE user SET last_login = NOW() WHERE email = {email}", is_modifying=True)
    def update_last_login(self, email: str) -> None: ...
    
    def create_or_update_login(self, email: str, name: str, age: int) -> User:
        """Create user if not exists, otherwise update last login"""
        existing_user = self.find_by_email(email)
        if existing_user:
            self.update_last_login(email)
            return existing_user
        else:
            return self.create_user(name, email, age)
```

### Error Handling with Rollback

Handle errors gracefully with proper rollback behavior:

```python
class UserRepository(CrudRepository[int, User]):
    @Query("UPDATE user SET email = {new_email} WHERE id = {user_id}", is_modifying=False)
    def update_email_no_commit(self, user_id: int, new_email: str) -> None: ...
    
    @Query("INSERT INTO user_audit (user_id, action, timestamp) VALUES ({user_id}, {action}, NOW())", is_modifying=False)
    def log_audit_no_commit(self, user_id: int, action: str) -> None: ...
    
    def update_email_with_audit(self, user_id: int, new_email: str) -> bool:
        """Update email and log audit entry in a single transaction"""
        try:
            with PySpringModel.create_managed_session(should_commit=True) as session:
                # Both operations succeed or both fail
                self.update_email_no_commit(user_id, new_email)
                self.log_audit_no_commit(user_id, f"email_updated_to_{new_email}")
                return True
        except Exception as e:
            logger.error(f"Email update with audit failed: {e}")
            return False
```

## Session Management Deep Dive

### How is_modifying Works

The `is_modifying` parameter controls the `should_commit` parameter passed to `PySpringModel.create_managed_session()`:

```python
# When is_modifying=True
PySpringModel.create_managed_session(should_commit=True)

# When is_modifying=False (default)
PySpringModel.create_managed_session(should_commit=False)
```

### Session Lifecycle

1. **is_modifying=True**: Session commits automatically on successful completion
2. **is_modifying=False**: Session does not commit, changes are rolled back when session ends
3. **Manual Control**: Use `PySpringModel.create_managed_session(should_commit=True/False)` directly

### Performance Implications

```python
class UserRepository(CrudRepository[int, User]):
    # Efficient for read-heavy operations
    @Query("SELECT * FROM user WHERE status = {status}")
    def find_by_status(self, status: str) -> List[User]: ...
    
    # Efficient for single write operations
    @Query("INSERT INTO user (name, email) VALUES ({name}, {email})", is_modifying=True)
    def create_user(self, name: str, email: str) -> User: ...
    
    # Efficient for batch operations (manual transaction control)
    def create_multiple_users(self, users: List[dict]) -> bool:
        """More efficient than multiple individual commits"""
        with PySpringModel.create_managed_session(should_commit=True) as session:
            for user_data in users:
                # Use is_modifying=False for batch items
                self._insert_user_batch_item(**user_data)
```

## Best Practices

### 1. Use Appropriate is_modifying Settings

```python
# ✅ Good - Read operations
@Query("SELECT * FROM user WHERE email = {email}")
def find_by_email(self, email: str) -> Optional[User]: ...

# ✅ Good - Single write operations
@Query("INSERT INTO user (name, email) VALUES ({name}, {email})", is_modifying=True)
def create_user(self, name: str, email: str) -> User: ...

# ❌ Avoid - Write operations without explicit is_modifying
@Query("DELETE FROM user WHERE id = {user_id}")  # Should specify is_modifying=True
def delete_user(self, user_id: int) -> None: ...
```

### 2. Batch Operations Strategy

```python
# ✅ Good - Manual transaction control for batches
def create_users_efficiently(self, users: List[dict]) -> bool:
    with PySpringModel.create_managed_session(should_commit=True) as session:
        for user in users:
            self._insert_single_no_commit(**user)  # is_modifying=False

# ❌ Avoid - Individual commits for batch operations
def create_users_inefficiently(self, users: List[dict]) -> bool:
    for user in users:
        self.create_user(**user)  # is_modifying=True - commits each time
```

### 3. Error Handling

```python
def safe_update_with_validation(self, user_id: int, new_email: str) -> bool:
    """Update with validation and proper error handling"""
    try:
        # Validate first (read-only)
        existing_user = self.find_by_id(user_id)
        if not existing_user:
            return False
            
        # Check for email conflicts (read-only)
        email_exists = self.find_by_email(new_email)
        if email_exists and email_exists.id != user_id:
            return False
            
        # Perform update (with commit)
        self.update_user_email(user_id, new_email)  # is_modifying=True
        return True
    except Exception as e:
        logger.error(f"Safe update failed: {e}")
        return False
```

### 4. Complex Transactions

```python
def transfer_user_data(self, from_user_id: int, to_user_id: int) -> bool:
    """Complex operation requiring multiple database changes"""
    try:
        with PySpringModel.create_managed_session(should_commit=True) as session:
            # All operations use is_modifying=False for manual control
            user_data = self._get_user_data_no_commit(from_user_id)
            self._transfer_user_posts_no_commit(from_user_id, to_user_id)
            self._merge_user_preferences_no_commit(from_user_id, to_user_id)
            self._deactivate_user_no_commit(from_user_id)
            self._log_transfer_audit_no_commit(from_user_id, to_user_id)
            # All succeed together or all fail together
            return True
    except Exception as e:
        logger.error(f"User data transfer failed: {e}")
        return False
```

## Migration Guide

### From Read-Only to Modifiable Queries

If you have existing read-only queries that you want to make modifiable:

```python
# Before (read-only limitation)
# This would not have persisted changes

# After (with modifiable support)
class UserRepository(CrudRepository[int, User]):
    # Keep existing read operations unchanged
    @Query("SELECT * FROM user WHERE email = {email}")
    def find_by_email(self, email: str) -> Optional[User]: ...
    
    # Add new modifying operations
    @Query("INSERT INTO user (name, email, age) VALUES ({name}, {email}, {age}) RETURNING *", is_modifying=True)
    def create_user(self, name: str, email: str, age: int) -> User: ...
    
    @Query("UPDATE user SET name = {name} WHERE id = {user_id} RETURNING *", is_modifying=True)
    def update_user_name(self, user_id: int, name: str) -> User: ...
```

### Gradual Adoption

1. **Start Small**: Add `is_modifying=True` to simple INSERT/UPDATE/DELETE operations
2. **Test Thoroughly**: Verify that commits happen as expected
3. **Optimize Batches**: Convert high-volume operations to use manual transaction control
4. **Monitor Performance**: Ensure the new transaction behavior meets performance requirements

## Troubleshooting

### Common Issues

1. **Changes Not Persisting**: Make sure `is_modifying=True` for write operations
2. **Performance Issues**: Use manual transaction control for batch operations
3. **Partial Failures**: Wrap related operations in a single session context
4. **Rollback Behavior**: Understand that `is_modifying=False` will rollback changes

### Debugging Tips

```python
# Add logging to understand transaction behavior
import logging

class UserRepository(CrudRepository[int, User]):
    @Query("INSERT INTO user (name, email) VALUES ({name}, {email})", is_modifying=True)
    def create_user(self, name: str, email: str) -> User: ...
    
    def create_user_with_logging(self, name: str, email: str) -> User:
        logging.info(f"Creating user: {name} ({email})")
        try:
            user = self.create_user(name, email)
            logging.info(f"User created successfully with ID: {user.id}")
            return user
        except Exception as e:
            logging.error(f"User creation failed: {e}")
            raise
```

## Summary

The `is_modifying` parameter provides powerful control over database transactions in PySpringModel:

- **Default**: `is_modifying=False` for read-only operations
- **Modifying**: `is_modifying=True` for operations that should commit changes
- **Manual Control**: Use `PySpringModel.create_managed_session()` for complex transactions
- **Performance**: Choose the right strategy based on your use case
- **Safety**: Proper error handling ensures data consistency 