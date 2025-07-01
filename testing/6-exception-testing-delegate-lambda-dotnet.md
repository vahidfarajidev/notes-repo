
# 🧠 Understanding `Action` and `Func<Task>` with Lambda Expressions in .NET

This guide provides a clear and practical explanation of the syntax behind `Action act = () => ...` and `Func<Task> act = async () => ...` in the context of unit testing with xUnit and FluentAssertions.

---

## ✅ Core Concepts

### `Action`
Represents a **method that returns void** and takes no parameters.

Example:
```csharp
void DoSomething()
```

### `Func<Task>`
Represents an **async method returning a Task**, with no parameters.

Example:
```csharp
async Task DoSomethingAsync()
```

---

## 🔍 Code Walkthrough

### Synchronous case:
✔ Version 1 (concise lambda syntax):

```csharp
Action act = () => calc.SquareRoot(-5);
```

This means:
- We define a delegate `act` that takes no parameters.
- When `act()` is invoked, it will call `calc.SquareRoot(-5)`.

This is useful in FluentAssertions:

```csharp
act.Should().Throw<ArgumentException>();
```

✔ Version 2 (explicit new Action with code block):
```csharp
var result = new Action(() => 
{
    calc.SquareRoot(-5);
});

result.Should().Throw<ArgumentException>();
```

This form is especially helpful when your test logic includes multiple lines, or when you prefer a more explicit delegate creation syntax.

For single-line actions, the lambda expression syntax is generally preferred.

---

### Asynchronous case:

```csharp
Func<Task> act = async () => await calc.SquareRootAsync(-5);
```

This creates an async delegate that:
- Returns a Task
- Can be awaited inside your unit test

Used like this:

```csharp
await act.Should().ThrowAsync<ArgumentException>();
```

---

## ❓ Why not call the method directly?

```csharp
calc.SquareRoot(-5); // ❌ This throws immediately
```

If you call it directly, FluentAssertions never gets the chance to verify the exception.

By wrapping it in a delegate:

```csharp
Action act = () => calc.SquareRoot(-5);
```

You give control to the test framework to invoke and assert as needed.

---

## 🧠 Analogy Table

| Concept            | Meaning                                      |
|--------------------|----------------------------------------------|
| `Action`           | A method that returns nothing (void)         |
| `Func<Task>`       | An async method returning a Task             |
| Lambda expression  | A short-hand way to define an inline method  |
| `act()`            | Executes the code inside the lambda          |

---

## 🔁 Traditional Alternatives to Lambda

If lambda expressions didn’t exist, you’d write this instead:

### ✅ With Named Method:
```csharp
void DoTheThing()
{
    calc.SquareRoot(-5);
}
Action act = DoTheThing;
```

### ✅ With Anonymous Method (old-style syntax):
```csharp
Action act = delegate {
    calc.SquareRoot(-5);
};
```

### 🔄 Comparison:

| Type             | Syntax                                  |
|------------------|------------------------------------------|
| Lambda           | `() => calc.SquareRoot(-5)`              |
| Anonymous method | `delegate { calc.SquareRoot(-5); }`      |
| Named method     | `void Do() { ... } → Action act = Do;`   |

---

## 🧾 Summary

Understanding lambda expressions with `Action` and `Func<Task>` is essential for clean unit tests — especially when verifying exceptions using libraries like FluentAssertions. They offer a flexible and elegant way to defer execution and maintain readable code.

---
