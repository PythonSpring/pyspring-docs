# Scheduling

After this page, you'll know how to run scheduled tasks in PySpring using the scheduler plugin.

PySpring's scheduler is built on APScheduler and fully integrated with the component system — so your scheduled tasks can use dependency injection.

## Install the scheduler

```console
$ pip install git+ssh://git@github.com/PythonSpring/pyspring-scheduler.git
```

## Set up the scheduler

Register the scheduler as an entity provider in your application entry point:

```python
from py_spring_core import PySpringApplication
from pyspring_scheduler import provide_scheduler


def main():
    app = PySpringApplication(
        "./app-config.json",
        entity_providers=[provide_scheduler()],
    )
    app.run()


if __name__ == "__main__":
    main()
```

## Create a scheduled task

Use the `@Scheduled` decorator on any component method:

```python
from py_spring_core import Component
from pyspring_scheduler import Scheduled
from apscheduler.triggers.interval import IntervalTrigger


class HealthCheckService(Component):
    @Scheduled(trigger=IntervalTrigger(seconds=30))
    def check_health(self):
        print("Running health check...")
```

This runs `check_health()` every 30 seconds. Because it's a component method, you have full access to injected dependencies.

## Trigger types

PySpring's scheduler supports several trigger types:

### Interval trigger

Run at fixed intervals:

```python
from apscheduler.triggers.interval import IntervalTrigger

@Scheduled(trigger=IntervalTrigger(seconds=5))
def run_every_5_seconds(self):
    print("Tick!")
```

### Cron trigger

Run on a cron schedule:

```python
from apscheduler.triggers.cron import CronTrigger

@Scheduled(trigger=CronTrigger(hour=2, minute=0))
def run_at_2am(self):
    print("Nightly job running...")
```

### Combining triggers

Use `AndTrigger` or `OrTrigger` for complex schedules:

```python
from apscheduler.triggers.combining import AndTrigger, OrTrigger
from apscheduler.triggers.cron import CronTrigger
from apscheduler.triggers.interval import IntervalTrigger

@Scheduled(trigger=OrTrigger([
    CronTrigger(hour=9, minute=0),
    CronTrigger(hour=17, minute=0),
]))
def run_at_9am_and_5pm(self):
    print("Morning/evening report!")
```

## Scheduled tasks with DI

The real power is combining scheduling with dependency injection:

```python
from py_spring_core import Component
from pyspring_scheduler import Scheduled
from apscheduler.triggers.interval import IntervalTrigger


class MetricsService(Component):
    def collect_metrics(self):
        return {"cpu": 42, "memory": 78}


class MetricsReporter(Component):
    metrics_service: MetricsService  # Injected!

    @Scheduled(trigger=IntervalTrigger(minutes=5))
    def report_metrics(self):
        metrics = self.metrics_service.collect_metrics()
        print(f"Metrics: {metrics}")
```

## Configure the scheduler

Add scheduler configuration to your properties file:

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

## Recap

The scheduler lets you run tasks on a schedule with full DI support.

* Install `pyspring-scheduler` and register with `provide_scheduler()`
* Use `@Scheduled(trigger=...)` on component methods
* Supports interval, cron, and combined triggers
* Full dependency injection in scheduled tasks
* Configurable via properties

Next, let's learn about **Graceful Shutdown** — how to shut down your application cleanly.
