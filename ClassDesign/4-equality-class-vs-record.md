# Equality in `class` vs `record` in C#

Understanding how equality works in C# is crucial when designing types. Let's break down the differences between how `class` and `record` handle equality.

---

## 🔍 Feature: Equality

| Feature  | `class`              | `record`                      |
|----------|----------------------|-------------------------------|
| Equality | Reference equality   | Value (structural) equality  |

---

## 🟡 `class` → Reference Equality

Two instances of a class are considered equal **only if they reference the exact same object in memory**.

### Example:

```csharp
var p1 = new Person { Name = "Ali" };
var p2 = new Person { Name = "Ali" };

Console.WriteLine(p1 == p2); // ❌ False (different references)
```

Even if the data is the same, the result is `false` because the objects are separate instances.

---

## 🔵 `record` → Value (Structural) Equality

Two `record` instances are considered equal **if all their properties have the same values**, even if they are different objects.

### Example:

```csharp
public record Person(string Name);

var p1 = new Person("Ali");
var p2 = new Person("Ali");

Console.WriteLine(p1 == p2); // ✅ True (same values)
```

`record` types use **value-based equality** out of the box — no need for extra code.

---

## 🔧 What does "Requires overriding `Equals()`" mean?

By default, `class` uses reference equality. If you want value-based comparison, you must manually override `Equals()` and `GetHashCode()`.

### Without override:

```csharp
public class Person
{
    public string Name { get; set; }
}

var p1 = new Person { Name = "Ali" };
var p2 = new Person { Name = "Ali" };

Console.WriteLine(p1.Equals(p2)); // ❌ False — compares reference
```

### With override:

```csharp
public class Person
{
    public string Name { get; set; }

    public override bool Equals(object obj) =>
        obj is Person other && Name == other.Name;

    public override int GetHashCode() =>
        Name.GetHashCode();
}

var p1 = new Person { Name = "Ali" };
var p2 = new Person { Name = "Ali" };

Console.WriteLine(p1.Equals(p2)); // ✅ True — now compares value
```

This is what `record` does **automatically** for you. 🎉

---

## 👥 Identity-based vs Value-based Objects

### 🔐 Identity-based object (`class`)

Objects where the **identity (e.g., ID)** is more important than just the data.

```csharp
class User
{
    public Guid Id { get; set; }
    public string Name { get; set; }
}

var u1 = new User { Id = Guid.NewGuid(), Name = "Ali" };
var u2 = new User { Id = u1.Id, Name = "Ali" };

Console.WriteLine(u1 == u2); // ❌ False — different references
```

Here, identity is key — not just the values.


In C#, `class` objects use **reference equality** by default when using `==`.

Even if `u1.Id == u2.Id` evaluates to `true`, `u1` and `u2` are still **separate objects** — like two different boxes with the same content.

> ⚠️ Because `class` is a **reference type**, `==` only checks if both variables point to the **same memory address**, not whether their data is equal.


---

### 📄 Value-based object (`record`)

Objects where only the **data values matter**, not the object identity.

```csharp
public record Address(string City, string ZipCode);

var a1 = new Address("Tehran", "12345");
var a2 = new Address("Tehran", "12345");

Console.WriteLine(a1 == a2); // ✅ True — same value
```

---

## 🧠 Summary

| Concept               | `class`                                 | `record`                          |
|------------------------|------------------------------------------|------------------------------------|
| Equality by default    | Reference (manual override needed)       | Value-based (auto-handled)         |
| Use case               | Identity-based (e.g., users, orders)     | Value-based (e.g., DTOs, configs)  |
| Requires `Equals()`?   | ✅ Yes                                    | ❌ No                               |
