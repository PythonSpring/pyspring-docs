# Dependency Injection

After this page, you'll understand how PySpring automatically resolves and injects dependencies — and why it matters.

## What is dependency injection?

Dependency injection (DI) is a pattern where an object receives its dependencies from an external source, rather than creating them itself.

Without DI, your code looks like this:

```python
class UserService:
    def __init__(self):
        self.repository = UserRepository()  # Tight coupling!
```

The problem: `UserService` is hardwired to `UserRepository`. You can't swap it for testing, you can't reuse it with a different repository, and changing `UserRepository`'s constructor breaks `UserService`.

With PySpring's DI:

```python
from py_spring_core import Component


class UserRepository(Component):
    def find_all(self):
        return []


class UserService(Component):
    user_repository: UserRepository  # Injected by PySpring

    def get_all_users(self):
        return self.user_repository.find_all()
```

PySpring sees the type hint `user_repository: UserRepository`, finds the matching component, and injects it. You never call a constructor manually.

## How it works

During startup, PySpring:

1. **Scans** your project for all `Component`, `Properties`, `RestController`, and `BeanCollection` classes
2. **Resolves** the dependency graph — figuring out which components depend on which
3. **Instantiates** them in the right order
4. **Injects** each dependency by matching type hints to registered instances
5. **Calls** `post_construct()` on each component after all dependencies are injected

!!! note
    PySpring resolves dependencies by **type**, not by name. If you have a field `my_service: UserService`, PySpring looks for a registered component of type `UserService`.

## Declaring dependencies

You declare dependencies as class-level type annotations:

```python
from py_spring_core import Component


class EmailService(Component):
    def send(self, to: str, message: str):
        print(f"Sending to {to}: {message}")


class NotificationService(Component):
    email_service: EmailService  # Injected!

    def notify(self, user_email: str):
        self.email_service.send(user_email, "You have a notification!")
```

You can inject:

* **Components** — any class extending `Component`
* **Properties** — configuration classes extending `Properties`
* **Beans** — objects created by `BeanCollection` methods

## Accessing injected dependencies

Use `post_construct()` to work with injected dependencies — that's when they're guaranteed to be available:

```python
from py_spring_core import Component


class AppStartupService(Component):
    email_service: EmailService
    notification_service: NotificationService

    def post_construct(self):
        print("All dependencies are ready!")
        print(f"Email service: {self.email_service}")
        print(f"Notification service: {self.notification_service}")
```

!!! warning
    Don't access injected dependencies in `__init__()`. At that point, injection hasn't happened yet. Always use `post_construct()`.

## Recap

PySpring's dependency injection system:

* Resolves dependencies automatically from **type hints**
* Supports Components, Properties, and Beans
* Handles the full dependency graph — including transitive dependencies
* Requires no decorators or manual registration
* Makes your code loosely coupled and testable

Next, let's learn about **REST Controllers** — how to build APIs with PySpring.
