# SQL Statement Generation Algorithm

## Overview


## Overview
In PySpringModel, repository methods can be defined using expressive method names like `find_by_email_and_age_or_status`. These method names are parsed and dynamically translated into SQL queries. This allows developers to define powerful queries declaratively without writing raw SQL or using query builders explicitly.

When you define a repository method like `find_by_email_and_age_or_status`, PySpringModel needs to convert this method name into a valid SQL query. The algorithm parses the method name, extracts field names and boolean operators, then constructs the appropriate WHERE clause.

## How It Works
The query generation engine transforms method names into SQL WHERE clauses using a structured, stack-based algorithm that interprets field names and boolean logic (AND/OR). It ensures correct operator precedence and type-safe condition generation via SQLAlchemy.


## Algorithm Steps

### 1. Initial Condition Stack Creation

The algorithm starts by creating a stack of basic equality conditions:

```python
# For method: find_by_email_and_age_or_status
# Required fields: ["email", "age", "status"]
# Parameters: {"email": "john@example.com", "age": 25, "status": "active"}

# Initial stack contains basic equality conditions
stack = [
    User.email == "john@example.com",
    User.age == 25,
    User.status == "active"
]
```

Each condition compares a model field with its corresponding parameter value.

### 2. Boolean Expression Processing

The algorithm processes boolean operators in order (left-to-right) using a stack-based approach:

```python
# Notations: [ConditionNotation.AND, ConditionNotation.OR]

# Step 1: Process AND operator
# Pop two conditions from stack
right_condition = stack.pop()  # User.age == 25
left_condition = stack.pop()   # User.email == "john@example.com"

# Combine with AND
combined_condition = and_(left_condition, right_condition)
# Result: and_(User.email == "john@example.com", User.age == 25)

# Push result back to stack
stack = [combined_condition, User.status == "active"]

# Step 2: Process OR operator
# Pop two conditions from stack
right_condition = stack.pop()  # User.status == "active"
left_condition = stack.pop()   # and_(User.email == "john@example.com", User.age == 25)

# Combine with OR
final_condition = or_(left_condition, right_condition)
# Result: or_(and_(User.email == "john@example.com", User.age == 25), User.status == "active")

# Final stack contains the complete WHERE condition
stack = [final_condition]
```

### 3. Final Query Construction

The algorithm creates the complete SQL query by applying the final condition:

```python
# Create base SELECT statement
base_query = select(User)

# Apply the final combined condition as WHERE clause
final_query = base_query.where(stack[0])

# Result: SELECT * FROM user WHERE (email = ? AND age = ?) OR status = ?
```

## Example Walkthrough

### Input
- **Method Name**: `find_by_email_and_age_or_status`
- **Required Fields**: `["email", "age", "status"]`
- **Parameters**: `{"email": "john@example.com", "age": 25, "status": "active"}`
- **Boolean Operators**: `[AND, OR]`

### Step-by-Step Execution

#### Initial State
```
Stack: [email == "john@example.com", age == 25, status == "active"]
Operators: [AND, OR]
```

#### Step 1: Process AND
```
Pop: age == 25 (right)
Pop: email == "john@example.com" (left)
Combine: and_(email == "john@example.com", age == 25)
Stack: [and_(email == "john@example.com", age == 25), status == "active"]
```

#### Step 2: Process OR
```
Pop: status == "active" (right)
Pop: and_(email == "john@example.com", age == 25) (left)
Combine: or_(and_(email == "john@example.com", age == 25), status == "active")
Stack: [or_(and_(email == "john@example.com", age == 25), status == "active")]
```

#### Final Result
```sql
SELECT * FROM user WHERE (email = ? AND age = ?) OR status = ?
```

## Key Characteristics

### Stack-Based Processing
- Uses a stack data structure to maintain intermediate results
- Each boolean operation pops two conditions and pushes one combined result
- Ensures proper operator precedence and grouping

### Left-to-Right Evaluation
- Processes boolean operators in the order they appear in the method name
- Maintains the logical flow of the original method name
- Supports complex nested expressions

### Dynamic Field Mapping
- Maps method parameters to model attributes automatically
- Handles different data types (strings, integers, booleans, etc.)
- Supports field name conversion between Python and SQL conventions

### Type Safety
- Uses SQLAlchemy's `ColumnElement[bool]` for type-safe conditions
- Prevents SQL injection through parameterized queries
- Maintains type checking throughout the process

### Extensibility
- Supports any number of fields and boolean operators
- Can be extended for additional operators (NOT, XOR, etc.)
- Framework for adding custom query patterns

## Performance Considerations

### Memory Efficiency
- Stack operations are O(1) for push/pop operations
- No unnecessary intermediate objects created
- Minimal memory footprint for complex expressions

### Query Optimization
- Generates optimized SQLAlchemy expressions
- Leverages SQLAlchemy's query optimization
- Supports database-specific optimizations

## Real-World Examples

### Simple AND Condition
```python
# Method: find_by_name_and_email
# Generated SQL: SELECT * FROM user WHERE name = ? AND email = ?
```

### Simple OR Condition
```python
# Method: find_by_status_or_age
# Generated SQL: SELECT * FROM user WHERE status = ? OR age = ?
```

### Complex Mixed Conditions
```python
# Method: find_by_name_and_email_or_status_and_age
# Generated SQL: SELECT * FROM user WHERE (name = ? AND email = ?) OR (status = ? AND age = ?)
```

### Multiple OR Conditions
```python
# Method: find_by_name_or_email_or_status
# Generated SQL: SELECT * FROM user WHERE name = ? OR email = ? OR status = ?
```

## Integration with PySpringModel

This algorithm is used by the `CrudRepositoryImplementationService` to:

1. **Parse Method Names**: Extract field names and boolean operators
2. **Generate SQL**: Create type-safe SQLAlchemy expressions
3. **Execute Queries**: Run the generated queries with proper parameter binding
4. **Return Results**: Handle single results (`Optional[Model]`) or multiple results (`List[Model]`)

## Best Practices

### Method Naming
- Use clear, descriptive field names
- Follow the established naming conventions
- Keep method names readable and logical

### Parameter Types
- Ensure parameter types match model field types
- Use appropriate data types for database columns
- Handle null values appropriately

### Query Complexity
- For simple queries, use dynamic method names
- For complex queries, use custom SQL with `@Query` decorator
- Consider performance implications for large datasets

## Limitations

### Current Limitations
- Only supports AND and OR operators
- No support for NOT, XOR, or other boolean operators
- Limited to equality comparisons (no <, >, LIKE, etc.)
- No support for complex expressions or subqueries

## Conclusion
This dynamic query generation mechanism empowers developers to write clean, maintainable repository code without sacrificing the expressiveness or performance of SQL. By leveraging naming conventions and a structured parsing approach, PySpringModel minimizes boilerplate and maximizes flexibility.