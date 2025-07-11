# PySpringModel

PySpringModel is a Python module built on top of PySpring that provides a simple and efficient way to interact with SQL databases. It leverages the power of SQLAlchemy and SQLModel to provide a streamlined interface for CRUD operations, and integrates seamlessly with the PySpring framework for Dependency Injection and RESTful API development.

## Features

- **SQLModel Integration**: PySpringModel uses SQLModel as its core ORM, providing a simple and Pythonic way to define your data models and interact with your database.
- **Automatic CRUD Repository**: PySpringModel automatically generates a CRUD repository for each of your SQLModel entities, providing common database operations such as Create, Read, Update, and Delete.
- **Managed Sessions**: PySpringModel provides a context manager for database sessions, automatically handling session commit and rollback to ensure data consistency.
- **Dynamic Query Generation**: PySpringModel can dynamically generate and execute SQL queries based on method names in your repositories.
- **Custom SQL Queries**: PySpringModel supports custom SQL queries using the `@Query` decorator for complex database operations.
- **RESTful API Integration**: PySpringModel integrates with the PySpring framework to automatically generate basic table CRUD APIs for your SQLModel entities.

## Transaction Management

PySpringModel provides fine-grained control over database transactions through the `is_modifying` parameter in custom queries. This allows you to specify whether a query operation should automatically commit changes to the database:

- **Read Operations**: Default behavior with `is_modifying=False` (no automatic commit)
- **Modifying Operations**: Set `is_modifying=True` for INSERT, UPDATE, DELETE operations that should commit
- **Session Control**: Explicit control over when database changes are persisted

## Installation

**Note**: PySpringModel is currently under active development and is not yet available on PyPI. Please install it directly from the GitHub repository:

```bash
pip install git+https://github.com/PythonSpring/pyspring-model.git
```

## Quick Start

### 1. Define Your Data Models

Define your data models by subclassing `PySpringModel`:

```python
from py_spring_model import PySpringModel
from sqlmodel import Field

class User(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    name: str = Field()
    email: str = Field()
    age: int = Field()
    status: str = Field()
```

### 2. Create a Repository

Define a repository for your model by subclassing `CrudRepository`:

```python
from py_spring_model import CrudRepository, Query
from typing import Optional, List

class UserRepository(CrudRepository[int, User]):
    # Dynamic method-based queries (auto-implemented)
    def find_by_name(self, name: str) -> Optional[User]: ...
    def find_by_email(self, email: str) -> Optional[User]: ...
    def find_by_name_and_email(self, name: str, email: str) -> Optional[User]: ...
    def find_by_name_or_email(self, name: str, email: str) -> Optional[User]: ...
    def find_all_by_status(self, status: str) -> List[User]: ...
    def find_all_by_age_and_status(self, age: int, status: str) -> List[User]: ...
    
    # Custom SQL queries using @Query decorator
    @Query("SELECT * FROM user WHERE age > {min_age}")
    def find_users_older_than(self, min_age: int) -> List[User]: ...
    
    @Query("SELECT * FROM user WHERE email LIKE '%{domain}%'")
    def find_users_by_email_domain(self, domain: str) -> List[User]: ...
    
    @Query("SELECT * FROM user WHERE age BETWEEN {min_age} AND {max_age}")
    def find_users_by_age_range(self, min_age: int, max_age: int) -> List[User]: ...
    
    # Modifying operations with commit control
    @Query("INSERT INTO user (name, email, age) VALUES ({name}, {email}, {age}) RETURNING *", is_modifying=True)
    def create_user(self, name: str, email: str, age: int) -> User: ...
    
    @Query("UPDATE user SET name = {name}, age = {age} WHERE email = {email} RETURNING *", is_modifying=True)
    def update_user(self, name: str, email: str, age: int) -> User: ...
    
    @Query("DELETE FROM user WHERE id = {user_id}", is_modifying=True)
    def delete_user(self, user_id: int) -> None: ...
```

### 3. Use in Your Application

Use your repository in your service or controller:

```python
class UserService:
    user_repository: UserRepository
    
    def get_user_by_name(self, name: str) -> Optional[User]:
        return self.user_repository.find_by_name(name)
    
    def get_active_users_older_than(self, min_age: int) -> List[User]:
        return self.user_repository.find_users_older_than(min_age)
```

### 4. Run Your Application

Run your application with `PySpringApplication`:

```python
from py_spring_core import PySpringApplication
from py_spring_model.py_spring_model_provider import provide_py_spring_model

PySpringApplication(
    "./app-config.json",
    entity_providers=[provide_py_spring_model()]
).run()
```

## What's Next?

- Learn about [Dynamic Query Generation](dynamic-queries.md)
- Explore [Custom SQL Queries](custom-queries.md)
- Master [Modifiable Queries with Commit Control](modifiable-queries.md)
- Understand [Built-in CRUD Operations](crud-operations.md)
- See [Complete Examples](examples.md)
- Dive into the [SQL Statement Generation Algorithm](sql-statement-algorithm.md) 