# Graceful Shutdown

PySpring provides a graceful shutdown mechanism that ensures your application shuts down cleanly, completing any ongoing operations before terminating. This is particularly important for applications that need to handle database connections, background tasks, or other resources that require proper cleanup.

## Overview

The graceful shutdown system in PySpring is implemented through the `GracefulShutdownHandler` interface. It provides:

- Automatic handling of SIGINT (Ctrl+C) and SIGTERM signals
- Different shutdown type handling (MANUAL, SIGTERM, TIMEOUT, ERROR)
- Thread-safe shutdown state management
- Custom error handling capabilities

## Implementing Graceful Shutdown

To implement graceful shutdown, create a class that extends `GracefulShutdownHandler`:

```python
from py_spring_core import GracefulShutdownHandler, ShutdownType

class AppShutdownHandler(GracefulShutdownHandler):
    db_service: DatabaseService

    def on_shutdown(self, shutdown_type: ShutdownType) -> None:
        """
        Handle shutdown based on the type.
        Called when a shutdown signal is received.
        """
        if shutdown_type == ShutdownType.MANUAL:
            logger.info("Manual shutdown initiated (Ctrl+C)")
        elif shutdown_type == ShutdownType.SIGTERM:
            logger.info("SIGTERM shutdown initiated")
        
        # Close database connections
        self.db_service.close_all_connections()
        
        # Cancel background tasks
        self.cancel_background_tasks()

    def on_timeout(self) -> None:
        """
        Handle timeout situation.
        Called when shutdown takes too long.
        """
        logger.warning("Shutdown timed out, forcing cleanup")
        self.force_cleanup()

    def on_error(self, error: Exception) -> None:
        """
        Handle errors during shutdown.
        Called when an error occurs during shutdown process.
        """
        logger.error(f"Error during shutdown: {error}")
        # Implement emergency cleanup if needed
```

The framework will:
1. Automatically discover your `GracefulShutdownHandler` implementation
2. Register it as the shutdown handler
3. Call appropriate methods based on shutdown triggers:
   - `on_shutdown()` when SIGINT or SIGTERM is received
   - `on_timeout()` if shutdown process times out
   - `on_error()` if any errors occur during shutdown

## Shutdown Types

PySpring defines several shutdown types through the `ShutdownType` enum:

```python
class ShutdownType(Enum):
    MANUAL = auto()      # e.g., Ctrl+C
    SIGTERM = auto()     # e.g., docker stop, systemctl stop
    TIMEOUT = auto()     # e.g., shutdown triggered by time constraint
    ERROR = auto()       # e.g., unrecoverable fault
    UNKNOWN = auto()
```

Use these types to implement different shutdown behaviors:

```python
def on_shutdown(self, shutdown_type: ShutdownType) -> None:
    match shutdown_type:
        case ShutdownType.MANUAL:
            # Handle manual shutdown (e.g., Ctrl+C)
            self.handle_manual_shutdown()
        case ShutdownType.SIGTERM:
            # Handle SIGTERM (e.g., from container orchestrator)
            self.handle_sigterm_shutdown()
        case ShutdownType.TIMEOUT:
            # Handle timeout-triggered shutdown
            self.handle_timeout_shutdown()
        case ShutdownType.ERROR:
            # Handle error-triggered shutdown
            self.handle_error_shutdown()
        case _:
            # Handle unknown shutdown type
            self.handle_unknown_shutdown()
```

## Best Practices

1. **Implement Proper Error Handling**:
   ```python
   def on_error(self, error: Exception) -> None:
       logger.error(f"Shutdown error: {error}")
       try:
           # Attempt emergency cleanup
           self.emergency_cleanup()
       except Exception as e:
           logger.critical(f"Emergency cleanup failed: {e}")
   ```

2. **Handle Timeouts Gracefully**:
   ```python
   def on_timeout(self) -> None:
       logger.warning("Shutdown timeout occurred")
       # Force close any remaining connections
       self.force_close_connections()
       # Kill any hanging tasks
       self.terminate_tasks()
   ```

## Common Use Cases

### Database Cleanup
```python
class DatabaseShutdownHandler(GracefulShutdownHandler):
    db_pool: ConnectionPool

    def on_shutdown(self, shutdown_type: ShutdownType) -> None:
        logger.info(f"Database cleanup initiated: {shutdown_type}")
        self.db_pool.stop_accepting_new_connections()
        self.db_pool.wait_for_active_transactions(timeout=10)
        self.db_pool.close_all()

    def on_timeout(self) -> None:
        logger.warning("Database cleanup timed out")
        self.db_pool.force_close_all()

    def on_error(self, error: Exception) -> None:
        logger.error(f"Database cleanup error: {error}")
        self.db_pool.emergency_shutdown()
```

### Task Manager Cleanup
```python
class TaskShutdownHandler(GracefulShutdownHandler):
    task_manager: TaskManager

    def on_shutdown(self, shutdown_type: ShutdownType) -> None:
        logger.info(f"Task cleanup initiated: {shutdown_type}")
        self.task_manager.stop_accepting_new_tasks()
        self.task_manager.wait_for_running_tasks(timeout=30)

    def on_timeout(self) -> None:
        logger.warning("Task cleanup timed out")
        self.task_manager.force_stop_all_tasks()

    def on_error(self, error: Exception) -> None:
        logger.error(f"Task cleanup error: {error}")
        self.task_manager.kill_all_tasks()
```