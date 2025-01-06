# Component

The `Component` class in PySpring serves as the foundational building block for creating modular, reusable functionalities. It supports lifecycle management, dependency injection, and property configuration, making it an essential entity in the PySpring framework.

---

## Overview

The `Component` class encapsulates specific functionalities and enables seamless integration with the PySpring application context. It is highly configurable, allowing developers to define scope, inject dependencies, and manage lifecycle hooks.

---

## Features

1. **Reusability and Modularity**  
   Components are designed to be reusable and modular, encapsulating specific functionalities that can be leveraged across the application.

2. **Scope Management**  
    The scope determines whether the component instance is shared (`Singleton`) or created a new each time (`Prototype`),  which can be configured using the `ComponentScope` enum.  
    - `Singleton`: Single shared instance throughout the application.  
    - `Prototype`: A new instance is created per request.

3. **Lifecycle Hooks**  
   Components provide hooks for initialization and destruction:  
     - `post_construct()`: Invoked after the component is initialized.  
     - `pre_destroy()`: Invoked before the component is destroyed.

4. **Dependency Injection**  
   Components can be declare as dependencies for being injected to other components by the PySpring framework.

5. **Properties Injection**  
   Properties from configuration files can be injected into components to simplify configuration management.

---

## Configuration

### Nested `Config` Class

The `Component` class includes a nested `Config` class to define configuration options, such as:  

- Allowed scope (`Singleton` or `Prototype`).

---

## Methods

### Lifecycle Methods
- `post_construct()`:  
    Called after initialization to set up the component. Subclasses can override this method.
- `pre_destroy()`:  
    Called before destruction for cleanup. Subclasses can override this method.
- `finish_initialization_cycle()`:  
    Invokes `post_construct()` and finalizes initialization.
- `finish_destruction_cycle()`:  
    Invokes `pre_destroy()` and finalizes destruction.

### Scope Management
- `get_scope()`:  
  Returns the current scope of the component (`Singleton` or `Prototype`).
- `set_scope(scope: ComponentScope)`:  
  Sets the scope of the component.

---

## Usage Example

```py
from py_spring_core import Component, ComponentScope

class MyComponent(Component):
    class Config:
        scope = ComponentScope.Singleton

    def post_construct(self):
        print("Component initialized.")

    def pre_destroy(self):
        print("Component about to be destroyed.")
```