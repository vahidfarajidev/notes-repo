# Four Phases of Unit Testing

In unit testing, it's best practice to structure your tests following the **Four-Phase Test** pattern. This pattern improves **readability**, **maintainability**, and **clarity** of test cases.

---

## ✅ The Four Phases

| Phase     | Purpose                                       |
|-----------|-----------------------------------------------|
| Setup     | Prepare the test context and inputs           |
| Exercise  | Invoke the method or behavior under test      |
| Verify    | Assert that the result is as expected         |
| Teardown  | Clean up any resources or state (optional)    |

---

## 1. 🛠️ Setup (Arrange)

Prepare everything needed for the test.

```csharp
var calculator = new Calculator();
```

---

## 2. ▶️ Exercise (Act)

Execute the code that you want to test.

```csharp
var result = calculator.Add(2, 3);
```

---

## 3. ✅ Verify (Assert)

Check that the outcome is what you expect.

```csharp
Assert.Equal(5, result);
```

---

## 4. ♻️ Teardown (Cleanup)

Clean up any resources, files, or connections used in the test.  
In xUnit, this is commonly done using `Dispose()`:

```csharp
public class CalculatorTests : IDisposable
{
    public void Dispose()
    {
        // Cleanup code here (e.g., delete temp files, close DB)
    }
}
```

---

## 🧾 Summary Example

```csharp
[Fact]
public void Test_Add_ShouldReturnCorrectResult()
{
    // Arrange
    var calc = new Calculator();

    // Act
    var result = calc.Add(2, 3);

    // Assert
    Assert.Equal(5, result);

    // Teardown: handled by Dispose if needed
}
```

---

Using this structure helps ensure your tests are **clean, consistent, and easy to understand.**
