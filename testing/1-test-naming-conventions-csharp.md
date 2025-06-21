# Test Naming Conventions in C#

Good naming in tests is just as important as naming in production code. Clear and consistent test names help teams working on long-term projects maintain readability, reduce onboarding time, and improve code quality.

## Why Use a Naming Convention?

Before choosing a naming convention, it’s helpful to ask:
- Why do we need consistent test names?
- What purpose should a test name serve?

Test names should:
- Express a specific requirement or behavior
- Include the input or state being tested
- Include the expected result or outcome
- Be readable and follow a known pattern

---

## Popular Naming Styles in C#

### 1. `MethodName_StateUnderTest_ExpectedBehavior`

**Example:**  
`IsAdult_AgeLessThan18_ReturnsFalse()`

✅ Clear structure  
❌ Needs renaming if method name changes

---

### 2. `Should_ExpectedBehavior_When_StateUnderTest`

**Example:**  
`Should_ReturnFalse_When_AgeIsLessThan18()`

✅ More readable / behavior-oriented  
❌ Can become long

---

### 3. `MethodName_Should_ExpectedBehavior_When_StateUnderTest`

**Example:**  
`IsAdult_Should_ReturnFalse_When_AgeIsLessThan18()`

✅ Useful in larger projects or when multiple methods are involved
❌ Slightly longer method names

---

### 4. `Given_Preconditions_When_State_Then_ExpectedResult` (BDD style)

**Example:**  
`Given_UserIsAuthenticated_When_InvalidAccountUsed_Then_TransactionFails()`

✅ Behavior-driven clarity  
❌ Verbose, may feel bloated for unit tests

---

### 5. `testFeatureBeingTested`

**Example:**  
`testIsNotAnAdultIfAgeLessThan18()`

✅ Simple  
❌ `test` prefix is redundant, unclear structure

---

## Recommendation

There are many valid naming conventions — the **important thing is to pick one and be consistent** across the codebase and team. In C#, the most common and maintainable styles are:

- `MethodName_StateUnderTest_ExpectedBehavior`  
- `Should_ExpectedBehavior_When_StateUnderTest`
- `MethodName_Should_ExpectedBehavior_When_StateUnderTest`

  | Pattern                                  | Best For                 | Notes                                 |
|------------------------------------------|---------------------------|----------------------------------------|
| `Should_ExpectedBehavior_When_State`     | Simpler, smaller projects | Emphasizes behavior                    |
| `MethodName_Should_ExpectedBehavior_When_State` | Larger projects, teams     | Emphasizes traceability to the method |

Both are valid. Choose the one that fits your team, project size, and codebase clarity needs.

Choose based on your team's preference and project style (e.g., unit tests vs. BDD).

---

## Example in xUnit (C#)

```csharp
public class PersonTests
{
    [Fact]
    public void IsAdult_AgeLessThan18_ReturnsFalse()
    {
        var person = new Person { Age = 15 };
        var result = person.IsAdult();
        Assert.False(result);
    }

    [Fact]
    public void Should_ReturnFalse_When_AgeIsLessThan18()
    {
        var person = new Person { Age = 15 };
        var result = person.IsAdult();
        Assert.False(result);
    }

    [Fact]
    public void IsAdult_Should_ReturnFalse_When_AgeIsLessThan18()
    {
        var person = new Person { Age = 15 };
        var result = person.IsAdult();
        Assert.False(result);
    }
}
