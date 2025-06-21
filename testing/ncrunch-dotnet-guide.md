
# ⚡ Quick Guide: NCrunch for .NET

**NCrunch** is a powerful continuous testing tool for .NET that runs your tests automatically as you write code, giving instant feedback on test results and code coverage.

---

## ✅ What is NCrunch?

NCrunch is a Visual Studio extension that:

- Runs tests **automatically in the background**
- Highlights **test status** and **code coverage** live in the editor
- Executes tests in **parallel** across CPU cores
- Re-runs only tests affected by recent code changes

---

## 📎 Example

### 🔹 Production Code
```csharp
public class Calculator
{
    public int Add(int a, int b) => a + b;
}
```

### 🔹 Unit Test (xUnit + FluentAssertions)
```csharp
public class CalculatorTests
{
    [Fact]
    public void Add_TwoPositiveIntegers_ReturnsCorrectSum()
    {
        var calc = new Calculator();
        var result = calc.Add(2, 3);
        result.Should().Be(5); // NCrunch shows pass/fail here
    }
}
```

---
NCrunch places colored indicators not only next to the test method name, but also beside each line inside the test method — showing whether that specific line was executed and if the related test passed, failed, or remained untested.

## 🧠 Visual Indicators

| **Indicator**            | **Meaning**                                             |
|--------------------------|----------------------------------------------------------|
| 🟢 **Green**              | Line executed by a test that passed                      |
| 🔴 **Red**                | Line executed by a test that failed                      |
| ⚪ **White / Light Gray** | Line not executed by any test (no coverage)              |
| 🟡 **Yellow**             | Test pending / queued (not yet executed)                 |

---

## 📦 Install

- Website: [https://www.ncrunch.net](https://www.ncrunch.net)
- Visual Studio extension (paid, trial available)

---

## 🧾 Summary

NCrunch gives **instant, visual feedback** while coding and testing in .NET, making it ideal for TDD and fast feedback loops. A must-have for serious test-driven developers.
