# Tuples in C#

A **Tuple** is a lightweight data structure that allows you to group multiple values into a single object, without creating a separate class or struct.

---

## 🔹 Basic Example

```csharp
var tuple = (1, "hello", true);
Console.WriteLine(tuple.Item1); // 1
Console.WriteLine(tuple.Item2); // hello
Console.WriteLine(tuple.Item3); // true
```

---

## 🕐 History in C#

| C# Version | Feature                                 | Year |
|------------|------------------------------------------|------|
| C# 4.0     | Introduced `System.Tuple<T1, T2, ...>`   | 2010 |
| C# 7.0     | Introduced `ValueTuple`, named elements, deconstruction | 2017 |

---

## 🔁 `Tuple` vs `ValueTuple`

| Feature          | `System.Tuple`        | `System.ValueTuple`         |
|------------------|------------------------|------------------------------|
| Reference Type   | ✅ Yes                 | ❌ No (it's a struct)        |
| Performance      | Slower                 | Faster (no heap allocation) |
| Named Elements   | ❌ No (`Item1`, etc.)  | ✅ Yes (`Name`, `Age`, etc.) |
| Introduced In    | C# 4.0 / .NET 4.0      | C# 7.0 / .NET Core / 4.7+    |

---

## 🧾 Example with Named Elements (C# 7+)

```csharp
var person = (Name: "Ali", Age: 28);
Console.WriteLine(person.Name); // Ali
Console.WriteLine(person.Age);  // 28
```

You can also deconstruct:

```csharp
var (name, age) = person;
Console.WriteLine($"{name} is {age} years old.");
```

---

## ✅ Use Cases

- Returning multiple values from a method
- Temporary data structures
- Pattern matching and deconstruction
- Functional-style programming

---

## Summary

Tuples provide a simple way to group values without needing custom classes. For better performance and readability, prefer `ValueTuple` (C# 7.0+) over the older `System.Tuple`.



---

## 🧠 What does `var (name, age) = person;` mean?

This is called **deconstruction assignment**. It allows you to extract (or unpack) the components of an object into individual variables.

### ✅ How it works:

```csharp
public record Person(string Name, int Age);

var person = new Person("Sara", 25);
var (name, age) = person;

Console.WriteLine(name); // Sara
Console.WriteLine(age);  // 25
```

The compiler calls the generated `Deconstruct` method and assigns `Name` to `name` and `Age` to `age`.

### Also works with tuples:

```csharp
var person = (Name: "Ali", Age: 30);
var (name, age) = person;
```

> Works only when the type supports deconstruction (e.g., records or ValueTuple).


---

## 🧠 Do variable names have to match during deconstruction?

No — the variable names used during deconstruction **do not need to match** the property names in the original type.

### ✅ Example:

```csharp
public record Person(string Name, int Age);

var person = new Person("Sara", 25);

// These variable names can be anything:
var (x, y) = person;

Console.WriteLine(x); // Sara
Console.WriteLine(y); // 25
```

> Deconstruction is based on **position**, not **name**.

As long as the number and order of variables matches the components provided by the `Deconstruct` method, it works.

### ❗ Just be sure the count matches:

```csharp
public record Person(string FirstName, string LastName, int Age);

var (a, b, c) = new Person("Ali", "Rezaei", 30); // ✅ Works
var (a, b) = new Person("Ali", "Rezaei", 30);    // ❌ Compile error (missing one)
```
