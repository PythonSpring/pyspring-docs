# Reference

API reference documentation for PySpring.

!!! info
    This section is under development. For now, refer to the source code and the Tutorial for usage details.

## Core Classes

* `PySpringApplication` — the main application entry point
* `Component` — base class for managed components
* `Properties` — base class for configuration models
* `RestController` — base class for REST API controllers
* `BeanCollection` — base class for third-party integration

## Decorators

* `@GetMapping`, `@PostMapping`, `@PutMapping`, `@DeleteMapping`, `@PatchMapping` — route decorators
* `@EventListener` — event handler decorator
* `@Scheduled` — scheduled task decorator (via pyspring-scheduler)
