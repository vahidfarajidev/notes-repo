
# Mutable vs Immutable in C#

In C#, the concepts of **mutable** (changeable) and **immutable** (unchangeable) objects are important in designing robust applications, especially in areas like API design, concurrent programming, and data modeling.

---

## ✅ Definitions

### 🔁 Mutable
An object is mutable if its internal state can be changed after it is created.

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}

var person = new Person { Name = "Ali", Age = 30 };
person.Age = 31; // This is allowed
```

---

### 🔒 Immutable
An object is immutable if its state cannot be changed after it is created. All properties must be set at construction and cannot be modified later.

```csharp
public class Person
{
    public string Name { get; }
    public int Age { get; }

    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}

var person = new Person("Ali", 30);
// person.Age = 31; // Error: read-only property
```

---

## 🔍 Key Notes

- **`string` in C# is immutable.**

```csharp
string a = "hello";
string b = a;
a = "world"; // b is still "hello"
```

- **Structs should generally be designed as immutable**, since mutable structs can behave unexpectedly in certain scenarios like collections.

- To create an immutable class:
  - Use `readonly` fields or properties with only getters.
  - Set all values in the constructor.
  - Use `record` types (from C# 9 onwards).

---

## 💡 Example with `record` (Immutable by default)

```csharp
public record Person(string Name, int Age);

var p1 = new Person("Ali", 30);
var p2 = p1 with { Age = 31 }; // p1 remains unchanged
```

---

## 📌 When to Use Immutable Objects

- In multithreaded (thread-safe) applications
- To prevent bugs caused by unexpected state changes
- When designing domain models
- In functional programming patterns
