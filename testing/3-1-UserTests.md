# Unit Test Explanation for `User.Create` Validation

This document explains the purpose and behavior of the following unit test, written using the xUnit testing framework in C#.

---

## 🧪 Test Code

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

---

## 🧩 Explanation of Attributes

### `[Theory]`
- Indicates that this test method is a parameterized test.
- It will run **once for each** `[InlineData]` provided.

### `[InlineData(...)]`
Each `InlineData` provides a different input to the test method:

| Value         | Description                        |
|---------------|------------------------------------|
| `null`        | No value provided (null reference) |
| `""`          | An empty string                    |
| `"   "`       | A whitespace-only string           |

Thus, this test will execute **three times**, once with each invalid name input.

---

## 🧪 Test Logic Breakdown

```csharp
Action act = () => User.Create(invalidName, "ali@example.com");
```

- An `Action` delegate is created to call `User.Create` with the `invalidName` and a sample email.
- This allows us to **assert that an exception is thrown** when the input is invalid.

---

## ✅ Assertion

```csharp
act.Should().Throw<ArgumentException>()
   .WithMessage("Name is required");
```

This line uses **FluentAssertions** to verify that:

- An `ArgumentException` is thrown.
- The error message matches `"Name is required"`.

---

## 🎯 Purpose of This Test

To ensure that the `User.Create` method:

- Validates the `name` parameter properly.
- Throws an appropriate exception with a clear message when the name is `null`, empty, or whitespace.

---

## 📌 Notes

Make sure that the `User.Create` method internally checks for invalid names and throws an `ArgumentException` with the message `"Name is required"` accordingly.
