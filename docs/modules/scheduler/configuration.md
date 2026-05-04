# Configuration

The scheduler is configured through the `scheduler` key in your `application-properties.json`.

## Properties

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

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `number_of_workers` | `int` | `20` | Number of threads in the executor pool |
| `max_instances` | `int` | `3` | Maximum concurrent instances of the same job |
| `timezone` | `str` | `"UTC"` | Timezone for all jobs and triggers |
| `coalesce` | `bool` | `false` | Whether to combine missed executions into one |

All properties have defaults, so the `scheduler` key can be minimal or omitted entirely for default behavior.

## number_of_workers

Controls the size of the `ThreadPoolExecutor` used to run jobs. Increase this if you have many concurrent scheduled tasks:

```json
{
    "scheduler": {
        "number_of_workers": 50
    }
}
```

Each scheduled job runs in its own thread from this pool. If all threads are busy, new job executions queue until a thread is available.

## max_instances

Prevents overlapping execution of the same job. If a job is still running when its trigger fires again, additional instances are blocked until the count drops below this limit:

```json
{
    "scheduler": {
        "max_instances": 1
    }
}
```

Setting `max_instances: 1` ensures a job never overlaps with itself.

## timezone

Timezone used for interpreting cron triggers and logging:

```json
{
    "scheduler": {
        "timezone": "America/New_York"
    }
}
```

Uses standard [IANA timezone names](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones).

## coalesce

Controls behavior when the scheduler was down and multiple executions were missed:

- `false` (default) — Run each missed execution separately when the scheduler resumes
- `true` — Combine all missed executions into a single run

```json
{
    "scheduler": {
        "coalesce": true
    }
}
```

Set to `true` for jobs where only the latest execution matters (e.g., cache refresh). Keep `false` for jobs where each execution has side effects that must not be skipped (e.g., sending notifications).

## SchedulerProperties class

The configuration maps to the `SchedulerProperties` class:

```python
from py_spring_scheduler import SchedulerProperties

class SchedulerProperties(Properties):
    __key__ = "scheduler"
    number_of_workers: int = 20
    max_instances: int = 3
    timezone: str = "UTC"
    coalesce: bool = False
```

You can inject `SchedulerProperties` into any component to read the scheduler configuration at runtime:

```python
from py_spring_core import Component
from py_spring_scheduler import SchedulerProperties

class SchedulerMonitor(Component):
    scheduler_properties: SchedulerProperties

    def post_construct(self):
        print(f"Scheduler using {self.scheduler_properties.number_of_workers} workers")
```
