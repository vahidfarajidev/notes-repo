# 🔍 How Transient Test Method Execution Works in xUnit

xUnit creates a **new instance of the test class for every test method**.  
This pattern is called **Transient Fixture** behavior, and it ensures test isolation and consistency.

---

## 🔁 How It Works

When you write a test class like this:

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

xUnit executes it as if it were doing this:

```csharp
var test1 = new CalculatorTests();  // new Calculator() happens here
test1.Add_One();

var test2 = new CalculatorTests();  // new Calculator() happens again
test2.Add_Two();
```

---

## 🧠 Why This Matters

| Feature      | Benefit                                  |
|--------------|-------------------------------------------|
| Isolation    | Each test has its own clean environment   |
| Consistency  | No shared state between tests             |
| Stability    | Easier to debug and avoids flaky tests    |
| Parallelism  | Safe to run tests in parallel             |

---

## ✅ Official xUnit Behavior

> "xUnit.net creates a new instance of the test class for every test that is run."  
> — [xUnit Docs – Shared Context](https://xunit.net/docs/shared-context#class-fixture)

---

## Summary

This default behavior ensures that tests are **independent, reliable, and clean** — and it's why you often see fields or constructor logic being re-executed per test.
