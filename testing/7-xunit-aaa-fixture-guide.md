
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

In xUnit, the default behavior is that **each test method gets a new instance of the test class** — this is called a **Transient Fixture**.

## ✅ What is a Transient Fixture?

- Each `[Fact]` test causes xUnit to instantiate the test class anew.
- It doesn't matter whether the class has a constructor or not.
- Until you explicitly use something like `IClassFixture<T>`, **this is the default behavior**.

### 🔧 Example (No constructor, still transient):

```csharp
public class CalculatorTests
{
    private Calculator _calc = new Calculator();

    [Fact]
    public void Add_One() => _calc.Add(1, 2).Should().Be(3);

    [Fact]
    public void Add_Two() => _calc.Add(3, 4).Should().Be(7);
}
```

### 🔍 What happens here?

- For each `[Fact]`, xUnit creates a **new instance** of `CalculatorTests`.
- So `_calc` is initialized **separately for each test**.

---

## ✅ What about constructor-based setup?

Many developers do setup inside the constructor like this:

```csharp
public class MyTests
{
    private readonly Service _service;

    public MyTests()
    {
        _service = new Service(); // setup logic
    }
}
```

🟡 Even here, the behavior is **still transient** — each test gets its own `MyTests` instance.

This does **not** make it shared unless you use a fixture like `IClassFixture<T>`.

---

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
    public void Add_TwoPositiveNumbers_ReturnsCorrectSum() => _calc.Add(1, 2).Should().Be(3);
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
