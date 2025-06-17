# Event System

The Event System in PySpring provides a powerful way to implement event-driven architecture in your applications. It allows components to communicate in a decoupled manner through events, following the publish-subscribe pattern.

## Overview

The Event System consists of three main components:

1. `ApplicationEventPublisher`: The core component for publishing events
2. `ApplicationEventHandlerRegistry`: Manages event handler registration and execution
3. `ApplicationContextRequired`: Interface for components needing application context access

## Key Features

- Thread-safe event publishing mechanism
- Decorator-based event listener registration
- Asynchronous event processing
- Type-safe events using Pydantic models
- Automatic event handler initialization
- Integration with the component system

## Usage

### Creating Events

Events are defined as Pydantic models that inherit from `ApplicationEvent`:

```python
from py_spring_core import ApplicationEvent

class MyEvent(ApplicationEvent):
    message: str
```

### Publishing Events

To publish events, inject the `ApplicationEventPublisher` into your component:

```python
from py_spring_core import ApplicationEventPublisher, Component

class MyComponent(Component):
    event_publisher: ApplicationEventPublisher
    
    def do_something(self):
        self.event_publisher.publish(MyEvent(message="Hello World!"))
```

### Subscribing to Events

Use the `@EventListener` decorator to subscribe to events:

```python
from py_spring_core import Component, EventListener

@Component
class MyListener:
    @EventListener(MyEvent)
    def handle_event(self, event: MyEvent):
        print(f"Received event: {event.message}")
```

## Technical Details

### Event Processing

- Events are processed asynchronously in a dedicated thread
- The system uses a thread-safe queue for event distribution
- Event handlers are automatically registered during application startup
- Events are validated using Pydantic models

### Thread Safety

The event system is designed to be thread-safe:
- Event publishing is thread-safe
- Event handling occurs in a separate thread
- The event queue ensures ordered processing

## Best Practices

1. **Event Design**
   - Keep events focused and specific
   - Use meaningful event names
   - Include only necessary data in events

2. **Event Handling**
   - Keep event handlers lightweight
   - Avoid long-running operations in event handlers
   - Use proper error handling in event handlers

3. **Performance**
   - Be mindful of event frequency
   - Consider using event filtering when appropriate
   - Monitor event queue size in production

## Example

Here's a complete example showing event publishing and handling:

```python
from py_spring_core import (
    ApplicationEvent,
    ApplicationEventPublisher,
    EventListener,
    Component
)

# Define an event
class UserCreatedEvent(ApplicationEvent):
    user_id: str
    username: str

# Publisher component
class UserService(Component):
    event_publisher: ApplicationEventPublisher
    
    def create_user(self, username: str):
        user_id = "generated_id"  # In real app, generate proper ID
        self.event_publisher.publish(UserCreatedEvent(
            user_id=user_id,
            username=username
        ))
        return user_id

# Listener component
@Component
class UserEventListener:
    @EventListener(UserCreatedEvent)
    def handle_user_created(self, event: UserCreatedEvent):
        print(f"New user created: {event.username} (ID: {event.user_id})")
```

## Version Information

This feature is available in PySpring version 0.0.11 and later.

## Dependencies

The Event System is built into PySpring core and doesn't require additional dependencies. 