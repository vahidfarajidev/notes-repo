# Teardown in Unit Testing (xUnit)

In unit testing, **Teardown** refers to the cleanup steps that occur **after each test runs**. It's used to reset the environment, release resources, or clean temporary data to ensure isolated and repeatable tests.

---

## 🧪 Why Teardown Is Important

- Frees up resources (e.g., files, memory, DB connections)
- Prevents tests from affecting each other
- Ensures reliable test outcomes
- Maintains a clean and predictable testing environment

---

## 🛠️ Teardown in xUnit

Unlike NUnit, xUnit does **not** use `[TearDown]` attributes. Instead, it uses the `IDisposable` interface to implement cleanup logic.

---

### ✅ Example: xUnit `IDisposable` Teardown

```csharp
public class MyTests : IDisposable
{
    public MyTests()
    {
        // Setup: runs before each test
        Console.WriteLine("Setup logic here");
    }

    [Fact]
    public void TestSomething()
    {
        Console.WriteLine("Running test...");
    }

    public void Dispose()
    {
        // Teardown: runs after each test
        Console.WriteLine("Cleanup logic here");
    }
}
```

---

### ✅ When `Dispose()` is called

- After **every test method**
- Even if the test **fails** or throws an exception
- It's the xUnit way of doing per-test cleanup

---

## 🧪 Teardown for Multiple Tests (Shared Context)

If you're sharing context between tests (using `IClassFixture<T>`), and want teardown after all tests in that class, implement `IDisposable` on the fixture class instead.

```csharp
public class DatabaseFixture : IDisposable
{
    public DatabaseFixture()
    {
        // Setup shared resource
    }

    public void Dispose()
    {
        // Teardown after all tests in class
    }
}
```

Then use it:

```csharp
public class MyDatabaseTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;

    public MyDatabaseTests(DatabaseFixture fixture)
    {
        _fixture = fixture;
    }

    [Fact]
    public void TestQuery()
    {
        // Use shared resource from fixture
    }
}
```

---

## 🧾 Summary

| Concept        | Description                               |
|----------------|-------------------------------------------|
| Teardown       | Cleanup logic after test runs             |
| xUnit method   | `Dispose()` via `IDisposable`             |
| Purpose        | Free resources, reset state, isolate tests|
| Shared cleanup | Use `IClassFixture<T>` and `IDisposable`  |

Using teardown properly ensures tests are **clean**, **independent**, and **reliable** — especially when dealing with real resources like databases or files.
