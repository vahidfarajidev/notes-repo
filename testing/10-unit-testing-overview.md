
# 🧪 xUnit, Assert, Action, FluentAssertions, and NCrunch

A clear and practical guide to key components in unit testing with C#.

---

## ✅ 1. What is xUnit?

**xUnit** is a modern, lightweight unit testing framework for .NET.  
It supports `[Fact]` for regular tests and `[Theory]` with `[InlineData]` for parameterized tests.

### Example:

```csharp
[Fact]
public void Should_Add_TwoNumbers()
{
    var result = Calculator.Add(2, 3);
    Assert.Equal(5, result);
}
```

---

## ✅ 2. Assert in xUnit

`Assert` is a static class with helper methods to verify expected behavior.

| Method                             | Description                    |
|------------------------------------|--------------------------------|
| `Assert.Equal(expected, actual)`   | Check for equality             |
| `Assert.True(condition)`           | Check if condition is true     |
| `Assert.Throws<Exception>(...)`    | Check if exception is thrown   |

### Example:

```csharp
Assert.Throws<ArgumentNullException>(() => myService.Do(null));
```

---

## ✅ 3. Using Action to Test Exceptions

If a method throws an exception, you can’t pass it directly to `Should()` or `Assert`.  
Instead, wrap it inside an `Action` delegate:

```csharp
Action act = () => productBuilder.Build();

act.Should().Throw<InvalidOperationException>();
```

This lets FluentAssertions or xUnit capture and evaluate the exception.

---

## ✅ 4. What is FluentAssertions?

**FluentAssertions** is a popular third-party library for writing clean, expressive assertions.

### Benefits:
- ✅ More readable
- ✅ Descriptive failure messages
- ✅ Supports objects, collections, exceptions, strings, etc.

### Examples:

```csharp
result.Should().Be(42);
result.Should().NotBeNull();
action.Should().Throw<ArgumentNullException>();
```

🧠 FluentAssertions uses **extension methods** like `.Should()`.

To use it:
```csharp
using FluentAssertions;
```

---

## ✅ 5. What is NCrunch?

**NCrunch** is a continuous testing tool (not an assertion framework).  
It automatically runs tests in the background and shows real-time feedback in Visual Studio.

- 🔁 Automatically runs tests as you type
- ✅ Shows green/red markers next to test lines
- ⏱ Helps with performance analysis and coverage

FluentAssertions ≠ NCrunch — but they’re often used together.

---

## 🧠 Summary Table

| Tool                | Purpose                          | Required? |
|---------------------|----------------------------------|-----------|
| `xUnit`             | Unit testing framework           | ✅        |
| `Assert`            | Core assertion methods           | ✅        |
| `Action`, `Func`    | Wrapping test logic (esp. exceptions) | ✅    |
| `FluentAssertions`  | Cleaner assertions via `.Should()` | Optional but recommended |
| `NCrunch`           | Auto test runner & visual feedback | Optional (commercial)   |

---

Let me know if you want examples using `Func<T>`, async/await, or test setup/teardown next!
