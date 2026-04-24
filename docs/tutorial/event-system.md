# Event System

After this page, you'll know how to use PySpring's publish-subscribe event system for decoupled communication between components.

Sometimes components need to react to things happening elsewhere in your application — without importing each other directly. That's what the event system is for.

## Define an event

Events are Pydantic models that extend `ApplicationEvent`:

```python
from py_spring_core.core.entities.event.application_event import ApplicationEvent


class UserCreatedEvent(ApplicationEvent):
    user_id: str
    username: str
```

Because events are Pydantic models, you get validation and serialization for free.

## Publish an event

Inject `ApplicationEventPublisher` and call `publish()`:

```python
from py_spring_core import Component
from py_spring_core.core.entities.event.application_event_publisher import (
    ApplicationEventPublisher,
)
from py_spring_core.core.entities.event.application_event import ApplicationEvent


class UserCreatedEvent(ApplicationEvent):
    user_id: str
    username: str


class UserService(Component):
    event_publisher: ApplicationEventPublisher  # Injected!

    def create_user(self, username: str):
        # ... create the user ...
        self.event_publisher.publish(
            UserCreatedEvent(user_id="123", username=username)
        )
```

## Listen for events

Use the `@EventListener` decorator on any component method:

```python
from py_spring_core import Component
from py_spring_core.core.entities.event.application_event import ApplicationEvent
from py_spring_core.core.entities.event.event_listener import EventListener


class UserCreatedEvent(ApplicationEvent):
    user_id: str
    username: str


class NotificationService(Component):
    @EventListener(UserCreatedEvent)
    def on_user_created(self, event: UserCreatedEvent):
        print(f"Welcome email sent to {event.username}")


class AuditService(Component):
    @EventListener(UserCreatedEvent)
    def on_user_created(self, event: UserCreatedEvent):
        print(f"Audit log: user {event.user_id} created")
```

Notice that `NotificationService` and `AuditService` don't know about each other — or about `UserService`. They just listen for events. That's the decoupling. 😎

!!! note
    Multiple listeners can subscribe to the same event type. They will all be called when the event is published.

## How it works

PySpring's event system is:

* **Thread-safe** — events can be published from any thread
* **Synchronous by default** — listeners are called in the publishing thread
* **Type-safe** — the `@EventListener` decorator specifies which event type to listen for

!!! info
    The event system was introduced in PySpring version 0.0.11.

## Recap

The event system gives you decoupled communication between components.

* Define events as `ApplicationEvent` subclasses (Pydantic models)
* Publish with `ApplicationEventPublisher.publish()`
* Listen with `@EventListener(EventType)` on any component method
* Thread-safe and type-safe
* Multiple listeners per event type

Next, let's learn about **Middleware** — how to process requests before they reach your controllers.
