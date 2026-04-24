# Graceful Shutdown

After this page, you'll know how to handle application shutdown cleanly — closing connections, flushing buffers, and finishing in-flight work.

When your application receives a shutdown signal (Ctrl+C, SIGTERM from a container orchestrator), you want to clean up properly. PySpring gives you clear hooks for this.

## Create a shutdown handler

Implement the `GracefulShutdownHandler` interface:

```python
from py_spring_core import Component
from py_spring_core.core.entities.graceful_shutdown_handler import (
    GracefulShutdownHandler,
    ShutdownType,
)


class AppShutdownHandler(GracefulShutdownHandler, Component):
    def on_shutdown(self, shutdown_type: ShutdownType) -> None:
        print(f"Shutting down: {shutdown_type}")
        # Close database connections, flush queues, etc.

    def on_timeout(self) -> None:
        print("Shutdown timed out!")
        # Force-close resources if needed

    def on_error(self, error: Exception) -> None:
        print(f"Error during shutdown: {error}")
```

## Shutdown types

The `ShutdownType` enum tells you what triggered the shutdown:

| Type | Description |
|---|---|
| `MANUAL` | Triggered programmatically |
| `SIGTERM` | Received SIGTERM (e.g., from Docker/Kubernetes) |
| `TIMEOUT` | Shutdown took too long |
| `ERROR` | An error occurred during shutdown |
| `UNKNOWN` | Unknown trigger |

## Shutdown with DI

Because the shutdown handler is a component, it has access to all injected dependencies:

```python
from py_spring_core import Component
from py_spring_core.core.entities.graceful_shutdown_handler import (
    GracefulShutdownHandler,
    ShutdownType,
)


class DatabaseService(Component):
    def close_connections(self):
        print("Database connections closed")


class AppShutdownHandler(GracefulShutdownHandler, Component):
    database_service: DatabaseService  # Injected!

    def on_shutdown(self, shutdown_type: ShutdownType) -> None:
        self.database_service.close_connections()
        print("Shutdown complete")

    def on_timeout(self) -> None:
        print("Shutdown timed out")

    def on_error(self, error: Exception) -> None:
        print(f"Shutdown error: {error}")
```

!!! tip
    Remember that components also have `pre_destroy()` hooks. Use `GracefulShutdownHandler` for application-level shutdown coordination, and `pre_destroy()` for component-level cleanup.

## Recap

Graceful shutdown gives you clean application termination.

* Implement `GracefulShutdownHandler` on a Component
* `on_shutdown()` — handle the shutdown signal
* `on_timeout()` — handle shutdown timeout
* `on_error()` — handle errors during shutdown
* `ShutdownType` tells you what triggered the shutdown
* Full dependency injection is available

Next, learn about **Skills** — how to integrate PySpring with Claude Code for accelerated development.

For more advanced topics, check out the **Advanced User Guide** for qualifier support and component lifecycle management.
