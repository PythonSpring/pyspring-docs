# Getting Started

This page walks you through installing the scheduler, registering it with your application, and creating your first scheduled task.

## Installation

```console
$ pip install git+https://github.com/PythonSpring/pyspring-scheduler.git
```

## Register the starter

Register `PySpringSchedulerStarter` with your application:

```python
from py_spring_core import PySpringApplication
from py_spring_scheduler import PySpringSchedulerStarter

def main():
    app = PySpringApplication(
        "./app-config.json",
        starters=[PySpringSchedulerStarter()],
    )
    app.run()

if __name__ == "__main__":
    main()
```

`PySpringSchedulerStarter`:

1. Registers `SchedulerProperties` for configuration
2. Creates a `BackgroundScheduler` with a thread pool
3. Binds all `@Scheduled` jobs to their component instances
4. Starts the scheduler after the IoC container is initialized

## Create a scheduled task

Use the `@Scheduled` decorator on a component method:

```python
from py_spring_core import Component
from py_spring_scheduler import Scheduled, IntervalTrigger

class HealthCheckService(Component):
    @Scheduled(trigger=IntervalTrigger(seconds=30))
    def check_health(self):
        print("Running health check...")
```

This runs `check_health()` every 30 seconds. Because it's a component method, you have full access to injected dependencies.

## Schedule with dependency injection

The real power is combining scheduling with DI:

```python
from py_spring_core import Component
from py_spring_scheduler import Scheduled, IntervalTrigger

class MetricsService(Component):
    def collect_metrics(self) -> dict:
        return {"cpu": 42, "memory": 78}

class MetricsReporter(Component):
    metrics_service: MetricsService  # Injected

    @Scheduled(trigger=IntervalTrigger(minutes=5))
    def report_metrics(self):
        metrics = self.metrics_service.collect_metrics()
        print(f"Metrics: {metrics}")
```

## Schedule standalone functions

You can also schedule regular functions (not bound to a component):

```python
from py_spring_scheduler import Scheduled, IntervalTrigger

@Scheduled(trigger=IntervalTrigger(seconds=10))
def background_cleanup():
    print("Cleaning up temp files...")
```

## Add configuration

Add scheduler settings to your `application-properties.json`:

```json
{
    "scheduler": {
        "number_of_workers": 10,
        "max_instances": 3,
        "timezone": "UTC",
        "coalesce": true
    }
}
```

See [Configuration](configuration.md) for all available options.

## Next steps

- Explore all **[trigger types](triggers.md)** — cron, interval, date, and combined triggers
- Tune the scheduler with **[configuration options](configuration.md)**
