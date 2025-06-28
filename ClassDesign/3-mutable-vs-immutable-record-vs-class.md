# 🔄 Mutable vs Immutable in C#

In C#, understanding **mutability** is key to writing predictable and safe code.

---

## 🟡 Mutable: "Can Be Changed"

> You can change the object's properties after it's created.

### ✅ Example: Mutable using `class`

```csharp
public class Person
{
    public string Name { get; set; }
}

var person = new Person { Name = "Ali" };
person.Name = "Sara"; // ✅ Allowed
```

- Uses `set`, so property is changeable.
- This object is **mutable**.

---

## 🔵 Immutable: "Cannot Be Changed"

> Once created, the object's data cannot be changed directly.

### ✅ Example: Immutable using `record`

```csharp
public record Person(string Name);

var person = new Person("Ali");
var updated = person with { Name = "Sara" }; // ✅ Creates a new copy
```

- You **cannot modify** `person.Name` directly.
- Using `with`, a **new object** is created.
- This object is **immutable**.

---

## 🔍 Why It Matters

| Feature         | Mutable (`class`)             | Immutable (`record`)               |
|----------------|-------------------------------|------------------------------------|
| In-place change | ✅ Yes                        | ❌ No (requires copy)              |
| Use case        | Complex logic, DB models      | Safe data transfer, pure data      |
| Thread safety   | ❌ Risk of side effects        | ✅ Safer in multithreaded code      |

---

## 🧠 Summary

- **Mutable** = changeable (`class`)
- **Immutable** = not changeable (`record`)
- `record` types are typically immutable and ideal for **data transfer**
- `class` types are better for **behavioral models**

---

# ✅ Is a `record` always immutable?

> **Not necessarily.**

`record` is **immutable by default** only when defined using **positional syntax**:

```csharp
public record Person(string Name, int Age); // Immutable
```

Behind the scenes, the compiler generates:

```csharp
public string Name { get; init; }
```

So you can only set the value **during initialization**.

---

## ❗ But `record` can be mutable if defined manually:

```csharp
public record Person
{
    public string Name { get; set; }  // ✅ Mutable
    public int Age { get; set; }
}
```

- Now it's just like a class with mutable properties.

---

## 🔁 Comparison Table

| Definition                        | Mutable? | Explanation                      |
|----------------------------------|----------|----------------------------------|
| `record Person(string Name)`     | ❌ No    | Uses auto-generated `init` only |
| `record` with `get; set;`        | ✅ Yes   | Writable properties allowed     |

---

# ❌ Common Mistake Example

```csharp
public record Person(string Name);

var person = new Person("Ali");
person.Name = "Sara"; // ❌ Compile-time error
```

- Since `Name` is `init-only`, you can't change it after creation.

---

## ✅ Correct Way (Using `with` expression)

```csharp
var updated = person with { Name = "Sara" };
```

---

## ❗ If You Really Want It Mutable

Define it like this:

```csharp
public record Person
{
    public string Name { get; set; }
}
```

```csharp
var person = new Person { Name = "Ali" };
person.Name = "Sara"; // ✅ Works fine
```

---

## ✅ Summary Table

| Definition                     | Mutable? |
|--------------------------------|----------|
| `record Person(string Name);`  | ❌ No     |
| `record` with `get; set;`      | ✅ Yes    |
