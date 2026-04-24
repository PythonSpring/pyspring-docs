# Skills — Claude Code Integration

After this page, you'll know how to use **PySpring Skills** to supercharge your development workflow with Claude Code — generating projects, adding entities, and avoiding common mistakes automatically.

## What are PySpring Skills?

PySpring Skills is a [Claude Code](https://docs.anthropic.com/en/docs/claude-code) custom skill package that teaches the AI assistant how to build and debug PySpring applications. It encodes framework conventions, common pitfalls, and code generation templates so Claude can help you write correct PySpring code from the start.

The skill activates automatically when Claude detects PySpring-related code — imports from `py_spring_core`, configuration files, or Spring Boot-style dependency injection patterns.

## Install the skill

Clone the skills repository into your Claude Code skills directory:

```console
$ git clone https://github.com/PythonSpring/pyspring-skills.git ~/.claude/skills/pyspring-skills
```

That's it. Claude Code will pick up the skill automatically on the next conversation.

!!! tip
    The skill is defined in `SKILL.md` at the repository root. Claude reads this file to understand PySpring's mental model, entity patterns, and common mistakes.

## Repository structure

```
pyspring-skills/
├── SKILL.md                           # Core skill definition and rules
├── README.md                          # Documentation
├── scripts/
│   ├── scaffold_project.py            # Full project skeleton generator
│   └── add_entity.py                  # Single entity file generator
└── references/
    ├── config-files.md                # Configuration file schemas
    ├── events.md                      # Event pub/sub system
    ├── middleware.md                   # Middleware configuration
    ├── qualifiers-and-lifecycle.md    # DI scopes & lifecycles
    ├── scheduling.md                  # @Scheduled task decorator
    └── shutdown.md                    # Graceful shutdown patterns
```

| Directory | Purpose |
|-----------|---------|
| `SKILL.md` | Main skill definition — mental model, patterns, pitfalls, verification checklist |
| `scripts/` | Code generation tools for scaffolding projects and adding entities |
| `references/` | Topic-specific guides loaded on-demand during conversations |

## Scaffold a new project

The `scaffold_project.py` script generates a complete, runnable PySpring project:

```console
$ python scripts/scaffold_project.py my_app --name "My App" --port 8080
```

This creates:

```
my_app/
├── main.py                        # Application entry point
├── app-config.json                # Framework configuration
├── application-properties.json    # Application properties
├── requirements.txt               # Dependencies
├── .gitignore
├── README.md
└── src/
    ├── controllers/               # REST controllers
    ├── services/                  # Business logic components
    ├── properties/                # Typed configuration classes
    ├── beans/                     # BeanCollection factories
    ├── events/                    # Application events
    └── middleware/                # Request interceptors
```

The generated project includes a working `HelloService`, `HelloController`, and `AppProperties` — so you can run it immediately and see something working.

```console
$ cd my_app
$ pip install -r requirements.txt
$ python main.py
```

!!! info
    The scaffold script refuses to overwrite existing non-empty directories, so it's safe to run without accidentally clobbering your work.

## Add entities with the CLI

The `add_entity.py` script generates individual entity files from templates:

```console
$ python scripts/add_entity.py <kind> <ClassName> [--project-root .] [--prefix /api/...]
```

### Supported entity kinds

| Kind | Description | Output directory |
|------|-------------|-----------------|
| `component` | Business-logic service | `src/services/` |
| `controller` | REST endpoint group | `src/controllers/` |
| `properties` | Typed configuration class | `src/properties/` |
| `bean-collection` | Third-party object factory | `src/beans/` |
| `event` | Application event model | `src/events/` |
| `middleware` | Request interceptor | `src/middleware/` |
| `scheduled` | Periodic task | `src/services/` |
| `shutdown` | Graceful shutdown handler | `src/` |

### Examples

Add a REST controller:

```console
$ python scripts/add_entity.py controller UserController --prefix /api/users
```

This generates `src/controllers/user_controller.py` with the correct class structure, route prefix, and import statements.

Add a component:

```console
$ python scripts/add_entity.py component OrderService
```

Add a properties class:

```console
$ python scripts/add_entity.py properties DatabaseProperties
```

Add a scheduled task:

```console
$ python scripts/add_entity.py scheduled CleanupTask
```

!!! tip
    Class names are automatically converted to snake_case filenames. The script also refuses to overwrite existing files.

## What the skill teaches Claude

When the skill is active, Claude understands PySpring's key conventions:

### Mental model

| PySpring concept | What it does |
|-----------------|--------------|
| `Component` | DI-managed class with lifecycle hooks |
| `Properties` | Typed config bound from JSON/YAML via `__key__` |
| `RestController` | Class-based route groups with `Config.prefix` |
| `BeanCollection` | Factory producing third-party DI-registered objects |
| `ApplicationEvent` | Pydantic pub/sub models |
| `Middleware` | Async request interceptors |
| `@Scheduled` | Cron/interval tasks (requires `pyspring-scheduler`) |
| `GracefulShutdownHandler` | Coordinated shutdown across components |

### Common mistakes Claude will help you avoid

The skill encodes the **top 8 PySpring-specific mistakes** so Claude catches them before you ship:

1. **DI-dependent setup in `__init__`** — dependencies aren't injected yet at `__init__` time. Use `post_construct()` instead.

2. **Missing return type on `create_*` methods** — `BeanCollection` factory methods require a return type annotation for DI registration.

3. **Method name doesn't start with `create`** — `BeanCollection` only picks up methods prefixed with `create`.

4. **Multiple implementations without qualifiers** — when two classes extend the same base, use `Annotated[Base, "ClassName"]` to disambiguate.

5. **Circular dependencies** — PySpring rejects these at startup. Restructure with events or interfaces.

6. **Missing `__key__` on Properties** — `Properties` classes must define `__key__` to map to the config file section.

7. **Using FastAPI patterns directly** — avoid `Depends()`, `APIRouter`, etc. Use PySpring's class-based approach instead.

8. **Wrong port assumption** — PySpring defaults to port **8080**, not 8000.

### Pre-ship checklist

Claude will also verify your code against this checklist:

- [ ] All used fields have class-level type annotations
- [ ] Setup logic is in `post_construct()`, not `__init__`
- [ ] `create_*` methods have return type annotations
- [ ] `Properties` classes have `__key__`
- [ ] Controllers set `Config.prefix`
- [ ] Scheduler is enabled in config if using `@Scheduled`
- [ ] Multiple implementations of the same base use `Annotated`

## Writing your own reference guides

The `references/` directory contains topic-specific deep-dives that Claude loads on-demand. You can add your own:

1. Create a Markdown file in `references/`:

    ```console
    $ touch references/my-topic.md
    ```

2. Write framework-specific guidance — patterns, examples, and anti-patterns:

    ```markdown
    # My Topic

    ## When to use this
    ...

    ## Example
    ...

    ## Common mistakes
    ...
    ```

3. Claude will automatically discover and reference it when the topic comes up in conversation.

## Recap

PySpring Skills gives Claude Code deep knowledge of the PySpring framework.

* Install by cloning into `~/.claude/skills/pyspring-skills`
* Scaffold entire projects with `scaffold_project.py`
* Add individual entities with `add_entity.py` — supports 8 entity kinds
* Claude automatically avoids the top 8 PySpring mistakes
* Extend by adding your own reference guides to `references/`

For more details, see the [pyspring-skills repository](https://github.com/PythonSpring/pyspring-skills).
