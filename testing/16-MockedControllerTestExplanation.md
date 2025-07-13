# ✅ Understanding Controller Tests with Mocked Services

In this test:

```csharp
var controller = new UsersController(mockService);
var result = controller.Add(dto);
```

### ❓ Does `UsersController.Add()` really get executed?

✅ **Yes** — the method `Add()` inside `UsersController` is executed for real.

---

### ❌ What does *not* get executed?

This line inside the controller:

```csharp
_service.AddUser(dto);
```

- It’s just **called**, but **not really executed**, because `mockService` is a **substitute** (fake).
- It only tracks that it was **invoked**, but its **logic does not run**.

---

### 🔍 So what actually happens?

| Part                            | Executed? | Explanation                                                  |
|----------------------------------|-----------|--------------------------------------------------------------|
| `UsersController.Add(dto)`      | ✅ Yes    | The controller action runs completely.                      |
| `mockService.AddUser(dto)`      | ❌ No     | It’s a mock — the method is *called* but not executed.       |
| `Received().AddUser(dto)`       | 🔍 Check  | Only checks that the method was called — no real behavior.   |

---

### 💡 Example for clarity

**Controller Code:**
```csharp
[HttpPost]
public IActionResult Add(AddUserDto dto)
{
    _service.AddUser(dto); // Called but not executed if it's mocked
    return Ok(); // This line really runs
}
```

**Test Code:**
```csharp
var mockService = Substitute.For<IUserService>();
var controller = new UsersController(mockService);

var dto = new AddUserDto { Name = "Ali", Email = "ali@example.com" };
var result = controller.Add(dto);  // ← This method runs

result.Should().BeOfType<OkResult>();
mockService.Received().AddUser(dto);  // ← Only verifies it was called
```

---

If you want to write a test that **actually executes `AddUser()` logic** instead of mocking it (i.e., an **Integration Test**), just let me know — I can provide that too.
