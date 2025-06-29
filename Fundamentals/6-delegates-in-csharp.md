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


## Why `delegate` is Used Again in Assignment

The keyword `delegate` here is used to define an **anonymous function**.

---

## 🔍 Detailed Explanation:

### 1. Declaring a delegate type:

```csharp
public delegate int Calculate(int a, int b);
```

This defines a new type called `Calculate` with a specific signature:
It takes two `int` parameters and returns an `int`.

---

### 2. Creating an instance of the delegate with an anonymous method:

```csharp
Calculate calc = delegate(int a, int b)
{
    return a + b;
};
```

In this example:

- `delegate(int a, int b) { return a + b; }` is an **anonymous method**.
- This method matches the exact signature defined by the `Calculate` delegate.
- Therefore, it can be assigned directly to `calc`, which is of type `Calculate`.

---

## ✅ Equivalent Using Lambda Expression:

```csharp
Calculate calc = (a, b) => a + b;
```

The lambda expression is just a **newer and cleaner syntax** for doing the same thing as `delegate(...) { ... }`.

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

To specify the signature of a function for use as a delegate, you **must define** a delegate type (such as `Calculate`);
    unless you use built-in generic delegates like `Func<>`, `Action<>` or `Predicate<>`, which cover common method signatures.

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

## Defining a Custom `delegate`:

```csharp
public delegate int Calculate(int a, int b); // Custom delegate definition
Calculate calc = (a, b) => a + b;
```

## ✅ Modern Replacement Using `Func`:

```csharp
Func<int, int, int> add = (a, b) => a + b;
```

## 🔍 Detailed Comparison:

| Part                                     | Explanation                                                             |
|------------------------------------------|-------------------------------------------------------------------------|
| `public delegate int Calculate(int a, int b)` | Defines a delegate type that takes two `int` inputs and returns an `int` |
| `Func<int, int, int>`                   | Equivalent generic built-in type provided by .NET                        |

---

## Summary:

```csharp
Func<int, int, int>
```

is effectively equivalent to:

```csharp
delegate int (int, int)
```

So:

```csharp
Func<int, int, int> add
```

does the same thing as:

```csharp
Calculate calc
```

— but without the need to define a custom delegate type.

---
### 🔹 `Action<>` Example (no return)

```csharp
Action<string> greet = name => Console.WriteLine($"Hello, {name}!");
greet("Ali"); // Hello, Ali!
```

or

```csharp
Action<string> greet = name =>
{
    Console.WriteLine("------");
    Console.WriteLine($"Hello, {name}!");
    Console.WriteLine("Welcome!");
};
```

---

### 🔹 `Predicate<>` Example (returns bool)

```csharp
Predicate<int> isEven = number => number % 2 == 0;
Console.WriteLine(isEven(4)); // True
```

---

If you need a specific delegate with a meaningful name (e.g., for clarity in APIs or events), use a custom delegate.
If you prefer simplicity and flexibility (e.g., for callbacks, LINQ, or quick usage), use Func<>, Action<> or Predicate<>.

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
