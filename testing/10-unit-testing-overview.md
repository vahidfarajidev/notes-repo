
# Unit Testing in C# with xUnit and FluentAssertions

## 🧪 What is xUnit?

xUnit is a modern testing framework for .NET. It supports attributes like:

- `[Fact]` for simple tests
- `[Theory]` with `[InlineData]` for parameterized tests

```csharp
[Fact]
public void Should_Add_TwoNumbers()
{
    var result = Calculator.Add(2, 3);
    Assert.Equal(5, result);
}
```

## ✅ Assert Class

xUnit's `Assert` class provides built-in static methods for validation:

| Method                        | Use                            |
|------------------------------|---------------------------------|
| `Assert.Equal(a, b)`         | Check equality                 |
| `Assert.True(condition)`     | Check if condition is true     |
| `Assert.Throws<>()`          | Check if exception is thrown   |

Example:

```csharp
Assert.Throws<ArgumentNullException>(() => myService.Do(null));
```

## 🔥 FluentAssertions

FluentAssertions is an external library to make your tests more expressive and readable:

```csharp
result.Should().Be(42);
list.Should().Contain("apple");
```

### Exception Assertions

```csharp
Action act = () => productBuilder.Build();

act.Should().Throw<InvalidOperationException>()
   .WithMessage("title is null or empty");
```

## 🎯 Action and Func for Exceptions

To test exceptions, we often wrap the operation inside a delegate:

```csharp
Action act = () => productBuilder.Build();
```

This is necessary because throwing an exception outside a lambda would crash the test.

## 🧠 Extension Methods Behind .Should()

FluentAssertions uses C# extension methods to provide `.Should()` on `Action`, `int`, `string`, etc.

```csharp
"hello".Should().StartWith("h");
someList.Should().HaveCount(3);
```

## ⚙️ What is NCrunch?

NCrunch is not a testing library — it’s a test runner that runs your unit tests continuously as you type.
It’s useful for fast feedback and productivity.

---

## ✅ Assert.Throws vs FluentAssertions for Exception Testing

Both approaches are valid — here's the difference:

### Using xUnit’s Assert:

```csharp
Assert.Throws<ArgumentNullException>(() => myService.Do(null));
```

✅ This is correct and commonly used when you're only working with xUnit.

---

### Using FluentAssertions:

```csharp
Action act = () => productBuilder.Build();
act.Should().Throw<InvalidOperationException>();
```

✅ This is preferred when using FluentAssertions for better readability and chaining:

```csharp
act.Should()
   .Throw<InvalidOperationException>()
   .WithMessage("title is null or empty");
```

---

### Summary:

| Feature               | `Assert.Throws<>()`         | `FluentAssertions`               |
|------------------------|-----------------------------|----------------------------------|
| Built into xUnit       | ✅ Yes                      | ❌ No (external library)         |
| Readability            | Moderate                    | ✅ Very readable and natural     |
| Detailed assertions    | ❌ Basic only               | ✅ Message, inner exception, etc.|
| Chaining support       | ❌ No                       | ✅ Yes                           |
| Better error messages  | ❌ Limited                  | ✅ More descriptive              |

Use `Assert.Throws` when working with plain xUnit.  
Use `act.Should().Throw<>()` when using FluentAssertions for a more expressive test style.
