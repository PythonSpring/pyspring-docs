# Triggers

Triggers define when a scheduled job runs. PySpring Scheduler re-exports all APScheduler trigger types for convenience.

## Interval trigger

Run at fixed intervals:

```python
from py_spring_scheduler import Scheduled, IntervalTrigger

@Scheduled(trigger=IntervalTrigger(seconds=5))
def run_every_5_seconds(self):
    print("Tick!")
```

Common interval parameters:

| Parameter | Type | Description |
|-----------|------|-------------|
| `weeks` | `int` | Interval in weeks |
| `days` | `int` | Interval in days |
| `hours` | `int` | Interval in hours |
| `minutes` | `int` | Interval in minutes |
| `seconds` | `int` | Interval in seconds |

Parameters can be combined: `IntervalTrigger(hours=1, minutes=30)`.

## Cron trigger

Run on a cron-like schedule:

```python
from py_spring_scheduler import Scheduled, CronTrigger

@Scheduled(trigger=CronTrigger(hour=2, minute=0))
def run_at_2am(self):
    print("Nightly job running...")
```

Common cron parameters:

| Parameter | Type | Description |
|-----------|------|-------------|
| `year` | `int/str` | 4-digit year |
| `month` | `int/str` | Month (1-12) |
| `day` | `int/str` | Day of month (1-31) |
| `week` | `int/str` | ISO week number (1-53) |
| `day_of_week` | `int/str` | Day of week (0-6 or mon-sun) |
| `hour` | `int/str` | Hour (0-23) |
| `minute` | `int/str` | Minute (0-59) |
| `second` | `int/str` | Second (0-59) |

### Cron expressions

```python
# Every weekday at 9am
CronTrigger(day_of_week="mon-fri", hour=9, minute=0)

# First day of every month at midnight
CronTrigger(day=1, hour=0, minute=0)

# Every 15 minutes
CronTrigger(minute="*/15")
```

## Date trigger

Run once at a specific date and time:

```python
from py_spring_scheduler import Scheduled, DateTrigger
from datetime import datetime

@Scheduled(trigger=DateTrigger(run_date=datetime(2025, 12, 31, 23, 59)))
def new_years_countdown(self):
    print("Happy New Year!")
```

## Calendar interval trigger

Run at calendar-based intervals (accounts for DST and month length variations):

```python
from py_spring_scheduler import Scheduled, CalendarIntervalTrigger

@Scheduled(trigger=CalendarIntervalTrigger(months=1))
def monthly_report(self):
    print("Generating monthly report...")
```

| Parameter | Type | Description |
|-----------|------|-------------|
| `years` | `int` | Interval in years |
| `months` | `int` | Interval in months |
| `weeks` | `int` | Interval in weeks |
| `days` | `int` | Interval in days |

## Combining triggers

Use `OrTrigger` to fire when **any** trigger matches, or `AndTrigger` to fire when **all** triggers match simultaneously.

### OrTrigger

```python
from py_spring_scheduler import Scheduled, OrTrigger, CronTrigger

@Scheduled(trigger=OrTrigger([
    CronTrigger(hour=9, minute=0),
    CronTrigger(hour=17, minute=0),
]))
def run_at_9am_and_5pm(self):
    print("Morning/evening report!")
```

### AndTrigger

```python
from py_spring_scheduler import Scheduled, AndTrigger, CronTrigger

@Scheduled(trigger=AndTrigger([
    CronTrigger(day_of_week="mon"),
    CronTrigger(hour=9, minute=0),
]))
def monday_morning_standup(self):
    print("Monday standup reminder!")
```

## Available imports

All trigger types are re-exported from `py_spring_scheduler`:

```python
from py_spring_scheduler import (
    IntervalTrigger,
    CronTrigger,
    DateTrigger,
    CalendarIntervalTrigger,
    AndTrigger,
    OrTrigger,
)
```
