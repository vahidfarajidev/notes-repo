
# Deconstruction: `record` vs `class` in C#

Deconstruction lets you extract values from an object using tuple syntax.

## ✅ With `record`

`record` types automatically support deconstruction — no extra code needed.

```csharp
public record Person(string Name, int Age);

var person = new Person("Alice", 30);
var (name, age) = person; // Works out of the box
```

> The compiler generates a `Deconstruct` method for you.

---

## ❌ With `class`

`class` types **don’t** support deconstruction by default. You have to implement it manually:

```csharp
public class Person
{
    public string Name { get; }
    public int Age { get; }

    public Person(string name, int age) =>
        (Name, Age) = (name, age);

    public void Deconstruct(out string name, out int age) =>
        (name, age) = (Name, Age);
}

var (name, age) = new Person("Bob", 40); // Works after manual setup
```

### 🔍 Why does this work?

The C# compiler looks for a method named **`Deconstruct`** with an exact signature matching the number and type of values you're extracting:

- It must be named `Deconstruct`
- It must return `void`
- All parameters must be marked `out`

For example:

```csharp
public void Deconstruct(out string name, out int age)
```

If you rename the method or change the signature, **destructuring will not work**:

```csharp
public void Split(out string name, out int age) // ❌ Won’t work
```

> ✅ The name `Deconstruct` is **not a reserved keyword**, but the compiler treats it specially in the context of destructuring.

---

## Summary Table

| Feature        | `class`          | `record`         |
|----------------|------------------|------------------|
| Deconstruction | Manual           | Built-in         |

> ✔ Use `record` for immutable data containers  
> ✔ Use `class` when you need full control over behavior
