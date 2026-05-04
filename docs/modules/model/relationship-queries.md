# Relationship Queries

PySpring Model supports querying across **SQLModel relationships** directly from method names. Instead of writing manual joins, declare a method that references a relationship field and PySpring generates the join automatically.

## How it works

When PySpring parses a dynamic query method name, it checks each field token against the model's `Relationship` fields. If a token starts with a relationship name followed by a field on the related model, PySpring:

1. Resolves the related model class via SQLAlchemy's mapper inspection
2. Generates a `JOIN` to the related table
3. Filters on the related model's column
4. Applies `DISTINCT` to the results to avoid duplicates from the join

## Setup

Define your models with SQLModel `Relationship` fields:

```python
from typing import Optional, List
from sqlmodel import Relationship
from py_spring_model import PySpringModel, Field

class Team(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    name: str = Field()
    members: List["User"] = Relationship(back_populates="team")

class User(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    name: str = Field()
    status: str = Field()
    team_id: Optional[int] = Field(default=None, foreign_key="team.id")
    team: Optional[Team] = Relationship(back_populates="members")
    posts: List["Post"] = Relationship(back_populates="author")

class Post(PySpringModel, table=True):
    id: int = Field(default=None, primary_key=True)
    title: str = Field()
    published: bool = Field(default=False)
    author_id: Optional[int] = Field(default=None, foreign_key="user.id")
    author: Optional[User] = Relationship(back_populates="posts")
```

## Basic usage

Reference the relationship name followed by the target field, separated by `_`:

```python
class TeamRepository(CrudRepository[int, Team]):
    # Find teams that have a member with a specific status
    def find_all_by_members_status(self, status: str) -> List[Team]: ...
    # → SELECT DISTINCT team.* FROM team JOIN user ON ... WHERE user.status = :status
```

```python
class UserRepository(CrudRepository[int, User]):
    # Find users who have a post with a specific title
    def find_all_by_posts_title(self, title: str) -> List[User]: ...
    # → SELECT DISTINCT user.* FROM user JOIN post ON ... WHERE post.title = :title
```

Usage:

```python
active_teams = team_repo.find_all_by_members_status(status="active")
authors = user_repo.find_all_by_posts_title(title="Hello World")
```

## Naming convention

The method name token format is `{relationship_name}_{related_field}`:

| Token | Relationship | Related field |
|-------|-------------|---------------|
| `members_status` | `members` | `status` |
| `posts_title` | `posts` | `title` |
| `team_name` | `team` | `name` |

!!! note
    If a token matches both a direct column and a relationship traversal, the **direct column takes precedence** for backwards compatibility. For example, if `User` has both a `team_name` column and a `team` relationship, `find_by_team_name` will query the direct `team_name` column.

## Combining with field operations

Relationship queries support all [field operations](field-operations.md):

```python
class TeamRepository(CrudRepository[int, Team]):
    # Members with status in a list
    def find_all_by_members_status_in(
        self, status: List[str]
    ) -> List[Team]: ...

    # Members with name matching a pattern
    def find_all_by_members_name_like(
        self, name: str
    ) -> List[Team]: ...

    # Members with name starting with a prefix
    def find_all_by_members_name_starts_with(
        self, name: str
    ) -> List[Team]: ...
```

## Combining with AND / OR

Relationship fields can be combined with direct fields and other relationship fields:

```python
class TeamRepository(CrudRepository[int, Team]):
    # Team name AND member status
    def find_all_by_name_and_members_status(
        self, name: str, status: str
    ) -> List[Team]: ...

class UserRepository(CrudRepository[int, User]):
    # User status AND post published flag
    def find_all_by_status_and_posts_published(
        self, status: str, published: bool
    ) -> List[User]: ...
```

## With count, exists, and delete

Relationship queries work with all query type prefixes:

```python
class TeamRepository(CrudRepository[int, Team]):
    # Count teams with active members
    def count_by_members_status(self, status: str) -> int: ...

    # Check if any team has a member with this name
    def exists_by_members_name(self, name: str) -> bool: ...

    # Delete teams whose members all have a certain status
    def delete_all_by_members_status(self, status: str) -> int: ...
```

For `count_by_` and `delete_` prefixes with joins, PySpring uses a distinct subquery on the primary key to avoid counting or deleting duplicates.

## Resolution rules

PySpring resolves relationship tokens using these rules:

1. **Longest match first** — relationship names are sorted by length (descending), so `team_members` is checked before `team`
2. **Direct columns take precedence** — if the full token is a column on the model, it is treated as a direct column query
3. **Remainder must be non-empty** — the token must have characters remaining after stripping the relationship prefix to form the target field name

## Limitations

- Only single-level relationship traversal is supported (no chained joins like `team_members_posts_title`)
- The related model must be resolvable at class initialization time (forward references must be resolved)
- Composite foreign keys are not supported for `delete_` operations with joins
