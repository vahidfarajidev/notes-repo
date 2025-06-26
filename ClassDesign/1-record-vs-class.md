# Record vs Class in C#

In C#, both `record` and `class` are used to define reference types, but they are designed for different use cases.

## 🧾 `class`

- Traditional reference type
- Supports inheritance, encapsulation, polymorphism
- By default, compared by reference
- Ideal for mutable objects with behavior (methods)

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

## 📄 `record`

- Introduced in C# 9.0
- Reference type primarily used for **immutable** data models
- By default, compared **by value** (structural equality)
- Built-in support for immutability and concise syntax

```csharp
public record Person(string Name, int Age);
```

## ⚖️ Key Differences

| Feature               | `class`                          | `record`                         |
|----------------------|----------------------------------|----------------------------------|
| Equality             | Reference equality               | Value (structural) equality      |
| Use case             | Mutable objects with behavior    | Immutable data carriers          |
| Syntax               | Verbose                          | Concise                          |
| Inheritance          | Full inheritance support         | Supports inheritance (less used) |
| Deconstruction       | Manual                           | Built-in                         |
| Immutability         | Manual implementation            | Built-in support                 |


| Feature         | `class`                          | `record`                         | Explanation |
|----------------|----------------------------------|----------------------------------|-------------|
| Equality        | Reference equality               | Value (structural) equality      | `record` compares values by default, `class` compares memory references |
| Use case        | Mutable objects with behavior    | Immutable data carriers          | `class` is suited for logic-heavy objects, `record` is for passing data safely |
| Syntax          | Verbose                          | Concise                          | `record` has shorter syntax, can be defined in a single line |
| Inheritance     | Full inheritance support         | Supports inheritance (less used) | Both support inheritance, but it's less common with `record` |
| Deconstruction  | Manual                           | Built-in                         | `record` supports automatic deconstruction into components |
| Immutability    | Manual implementation            | Built-in support                 | `record` is immutable by default; `class` needs manual setup |



## ✅ When to Use What?

- Use `class` when you need:
  - Mutable state
  - Encapsulation of logic
  - Full control over object behavior

- Use `record` when you need:
  - Immutable, concise data models
  - Comparison based on values
  - Easy-to-read DTOs or event models
