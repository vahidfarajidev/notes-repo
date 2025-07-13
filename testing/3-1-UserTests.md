# Understanding `[Theory]` vs `[Fact]` in xUnit with a Practical Example

This document explains the difference between `[Theory]` and `[Fact]` attributes in the xUnit testing framework in C#, using a real-world validation example.

---

## 🔍 `[Theory]` vs `[Fact]`

| Attribute | Purpose                                               | Example Use Case                        |
|----------|--------------------------------------------------------|------------------------------------------|
| `[Fact]` | Used for a single, simple test with no input parameters | Checking a fixed behavior                |
| `[Theory]` | Used for parameterized tests that run with multiple data sets | Validating input with different values |

---

## 🧪 Example: User Name Validation

The following test ensures the `User.Create` method throws an exception when an invalid name is provided.

```csharp
[Theory]
[InlineData(null)]
[InlineData("")]
[InlineData("   ")]
public void Create_Should_ThrowException_When_NameIsInvalid(string invalidName)
{
    // Act
    Action act = () => User.Create(invalidName, "ali@example.com");

    // Assert
    act.Should().Throw<ArgumentException>()
       .WithMessage("Name is required");
}
```

### 🔄 Why Use `[Theory]` Here?

Using `[Theory]` allows this test method to run **multiple times**, each time with a different value from the `[InlineData]` attributes.

This is more efficient and expressive than writing three separate `[Fact]` tests for each invalid input.

---

## 🧩 Explanation of `[InlineData]`

Each `[InlineData]` provides a specific invalid input:

- `null` → No value provided
- `""` → Empty string
- `"   "` → String with only whitespace

This simulates common real-world input validation cases.

---

## ✅ Assertion Logic

```csharp
act.Should().Throw<ArgumentException>()
   .WithMessage("Name is required");
```

This uses **FluentAssertions** to ensure:

- An `ArgumentException` is thrown.
- The exception message matches `"Name is required"`.

---

## 🎯 Conclusion

This test effectively demonstrates how `[Theory]` and `[InlineData]` are used to test **multiple input scenarios** efficiently. Use `[Fact]` for single-scenario tests, and `[Theory]` for data-driven testing.
