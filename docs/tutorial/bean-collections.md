# Bean Collections

After this page, you'll know how to integrate third-party code and external libraries into PySpring's dependency injection system using `BeanCollection`.

Sometimes you need to use a class you didn't write — a database client, an HTTP client, a cache adapter. You can't make it extend `Component`, because you don't control the source code. That's what `BeanCollection` is for.

## Create a BeanCollection

```python
from py_spring_core import BeanCollection


class MyBean:
    def __init__(self, config_value: str):
        self.config_value = config_value


class AppBeanCollection(BeanCollection):
    def create_my_bean(self) -> MyBean:
        return MyBean("some_value")
```

The convention is simple: any method that starts with `create` and has a return type annotation becomes a **bean factory**. PySpring calls these methods during startup and registers the return values as injectable dependencies.

!!! note
    The method name must start with `create`, and the return type annotation is required. PySpring uses the return type to register the bean in the DI container.

## Inject beans into components

Once a bean is registered, you inject it like any other dependency:

```python
from py_spring_core import Component


class MyBean:
    def __init__(self, config_value: str):
        self.config_value = config_value


class MyService(Component):
    my_bean: MyBean  # Injected from BeanCollection!

    def post_construct(self):
        print(f"Bean config: {self.my_bean.config_value}")
```

## Use Properties in BeanCollection

Bean factories can use Properties for configuration:

```python
from py_spring_core import Properties, BeanCollection


class RedisProperties(Properties):
    __key__ = "redis"
    host: str
    port: int


class MyBean:
    def __init__(self, host: str, port: int):
        self.host = host
        self.port = port


class InfrastructureBeans(BeanCollection):
    redis_properties: RedisProperties  # Injected!

    def create_redis_client(self) -> MyBean:
        return MyBean(
            host=self.redis_properties.host,
            port=self.redis_properties.port,
        )
```

!!! tip
    Properties are injected into `BeanCollection` classes *before* the `create_*` methods are called. So you can safely use them in your factory methods.

## Beans interacting with components

Beans are first-class citizens in PySpring's DI. They can be injected into components, and components can be injected alongside beans:

```python
from py_spring_core import Component, Properties, BeanCollection


class ExampleProperties(Properties):
    __key__ = "example"
    value: str


class MyBean:
    def __init__(self, value: str):
        self.value = value


class AppBeans(BeanCollection):
    example_properties: ExampleProperties

    def create_my_bean(self) -> MyBean:
        return MyBean(self.example_properties.value)


class MainService(Component):
    example_properties: ExampleProperties
    my_bean: MyBean

    def post_construct(self):
        print(f"Properties value: {self.example_properties.value}")
        print(f"Bean value: {self.my_bean.value}")
```

## Recap

Bean Collections let you bring third-party code into PySpring's DI system.

* Create a `BeanCollection` with `create_*` methods
* Return type annotation is required — it's how PySpring registers the bean
* Properties are injected before bean creation
* Beans are injectable into any Component or Controller
* Beans are singletons by default

Next, let's learn about the **Event System** — how components can communicate without direct coupling.
