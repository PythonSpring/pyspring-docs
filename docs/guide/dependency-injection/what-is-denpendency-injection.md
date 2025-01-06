# Dependency Injection (DI)

**Dependency Injection (DI)** is a design pattern and a fundamental concept in software engineering. It is a technique where an object (or function) receives its dependencies from an external source rather than creating them itself. This approach promotes loose coupling between components, making the code more modular, testable, and maintainable.

---

## Core Concepts

### 1. **Dependency**
A dependency is any object or service that another object requires to function. For example, a `Service` class might rely on a `Repository` class to fetch data.

### 2. **Injection**
Injection refers to the process of providing the dependency to the object that needs it. This is usually handled by a **DI container** or **framework**.

---

## Types of Dependency Injection

### 1. **Constructor Injection**
Dependencies are provided through the constructor of the dependent class.

```py
class Repository:
    def fetch_data(self):
        return "Data from repository"

class Service:
    def __init__(self, repository: Repository):
        self.repository = repository

    def get_data(self):
        return self.repository.fetch_data()

repo = Repository()
service = Service(repo)
print(service.get_data())
```


### 2. **Setter Injection**
Dependencies are provided through a setter method after the object is constructed.
```py
class Repository:
    def fetch_data(self):
        return "Data from repository"

class Service:
    def set_repository(self, repository: Repository):
        self.repository = repository

    def get_data(self):
        return self.repository.fetch_data()

repo = Repository()
service = Service()
service.set_repository(repo)
print(service.get_data())
```

## Benefits of Dependency Injection
- **Loose Coupling**:
Classes are decoupled from their dependencies, making them easier to modify and extend.

- **Improved Testability**:
Dependencies can be replaced with mock objects, facilitating unit testing.

- **Easier Maintenance**:
Changes to a dependency's implementation do not affect the dependent classes.

- **Reusability**:
Components are more reusable because they do not rely on specific implementations.


## Challenges Without Dependency Injection

Without DI, objects are responsible for creating their own dependencies, which can lead to several issues:

- **Tight Coupling**
Classes directly instantiate their dependencies, making them tightly coupled. This means any change in the dependency's implementation requires changes in the dependent class.

- **Reduced Testability**
Since dependencies are hard-coded, it becomes challenging to replace them with mock objects during testing, complicating unit testing efforts.

- **Limited Reusability**
Components become less reusable because they are bound to specific implementations rather than abstractions.

- **Complex Maintenance**
With dependencies directly embedded, managing and updating dependencies becomes more difficult, leading to potential code duplication and increased maintenance overhead.

- **Violation of the Single Responsibility Principle**
When classes manage their own dependencies, they take on additional responsibilities beyond their primary purpose, violating the Single Responsibility Principle and reducing code clarity.


### Example Without Dependency Injection

```py
class Repository:
    def fetch_data(self):
        return "Data from repository"

class Service:
    def __init__(self):
        # Directly creating the dependency inside the class
        self.repository = Repository()

    def get_data(self):
        return self.repository.fetch_data()

# Usage
service = Service()
print(service.get_data())
```


---

## Problems in the Code

#### 1. **Tight Coupling**
- The `Service` class directly creates an instance of the `Repository` class in its constructor. This means the `Service` class is tightly coupled to the `Repository` class.
- If you wanted to change the data source, for example, switching from a `Repository` to an `ApiRepository`, you would need to modify the `Service` class. This breaks the principle of loose coupling.

#### 2. **Reduced Testability**
- In this setup, unit testing becomes difficult because we can't replace the `Repository` with a mock or a stub.
- The `Service` class always depends on a real instance of the `Repository` class, making tests harder to isolate and slower to run.

#### 3. **Limited Reusability**
- The `Service` class is not reusable with different kinds of repositories. 
- If you wanted to use the `Service` class with a different repository implementation, you'd have to modify the `Service` class itself. This is not flexible and goes against the idea of creating reusable components.

#### 4. **Complex Maintenance**
- As the application grows and more dependencies are added to the `Service` class, it becomes more difficult to manage.
- If every class is responsible for creating its own dependencies, the codebase becomes harder to maintain, leading to potential code duplication and tightly coupled components.
