# Delegates in C#

Delegates are a foundational feature in C#, enabling you to treat methods as first-class citizens.

---

## 🔹 What is a Delegate?

A **delegate** is a type that holds a reference to methods with a specific signature.  
It allows you to **store methods in variables**, **pass them as parameters**, and **invoke them dynamically**.

You can think of it as a type-safe function pointer.

---

## ✅ Basic Delegate Declaration (C# 1.0)

```csharp
public delegate int Calculate(int a, int b); // Declare delegate type

public class MathOps
{
    public static int Add(int x, int y) => x + y;
    public static int Multiply(int x, int y) => x * y;
}
```

### Usage:

```csharp
Calculate calc = MathOps.Add;
Console.WriteLine(calc(3, 4)); // 7

calc = MathOps.Multiply;
Console.WriteLine(calc(3, 4)); // 12
```

---

## 🔄 Anonymous Methods (C# 2.0)

To define an inline method with a delegate, the delegate type must be declared first:

```csharp
public delegate int Calculate(int a, int b); // Required

Calculate calc = delegate(int a, int b)
{
    return a + b;
};
Console.WriteLine(calc(2, 3)); // 5
```

> This still **requires the `Calculate` delegate type** to be defined beforehand.

---

## 🧠 Modern Syntax: Lambda Expressions (C# 3.0+)

Lambdas are shorthand for anonymous methods. Cleaner and more powerful:

```csharp
public delegate int Calculate(int a, int b); // Required

Calculate calc = (a, b) => a + b;
Console.WriteLine(calc(5, 6)); // 11
```

---

## ⚠️ Key Point

You **must define** the delegate type (`Calculate`) **unless** you're using built-in generic delegates like `Func<>` or `Action<>`.

---

## 🔌 Built-in Delegates: `Func<>`, `Action<>`, `Predicate<>`

To avoid custom delegate declarations, C# provides reusable generic delegate types:

| Type             | Parameters         | Return Type |
|------------------|--------------------|-------------|
| `Action<T>`      | 0 to 16 parameters | `void`      |
| `Func<T>`        | 0 to 16 parameters | Returns a value |
| `Predicate<T>`   | 1 parameter        | `bool`      |

---

### 🔹 `Func<>` Example (returns a value)

```csharp
Func<int, int, int> add = (a, b) => a + b;
Console.WriteLine(add(10, 20)); // 30
```
If you need a specific delegate with a meaningful name (e.g., for clarity in APIs or events), use a custom delegate.
If you prefer simplicity and flexibility (e.g., for callbacks, LINQ, or quick usage), use Func<> or Action<>.

---

### 🔹 `Action<>` Example (no return)

```csharp
Action<string> greet = name => Console.WriteLine($"Hello, {name}!");
greet("Ali"); // Hello, Ali!
```

---

### 🔹 `Predicate<>` Example (returns bool)

```csharp
Predicate<int> isEven = number => number % 2 == 0;
Console.WriteLine(isEven(4)); // True
```

---

## ✅ Use Cases for Delegates

- **Callback functions** (e.g., pass logic as parameter)
- **Event handling** (e.g., button.Click += ...)
- **LINQ and functional programming**
- **Loose coupling and extensibility**

---

## 🧠 Summary

| Concept           | Description                                |
|-------------------|--------------------------------------------|
| Delegate          | Points to methods with matching signature  |
| Anonymous method  | Inline method using `delegate` keyword     |
| Lambda            | Concise syntax for anonymous methods       |
| Func / Action     | Generic reusable delegates (no need to declare type) |
| Use cases         | Events, callbacks, LINQ, cleaner design    |

Delegates make your C# code **more flexible, testable, and expressive**.
