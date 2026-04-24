# Qualifier Support

After this page, you'll know how to handle multiple implementations of the same base type using qualifiers.

## The problem

Sometimes you have an abstract base class with multiple implementations:

```python
from py_spring_core import Component


class AbstractNotifier(Component):
    def notify(self, message: str) -> None:
        raise NotImplementedError


class EmailNotifier(AbstractNotifier):
    def notify(self, message: str) -> None:
        print(f"Email: {message}")


class SlackNotifier(AbstractNotifier):
    def notify(self, message: str) -> None:
        print(f"Slack: {message}")
```

If you try to inject `AbstractNotifier`, PySpring doesn't know which implementation you want. That's where qualifiers come in.

## Use `Annotated` with a qualifier

```python
from typing import Annotated
from py_spring_core import Component


class AbstractNotifier(Component):
    def notify(self, message: str) -> None:
        raise NotImplementedError


class EmailNotifier(AbstractNotifier):
    def notify(self, message: str) -> None:
        print(f"Email: {message}")


class SlackNotifier(AbstractNotifier):
    def notify(self, message: str) -> None:
        print(f"Slack: {message}")


class AlertService(Component):
    email: Annotated[AbstractNotifier, "EmailNotifier"]
    slack: Annotated[AbstractNotifier, "SlackNotifier"]

    def post_construct(self):
        self.email.notify("Server is down!")
        self.slack.notify("Server is down!")
```

The second argument in `Annotated` is the **qualifier** — the class name of the specific implementation you want.

!!! tip
    The qualifier string must match the class name of the implementation. PySpring uses it to disambiguate between multiple registered components of the same base type.

## When to use qualifiers

Use qualifiers when:

* You have multiple implementations of an abstract class
* You need to inject a specific one by name
* You want to use different implementations in different contexts

!!! note
    If you only have one implementation, you don't need a qualifier. PySpring will resolve it automatically.

## Recap

Qualifiers let you pick a specific implementation when multiple exist.

* Use `Annotated[BaseType, "ImplementationName"]` as the type hint
* The qualifier string matches the implementation class name
* Only needed when multiple implementations of the same base type exist
