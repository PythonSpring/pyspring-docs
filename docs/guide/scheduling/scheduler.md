PySpring Scheduler
=================

The PySpring Scheduler provides a robust and flexible way to manage scheduled tasks in your applications, seamlessly integrating with the PySpring framework.

Installation
-----------

Since the package is under active development, it can only be installed directly from the git repository:

```bash
pip install git+ssh://git@github.com/PythonSpring/pyspring-scheduler.git
```

* * * * *

Overview
--------

The scheduler is built on top of APScheduler and provides a component-based approach to scheduling tasks. It supports various trigger types, timezone-aware scheduling, and integrates with PySpring's dependency injection system.

* * * * *

Configuration
------------

The scheduler can be configured through your application's properties file:

```json
{
    "scheduler": {
        "number_of_workers": 20,
        "max_instances": 3,
        "timezone": "UTC",
        "coalesce": false
    }
}
```

### Configuration Options

- `number_of_workers`: Number of threads in the scheduler's thread pool
- `max_instances`: Maximum number of concurrent instances of a job
- `timezone`: Default timezone for all jobs
- `coalesce`: Whether to run missed jobs when the scheduler resumes

* * * * *

Setting Up the Scheduler
-----------------------

1. Add the scheduler provider to your application:

```python
from py_spring_core import PySpringApplication
from py_spring_scheduler import provide_scheduler

def main():
    app = PySpringApplication("./app-config.json", entity_providers=[provide_scheduler()])
    app.run()

if __name__ == "__main__":
    main()
```

* * * * *

Creating Scheduled Tasks
----------------------

### Using the @Scheduled Decorator

The `@Scheduled` decorator is the primary way to create scheduled tasks. It supports multiple trigger types:

#### Interval Trigger

```python
from py_spring_scheduler import Scheduled, IntervalTrigger

@Scheduled(trigger=IntervalTrigger(seconds=5))
def my_interval_task():
    # Runs every 5 seconds
    pass
```

#### Cron Trigger

```python
from py_spring_scheduler import Scheduled, CronTrigger

@Scheduled(trigger=CronTrigger(cron="0 0 * * *"))
def my_cron_task():
    # Runs daily at midnight
    pass
```

#### Combining Triggers

You can combine multiple triggers using `AndTrigger` and `OrTrigger`:

```python
from py_spring_scheduler import Scheduled, IntervalTrigger, CronTrigger, AndTrigger, OrTrigger

# Using OrTrigger (runs if either trigger condition is met)
@Scheduled(trigger=OrTrigger([
    IntervalTrigger(seconds=3),
    CronTrigger(minute=0)
]))
def my_or_task():
    # Runs every 3 seconds OR at the start of every hour
    pass

# Using AndTrigger (runs only if all trigger conditions are met)
@Scheduled(trigger=AndTrigger([
    CronTrigger(hour=9),  # Only during 9 AM
    CronTrigger(minute=0)  # Only at minute 0
]))
def my_and_task():
    # Runs only at 9:00 AM
    pass
```

* * * * *

Component-based Scheduling
------------------------

Scheduled tasks can be defined within PySpring components, allowing you to leverage dependency injection:

```python
from py_spring_core import Component
from py_spring_scheduler import Scheduled, IntervalTrigger

class MyComponent(Component):
    @Scheduled(trigger=IntervalTrigger(seconds=3))
    def scheduled_method(self):
        # Runs every 3 seconds
        pass
```

### Using Dependencies in Scheduled Tasks

Components can be injected and used within scheduled tasks:

```python
from py_spring_core import Component
from py_spring_scheduler import Scheduled, IntervalTrigger

class Service(Component):
    def do_something(self):
        print("Service method called")

class TaskComponent(Component):
    service: Service  # Dependency injection

    @Scheduled(trigger=IntervalTrigger(seconds=5))
    def scheduled_task(self):
        self.service.do_something()
```

* * * * *

Best Practices
-------------

1. **Resource Management**
    - Set appropriate `max_instances` to prevent resource exhaustion
    - Use `coalesce` carefully to avoid overwhelming the system after downtime

2. **Error Handling**
    - Implement proper error handling in scheduled tasks
    - Consider using PySpring's logging system for task monitoring

3. **Timezone Awareness**
    - Always specify timezones explicitly for cron jobs
    - Be mindful of daylight saving time transitions

4. **Component Design**
    - Keep scheduled tasks focused and single-purpose
    - Use dependency injection for better testability

* * * * *

Common Use Cases
---------------

1. **Periodic Data Processing**
   ```python
   @Scheduled(trigger=IntervalTrigger(minutes=30))
   def process_data():
       # Process data every 30 minutes
       pass
   ```

2. **Daily Maintenance Tasks**
   ```python
   @Scheduled(trigger=CronTrigger(cron="0 2 * * *"))  # 2 AM daily
   def maintenance_task():
       # Perform daily maintenance
       pass
   ```

3. **Health Checks**
   ```python
   @Scheduled(trigger=IntervalTrigger(minutes=5))
   def health_check():
       # Check system health every 5 minutes
       pass
   ```

* * * * *

Troubleshooting
--------------

### Common Issues

1. **Tasks Not Running**
   - Check if the scheduler is properly initialized
   - Verify timezone settings
   - Ensure the task is properly decorated

2. **Resource Exhaustion**
   - Monitor thread pool usage
   - Adjust `number_of_workers` based on workload
   - Review `max_instances` settings

3. **Timezone Issues**
   - Verify timezone configuration
   - Check for daylight saving time transitions
   - Use explicit timezone settings in triggers

* * * * *

Summary
-------

The PySpring Scheduler provides:

- Seamless integration with PySpring framework
- Flexible scheduling options (Interval, Cron)
- Component-based task management
- Dependency injection support
- Configurable thread pool
- Timezone-aware scheduling
- Job coalescing and instance limits

By following the patterns and best practices outlined in this guide, you can effectively implement scheduled tasks in your PySpring applications. 