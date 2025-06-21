
# 🧪 Unit Testing Structure: AAA Pattern & Fixtures in .NET

This guide covers two essential concepts in writing clean, maintainable unit tests in .NET:

1. **AAA Pattern** — a structural guideline for test readability  
2. **Fixtures** — mechanisms for managing setup and object lifetime across tests

---

## ✅ 1. AAA Pattern: Arrange – Act – Assert

A widely adopted pattern for structuring test methods, AAA divides tests into three logical sections:

| Phase      | Description                          | Purpose                             |
|------------|--------------------------------------|-------------------------------------|
| **Arrange** | Prepare objects and test data        | Set up context (e.g., mocks, input) |
| **Act**     | Execute the method under test        | Run the functionality               |
| **Assert**  | Verify the outcome                   | Confirm the expected result         |

### 🔍 Example in xUnit with FluentAssertions

```csharp
[Fact]
public void Add_TwoPositiveNumbers_ReturnsCorrectSum()
{
    // Arrange
    var calc = new Calculator();

    // Act
    var result = calc.Add(2, 3);

    // Assert
    result.Should().Be(5);
}
```

---

## ✅ 2. Fixtures: Transient vs Persistent

Fixtures handle **shared setup** across tests and define how long an object should live during test execution.

### 🔹 Transient Fixture

- A **new instance** of the test class is created for **each test method**
- Common when using constructor-based setup

```csharp
public class CalculatorTests
{
    private Calculator _calc = new Calculator();

    [Fact]
    public void Add_Works() => _calc.Add(1, 2).Should().Be(3);
}
```

### 🔹 Persistent Fixture (Shared Across Tests)

- One shared instance is used across multiple tests
- Use `IClassFixture<T>` in xUnit

```csharp
public class CalculatorFixture
{
    public Calculator Calc { get; } = new Calculator();
}

public class CalculatorTests : IClassFixture<CalculatorFixture>
{
    private readonly Calculator _calc;

    public CalculatorTests(CalculatorFixture fixture)
    {
        _calc = fixture.Calc;
    }

    [Fact]
    public void Add_Works() => _calc.Add(1, 2).Should().Be(3);
}
```

---

## 🧠 Summary

| Concept           | Use Case                             | Benefits                              |
|------------------|---------------------------------------|----------------------------------------|
| **AAA Pattern**   | Organizing individual test methods    | Readability, clarity                   |
| **Transient Fixture** | Isolated tests needing fresh state | Safe, isolated testing                 |
| **Persistent Fixture** | Shared setup/data for many tests  | Performance, reuse, consistent state   |

Mastering both **AAA structure** and **fixtures** helps you write professional, clean, and scalable unit tests in .NET.
