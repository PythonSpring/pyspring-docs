# First Steps

You're going to create a minimal PySpring application from scratch — a working web server with dependency injection in just a few files.

## Create the application entry point

Create a file `main.py`:

```python
from py_spring_core import PySpringApplication


def main():
    app = PySpringApplication("./app-config.json")
    app.run()


if __name__ == "__main__":
    main()
```

That's your entire entry point. `PySpringApplication` handles:

* Generating config files if they don't exist
* Loading and validating configuration
* Scanning for components
* Resolving dependencies
* Starting the web server

## Run it

```console
$ python main.py
```

On the first run, PySpring's `ConfigFileTemplateGenerator` detects that `app-config.json` and `application-properties.json` don't exist yet — and **generates them automatically** with sensible defaults. 🚀

You'll see something like:

```console
[APP CONFIG GENERATED] App config file not exists, ./app-config.json generated
[APP PROPERTIES GENERATED] App properties file not exists, ./application-properties.json generated
```

The generated `app-config.json` looks like this:

```json
{
    "app_src_target_dir": "./src",
    "server_config": {
        "host": "0.0.0.0",
        "port": 8080,
        "enabled": true
    },
    "properties_file_path": "./application-properties.json",
    "loguru_config": {
        "log_file_path": "./logs/app.log",
        "log_level": "DEBUG"
    },
    "type_checking_mode": "strict",
    "shutdown_config": {
        "timeout_seconds": 30.0,
        "enabled": true
    }
}
```

!!! tip
    You never need to write the config file by hand. Just run your app and PySpring generates it for you. Edit it afterwards if you need to change the defaults.

!!! info
    By default, PySpring starts the server on `http://0.0.0.0:8080`.

## Check the docs

Open your browser at <a href="http://127.0.0.1:8080/docs" target="_blank">http://127.0.0.1:8080/docs</a>.

You'll see the automatic Swagger UI documentation — even though you haven't defined any routes yet. That's FastAPI working under the hood. ✨

## Add a simple controller

Let's make it do something. Create `controllers.py`:

```python
from py_spring_core import RestController
from py_spring_core.core.entities.controllers.rest_controller import GetMapping


class HelloController(RestController):
    class Config:
        prefix = "/api"

    @GetMapping("/hello")
    def hello(self):
        return {"message": "Hello from PySpring!"}
```

Now restart the application and go to <a href="http://127.0.0.1:8080/docs" target="_blank">http://127.0.0.1:8080/docs</a>. You'll see your `/api/hello` endpoint listed.

!!! tip
    PySpring automatically discovers and registers your controllers. You don't need to import them into `main.py` — just make sure they're in the scanned package.

## Recap

You just built a working PySpring application. Here's what you did:

* Created an entry point (`main.py`) with `PySpringApplication`
* Config files (`app-config.json`, `application-properties.json`) were **auto-generated** on first run
* Added a REST controller with a route
* Got automatic OpenAPI docs for free

Next, let's learn about **Components** — the core building blocks of every PySpring application.
