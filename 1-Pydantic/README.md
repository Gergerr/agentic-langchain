# Pydantic Fundamentals - Learning Summary

> **Course Context**: This is Section 1 of the LangGraph & LangChain Learning course, focusing on Pydantic fundamentals for data validation and schema definition.

---

## Table of Contents

1. [Introduction to Pydantic](#1-introduction-to-pydantic)
2. [Basic Model Definition](#2-basic-model-definition)
3. [Pydantic vs Dataclasses](#3-pydantic-vs-dataclasses)
4. [Optional Fields](#4-optional-fields)
5. [Collections (Lists)](#5-collections-lists)
6. [Nested Models](#6-nested-models)
7. [Field Customization & Constraints](#7-field-customization--constraints)
8. [JSON Schema Generation](#8-json-schema-generation)
9. [Additional Knowledge & Best Practices](#9-additional-knowledge--best-practices)

---

## 1. Introduction to Pydantic

**Pydantic** is a data validation library for Python that uses Python type hints to validate data at runtime. It is built on top of Python's standard type hints and provides:

- **Automatic type validation** - Data is checked against declared types
- **Type coercion** - Pydantic attempts to convert data to the correct type when possible
- **Clear error messages** - Descriptive validation errors with specific field information
- **JSON serialization** - Easy conversion to/from JSON
- **IDE support** - Full autocompletion and type checking via type hints

### Why Pydantic Matters for LangGraph/LangChain

In AI agent workflows, data structures are critical for:
- State management in LangGraph
- Structured output from LLMs
- Tool input/output validation
- Message passing between agents

---

## 2. Basic Model Definition

### Creating Your First Model

```python
from pydantic import BaseModel

class Person(BaseModel):
    name: str
    age: int
    city: str

# Creating an instance
person = Person(name='Ger', age=35, city='Jakarta')
print(person)           # name='Ger' age=35 city='Jakarta'
print(type(person))     # <class '__main__.Person'>
```

### Key Characteristics

| Feature | Description |
|---------|-------------|
| `BaseModel` | The base class all Pydantic models inherit from |
| Type annotations | Define expected types for each field |
| Automatic validation | Data is validated during instantiation |
| Immutability (optional) | Models can be made immutable with `frozen=True` |

### Type Validation in Action

```python
# This will raise a ValidationError
person_invalid = Person(name='Gaga', age=11, city=12345)
# ValidationError: Input should be a valid string [type=string_type, input_value=12345, input_type=int]
```

**Key takeaway**: Pydantic strictly enforces types. If you declare `city: str`, passing an `int` (like `12345`) will raise a `ValidationError` with a clear message indicating:
- Which field failed (`city`)
- What was expected (`string_type`)
- What was received (`input_type=int`)

---

## 3. Pydantic vs Dataclasses

Python's standard library includes `@dataclass` decorator for creating data containers. Here's the critical difference:

### Dataclass (No Validation)

```python
from dataclasses import dataclass

@dataclass
class PersonDataclass:
    name: str
    age: int
    city: str

# No validation occurs!
person_d = PersonDataclass(name='Ger', age=11, city=123)
print(person_d)  # PersonDataclass(name='Ger', age=11, city=123) - No error!
```

### Comparison Table

| Feature | `@dataclass` | Pydantic `BaseModel` |
|---------|-------------|---------------------|
| Type hints | ✅ Supported | ✅ Supported |
| Type validation | ❌ No | ✅ Yes |
| Type coercion | ❌ No | ✅ Yes |
| JSON serialization | ❌ Manual | ✅ Built-in |
| Schema generation | ❌ No | ✅ Yes |
| Default values | ✅ Yes | ✅ Yes |
| Validation errors | ❌ None | ✅ Detailed |

### When to Use Each

- **Use Dataclasses**: Simple data containers where you don't need validation (internal use, trusted data)
- **Use Pydantic**: External data, API inputs, configuration files, LLM outputs - anywhere data integrity matters

---

## 4. Optional Fields

### Making Fields Optional

Use `Optional[type]` from `typing` module combined with default values:

```python
from typing import Optional

class Employee(BaseModel):
    id: int
    name: str
    department: str
    salary: Optional[float] = None      # Optional with default None
    is_active: Optional[bool] = True    # Optional with default True

# Example 1: Without optional fields
emp1 = Employee(id=1, name="John", department="IT")
print(emp1)  # id=1 name='John' department='IT' salary=None is_active=True

# Example 2: With all fields
emp2 = Employee(id=2, name="Ger", department="IT", salary=100.0, is_active=False)
print(emp2)  # id=2 name='Ger' department='IT' salary=100.0 is_active=False
```

### Understanding Optional Fields

| Syntax | Meaning |
|--------|---------|
| `field: Optional[type]` | Field can be `None` or the specified type |
| `field: Optional[type] = None` | Field is optional, defaults to `None` |
| `field: Optional[type] = default` | Field is optional, has default value |
| `field: type` | Field is **required** |

### Important Notes

- `Optional[X]` is equivalent to `Union[X, None]`
- Even optional fields are validated when values are provided
- Required fields must always be provided

---

## 5. Collections (Lists)

### Working with Lists

```python
from typing import List

class Classroom(BaseModel):
    room_number: str
    students: List[str]    # List of strings
    capacity: int

# Valid instance
classroom = Classroom(
    room_number='B123',
    students=['A', 'B'],
    capacity=30
)
print(classroom)  # room_number='B123' students=['A', 'B'] capacity=30
```

### List Validation

Each element in the list is validated against the declared type:

```python
try:
    invalid_classroom = Classroom(
        room_number='A1',
        students=['Ger', 123],  # 123 is not a string!
        capacity=30
    )
except ValueError as e:
    print(e)
    # 1 validation error for Classroom
    # students.1
    #   Input should be a valid string [type=string_type, input_value=123, input_type=int]
```

**Note**: The error shows `students.1` indicating the second element (index 1) of the students list failed validation.

### Other Collection Types

```python
from typing import List, Set, Dict, Tuple

class AdvancedModel(BaseModel):
    tags: Set[str]                    # Set of unique strings
    metadata: Dict[str, str]          # Dictionary with string keys and values
    coordinates: Tuple[float, float]  # Fixed-size tuple
    scores: List[int]                 # List of integers
```

---

## 6. Nested Models

### Creating Hierarchical Data Structures

Pydantic models can contain other Pydantic models:

```python
class Address(BaseModel):
    street: str
    city: str
    zip_code: int

class Customer(BaseModel):
    customer_id: int
    name: str
    address: Address  # Nested model

# Creating nested instance
customer = Customer(
    customer_id=1,
    name="Emma",
    address={
        "street": "abc",
        "city": "def",
        "zip_code": "10450"  # Note: string "10450" is coerced to int 10450
    }
)
print(customer)
# customer_id=1 name='Emma' address=Address(street='abc', city='def', zip_code=10450)
```

### Key Features of Nested Models

1. **Automatic instantiation**: Pydantic automatically converts dicts to model instances
2. **Type coercion**: `"10450"` (string) was automatically converted to `10450` (int)
3. **Deep validation**: All nested fields are validated
4. **Access pattern**: Use dot notation - `customer.address.city`

### Practical Example: Order System

```python
class Product(BaseModel):
    sku: str
    name: str
    price: float

class OrderItem(BaseModel):
    product: Product
    quantity: int

class Order(BaseModel):
    order_id: str
    items: List[OrderItem]
    total: float

# Complex nested structure
order = Order(
    order_id="ORD-001",
    items=[
        {
            "product": {"sku": "ABC", "name": "Widget", "price": 9.99},
            "quantity": 2
        }
    ],
    total=19.98
)
```

---

## 7. Field Customization & Constraints

### The `Field()` Function

`Field()` provides advanced customization beyond simple type hints:

```python
from pydantic import Field

class Item(BaseModel):
    name: str = Field(min_length=2, max_length=50)
    price: float = Field(gt=0, le=1000)  # Greater than 0, Less than or equal to 1000
    quantity: int = Field(ge=0)          # Greater than or equal to 0

item = Item(name="Book", price=29.99, quantity=10)
print(item)  # name='Book' price=29.99 quantity=10
```

### Common Field Constraints

| Constraint | Applies To | Description |
|------------|-----------|-------------|
| `min_length` | `str`, collections | Minimum length |
| `max_length` | `str`, collections | Maximum length |
| `gt` | numeric | Greater than |
| `ge` | numeric | Greater than or equal |
| `lt` | numeric | Less than |
| `le` | numeric | Less than or equal |
| `multiple_of` | numeric | Must be multiple of value |
| `regex` | `str` | Must match regex pattern |

### Default Values with Field

```python
class User(BaseModel):
    # Required field with description
    username: str = Field(..., description="Unique username for the user")
    
    # Optional with default value
    age: int = Field(default=18, description="User age, defaults to 18")
    
    # Optional with default factory
    email: str = Field(default_factory=lambda: "user@example.com", description="Default email address")

# Usage
user1 = User(username="Geral")
print(user1)  # username='Geral' age=18 email='user@example.com'

user2 = User(username="Bobo", age=25, email="bobo@domain.com")
print(user2)  # username='Bobo' age=25 email='bobo@domain.com'
```

### Understanding `...` (Ellipsis)

- `Field(...)` - The field is **required** (no default value)
- `Field(default=...)` - Sets a default value
- `Field(default_factory=...)` - Uses a callable to generate default (useful for mutable defaults)

---

## 8. JSON Schema Generation

### Automatic Schema Export

Pydantic can generate JSON Schema (OpenAPI compatible) for your models:

```python
print(User.model_json_schema())
```

Output:
```json
{
  "properties": {
    "username": {
      "description": "Unique username for the user",
      "title": "Username",
      "type": "string"
    },
    "age": {
      "default": 18,
      "description": "User age, defaults to 18",
      "title": "Age",
      "type": "integer"
    },
    "email": {
      "description": "Default email address",
      "title": "Email",
      "type": "string"
    }
  },
  "required": ["username"],
  "title": "User",
  "type": "object"
}
```

### Use Cases for JSON Schema

- **API Documentation**: Auto-generate OpenAPI specs for FastAPI
- **Frontend Validation**: Share schemas with frontend for form validation
- **Code Generation**: Generate TypeScript types or other language bindings
- **Data Contracts**: Formalize data exchange formats

### Other Serialization Methods

```python
# Convert to dictionary
user_dict = user1.model_dump()
# {'username': 'Geral', 'age': 18, 'email': 'user@example.com'}

# Convert to JSON string
user_json = user1.model_dump_json()
# '{"username":"Geral","age":18,"email":"user@example.com"}'

# Create from dictionary
user_from_dict = User.model_validate({"username": "Alice", "age": 30})

# Create from JSON string
user_from_json = User.model_validate_json('{"username": "Bob"}')
```

---

## 9. Additional Knowledge & Best Practices

### Type Coercion Examples

Pydantic automatically converts compatible types:

```python
class CoercionExample(BaseModel):
    int_field: int
    float_field: float
    str_field: str
    bool_field: bool

# These all work due to coercion:
example = CoercionExample(
    int_field="42",      # String "42" → int 42
    float_field="3.14",  # String "3.14" → float 3.14
    str_field=123,       # int 123 → str "123"
    bool_field=1         # int 1 → bool True
)
```

**Note**: While convenient, be cautious with coercion in production systems.

### Configuring Model Behavior

```python
from pydantic import ConfigDict

class StrictModel(BaseModel):
    model_config = ConfigDict(
        strict=True,           # Disable type coercion
        frozen=True,           # Make immutable
        extra='forbid',        # Forbid extra fields
        str_to_lower=True,     # Convert strings to lowercase
    )
    name: str
```

### Custom Validators

```python
from pydantic import field_validator

class User(BaseModel):
    name: str
    email: str
    age: int

    @field_validator('email')
    @classmethod
    def validate_email(cls, v: str) -> str:
        if '@' not in v:
            raise ValueError('Invalid email address')
        return v.lower()

    @field_validator('age')
    @classmethod
    def validate_age(cls, v: int) -> int:
        if v < 0 or v > 150:
            raise ValueError('Age must be between 0 and 150')
        return v
```

### Environment Variables with Pydantic Settings

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    api_key: str
    debug: bool = False
    
    class Config:
        env_file = ".env"

settings = Settings()
```

### Integration with LangChain/LangGraph

In LangGraph, Pydantic is commonly used for:

```python
from typing import TypedDict, Annotated
from langgraph.graph import StateGraph
from operator import add

# State definition using Pydantic
class AgentState(BaseModel):
    messages: Annotated[list, add]  # Reducer pattern
    next_step: str
    context: dict

# Structured output from LLM
class LLMResponse(BaseModel):
    reasoning: str
    action: str
    parameters: dict
```

### Performance Tips

1. **Use `model_validate()`** instead of `**dict` unpacking for better performance
2. **Consider `@validate_call`** for function input validation
3. **Use `TypeAdapter`** for validating individual types without models
4. **Enable `defer_build=True`** in config for faster import times

### Common Patterns Summary

```python
# Pattern 1: Basic validation
class Simple(BaseModel):
    value: int

# Pattern 2: Optional with default
class WithDefaults(BaseModel):
    name: str = "default"
    count: Optional[int] = None

# Pattern 3: Constrained fields
class Constrained(BaseModel):
    email: str = Field(pattern=r'^[\w\.-]+@[\w\.-]+\.\w+$')
    score: int = Field(ge=0, le=100)

# Pattern 4: Nested models
class Nested(BaseModel):
    user: User
    items: List[Item]

# Pattern 5: Immutable
class Immutable(BaseModel):
    model_config = ConfigDict(frozen=True)
    value: str
```

---

## Summary Checklist

After completing this section, you should understand:

- [x] How to create basic Pydantic models with `BaseModel`
- [x] The difference between Pydantic and Python dataclasses
- [x] How to define optional fields with `Optional[type]`
- [x] How to work with collection types like `List`, `Set`, `Dict`
- [x] How to create nested/hierarchical models
- [x] How to use `Field()` for constraints and customization
- [x] How to generate JSON Schema from models
- [x] Basic serialization methods (`model_dump`, `model_dump_json`)

---

## Next Steps

With Pydantic fundamentals mastered, you're ready for:
1. **LangChain Basics** - Using Pydantic for structured LLM outputs
2. **LangGraph State Management** - Defining state schemas with Pydantic
3. **Tool Definition** - Creating structured tools with Pydantic models

---

*Generated from `intro.ipynb` - Pydantic v2.x*
