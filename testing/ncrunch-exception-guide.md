
# ⚡ Testing Exceptions in .NET with NCrunch + xUnit + FluentAssertions

This guide explains how to write unit tests for exception scenarios in .NET using **xUnit** and **FluentAssertions**, and how **NCrunch** gives instant feedback on those tests.

---

## ✅ Example: Throwing an Exception (Sync)

Imagine you have a method that throws an `ArgumentException` for invalid input:

```csharp
public class Calculator
{
    public int SquareRoot(int x)
    {
        if (x < 0)
            throw new ArgumentException("Negative input is not allowed");

        return (int)Math.Sqrt(x);
    }
}
```

### 🧪 Test with xUnit + FluentAssertions

```csharp
public class CalculatorTests
{
    [Fact]
    public void SquareRoot_NegativeInput_ThrowsArgumentException()
    {
        var calc = new Calculator();

        Action act = () => calc.SquareRoot(-5);

        act.Should().Throw<ArgumentException>()
           .WithMessage("Negative input is not allowed");
    }
}
```

---

## ✅ Example: Throwing an Exception (Async)

```csharp
public class Calculator
{
    public async Task<int> SquareRootAsync(int x)
    {
        if (x < 0)
            throw new ArgumentException("Negative input is not allowed");

        await Task.Delay(10);
        return (int)Math.Sqrt(x);
    }
}
```

### 🧪 Test with xUnit + FluentAssertions (Async)

```csharp
public class CalculatorTests
{
    [Fact]
    public async Task SquareRootAsync_NegativeInput_ThrowsArgumentException()
    {
        var calc = new Calculator();

        Func<Task> act = async () => await calc.SquareRootAsync(-5);

        await act.Should().ThrowAsync<ArgumentException>()
                 .WithMessage("Negative input is not allowed");
    }
}
```

---

## 🧠 Notes

- Use `Action` for **sync** methods and `Func<Task>` for **async**.
- Use `.Throw<T>()` vs `.ThrowAsync<T>()` depending on method type.
- NCrunch will **automatically run** these tests and show pass/fail status next to each line.

---

## 🧾 Summary

Exception testing is essential for verifying error-handling logic. Combined with NCrunch and FluentAssertions, you get fast, clear feedback with minimal boilerplate.
