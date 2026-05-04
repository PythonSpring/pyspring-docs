# Getting Started

This page walks you through installing PySpring Model, configuring a database connection, defining your first entity, and running basic operations.

## Installation

```console
$ pip install git+https://github.com/PythonSpring/pyspring-model.git
```

## Configuration

Add the `py_spring_model` key to your `application-properties.json`:

```json
{
    "py_spring_model": {
        "sqlalchemy_database_uri": "sqlite:///./app.db",
        "create_all_tables": true
    }
}
```

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `sqlalchemy_database_uri` | `str` | *(required)* | Any SQLAlchemy-compatible connection URI |
| `create_all_tables` | `bool` | `true` | Automatically create all tables on startup |

Supported databases include SQLite, PostgreSQL, MySQL, and any database supported by SQLAlchemy.

## Register the starter

In your application entry point, register `PySpringModelStarter`:

```python
from py_spring_core import PySpringApplication
from py_spring_model import PySpringModelStarter

def main():
    app = PySpringApplication(
        "./app-config.json",
        starters=[PySpringModelStarter()],
    )
    app.run()

if __name__ == "__main__":
    main()
```

`PySpringModelStarter` handles engine creation, table creation, metadata registration, and auto-implementation of dynamic query methods.

## Define a model

Subclass `PySpringModel` and set `table=True`:

```python
from py_spring_model import PySpringModel, Field

class User(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    name: str = Field()
    email: str = Field()
    age: int = Field()
    status: str = Field()
```

`PySpringModel` extends `SQLModel`, so you get full Pydantic validation and SQLAlchemy column mapping from type hints.

## Define a repository

Subclass `CrudRepository[ID, Model]` to get built-in CRUD methods:

```python
from py_spring_model import CrudRepository
from typing import Optional, List

class UserRepository(CrudRepository[int, User]):
    # Dynamic method — auto-implemented from the name
    def find_by_name(self, name: str) -> Optional[User]: ...
    def find_all_by_status(self, status: str) -> List[User]: ...
```

The type parameters `CrudRepository[int, User]` specify the primary key type and the model class.

## Use it in a service

Inject the repository into any component:

```python
from py_spring_core import Component
from typing import Optional, List

class UserService(Component):
    user_repository: UserRepository  # Auto-injected

    def get_user(self, name: str) -> Optional[User]:
        return self.user_repository.find_by_name(name)

    def get_active_users(self) -> List[User]:
        return self.user_repository.find_all_by_status("active")
```

## Complete example

Putting it all together:

```python
# models.py
from py_spring_model import PySpringModel, Field

class User(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    name: str = Field()
    email: str = Field()

# repositories.py
from py_spring_model import CrudRepository
from typing import Optional

class UserRepository(CrudRepository[int, User]):
    def find_by_email(self, email: str) -> Optional[User]: ...

# services.py
from py_spring_core import Component

class UserService(Component):
    user_repository: UserRepository

    def create_user(self, name: str, email: str) -> User:
        user = User(name=name, email=email)
        return self.user_repository.save(user)

# main.py
from py_spring_core import PySpringApplication
from py_spring_model import PySpringModelStarter

def main():
    app = PySpringApplication(
        "./app-config.json",
        starters=[PySpringModelStarter()],
    )
    app.run()

if __name__ == "__main__":
    main()
```

## Next steps

Now that you have a working setup:

- Learn about all **[built-in CRUD operations](crud-repository.md)**
- Explore **[dynamic query methods](dynamic-queries.md)** for complex lookups
- Use **[field operations](field-operations.md)** for comparisons and pattern matching
- Write raw SQL with **[custom queries](custom-queries.md)**
- Manage transactions with **[@Transactional](transactions.md)**
