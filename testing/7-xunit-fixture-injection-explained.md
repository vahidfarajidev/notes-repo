## 🔍 What Is `IClassFixture<T>` in xUnit?

In xUnit, `IClassFixture<T>` is an interface that tells the test runner how to inject shared setup (fixtures) into your test class. It enables **dependency injection** of reusable objects that need to be set up once and reused across multiple test methods in a class.

---

### ✅ Without `IClassFixture<T>` — This Will Fail:

```csharp
public class CalculatorTests
{
    private readonly Calculator _calc;

    // ❌ This will not work — xUnit won't know how to resolve this parameter
    public CalculatorTests(CalculatorFixture fixture)
    {
        _calc = fixture.Calc;
    }

    [Fact]
    public void Add_TwoPositiveNumbers_ReturnsCorrectSum()
        => _calc.Add(1, 2).Should().Be(3);
}
```

🛑 You’ll get an error like:
> Test class has a constructor with parameters, and no matching `IClassFixture<>` or other DI configured.

---

### ✅ Correct Usage — With `IClassFixture<T>`:

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
    public void Add_TwoPositiveNumbers_ReturnsCorrectSum()
        => _calc.Add(1, 2).Should().Be(3);
}
```

✅ Now xUnit knows to:
- Create a single instance of `CalculatorFixture`
- Inject it into your test class constructor

---

### 🧠 Summary

| Case                                | Requires `IClassFixture<T>`? | Why?                                    |
|-------------------------------------|-------------------------------|-----------------------------------------|
| Constructor has parameters          | ✅ Yes                        | xUnit needs to know what to inject      |
| Constructor has no parameters       | ❌ No                         | xUnit can create instance automatically |

Always use `IClassFixture<T>` when your test class constructor takes arguments that require shared setup or context.
