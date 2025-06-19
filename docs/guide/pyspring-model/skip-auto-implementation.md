# SkipAutoImplementation Decorator

The `@SkipAutoImplmentation` decorator allows you to exclude specific methods from automatic implementation by the `CrudRepositoryImplementationService`. This is useful when you want to provide your own custom implementation for methods that would otherwise be automatically generated based on their naming convention.

## Usage

```python
from py_spring_model import CrudRepository, SkipAutoImplmentation
from typing import Optional, List

class UserRepository(CrudRepository[int, User]):
    # This method will be automatically implemented
    def find_by_name(self, name: str) -> Optional[User]: ...
    
    # This method will be skipped and you must implement it yourself
    @SkipAutoImplmentation
    def find_by_email(self, email: str) -> Optional[User]:
        # Your custom implementation here
        with PySpringModel.create_session() as session:
            return session.exec(
                select(User).where(User.email == email)
            ).first()
    
    # This method will also be automatically implemented
    def find_all_by_status(self, status: str) -> List[User]: ...
```

## When to Use

Use the `@SkipAutoImplmentation` decorator when you need:

- **Custom business logic**: When the standard query logic doesn't meet your requirements
- **Complex queries**: When you need joins, subqueries, or other complex SQL operations
- **Performance optimization**: When you need to optimize specific queries
- **Custom validation**: When you need to add custom validation logic before or after the query
- **Integration with external services**: When the method needs to call external APIs or services

## Example with Custom Logic

```python
class UserRepository(CrudRepository[int, User]):
    @SkipAutoImplmentation
    def find_by_email(self, email: str) -> Optional[User]:
        # Add custom validation
        if not email or '@' not in email:
            raise ValueError("Invalid email format")
        
        # Custom implementation with additional logic
        with PySpringModel.create_session() as session:
            user = session.exec(
                select(User).where(User.email == email.lower())
            ).first()
            
            # Add custom post-processing
            if user and user.status == "inactive":
                logger.warning(f"Found inactive user with email: {email}")
            
            return user
    
    @SkipAutoImplmentation
    def find_all_by_status(self, status: str) -> List[User]:
        # Custom implementation with joins
        with PySpringModel.create_session() as session:
            return session.exec(
                select(User)
                .join(UserProfile)  # Assuming there's a UserProfile table
                .where(User.status == status)
                .order_by(User.created_at.desc())
            ).fetchall()
```

## Important Notes

- The decorator must be applied to methods that follow the naming convention (`find_by_*`, `get_by_*`, etc.)
- You must provide your own implementation for decorated methods
- The decorator preserves the original method signature and type annotations
- Decorated methods are completely excluded from the automatic implementation process 