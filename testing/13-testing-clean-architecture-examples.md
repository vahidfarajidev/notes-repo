# 🧪 Testing in Clean Architecture

In Clean Architecture, responsibilities are separated into distinct layers, and the **type of test** you write depends on which layer you're testing.

---

## 🧱 1. Domain Layer — Unit Tests

- 💡 Purpose: Test pure business logic.
- ✅ Should not depend on any infrastructure (e.g., database, web, etc.)

### Example: `OrderTests.cs`

```csharp
public class Order
{
    public decimal TotalPrice { get; private set; }
    public decimal Discount { get; private set; }

    public void ApplyDiscount(decimal discount)
    {
        if (discount < 0 || discount > TotalPrice)
            throw new ArgumentException("Invalid discount");

        Discount = discount;
    }
}
```

#### ✅ Unit Test:

```csharp
public class OrderTests
{
    [Fact]
    public void ApplyDiscount_Should_SetDiscount_When_ValidAmount()
    {
        var order = new Order();
        order.ApplyDiscount(10);
        Assert.Equal(10, order.Discount);
    }
}
```

---

## 🧱 2. Application Layer — Integration or Unit

- 💡 Purpose: Coordinates use cases.
- May depend on domain, repositories, services.

You can write **unit tests with mocks**, or go **integration** if you want real DB/API.

### Example: `UserService.cs`

```csharp
public class UserService
{
    private readonly IUserRepository _repo;

    public UserService(IUserRepository repo)
    {
        _repo = repo;
    }

    public string GetWelcomeMessage(int userId)
    {
        var name = _repo.GetUserName(userId);
        return $"Welcome, {name}";
    }
}
```

#### ✅ Unit Test (with mock):

```csharp
[Fact]
public void GetWelcomeMessage_Should_ReturnMessage_When_UserExists()
{
    var repo = Substitute.For<IUserRepository>();
    repo.GetUserName(1).Returns("Ali");

    var service = new UserService(repo);
    var result = service.GetWelcomeMessage(1);

    result.Should().Be("Welcome, Ali");
}
```

---

## 🧱 3. Infrastructure Layer — Integration Tests

- 💡 Purpose: Actual file system, database, etc.
- ❌ Don’t write unit tests here — integration only.

### Example: `EfUserRepositoryTests.cs`

```csharp
[Fact]
public async Task GetUserName_Should_ReturnCorrectName_When_UserExists()
{
    using var context = new AppDbContext(_options);
    context.Users.Add(new User { Id = 1, Name = "Sara" });
    await context.SaveChangesAsync();

    var repo = new EfUserRepository(context);
    var name = repo.GetUserName(1);

    Assert.Equal("Sara", name);
}
```

---

## 🧱 4. API Layer — End-to-End / Integration Tests

- 💡 Purpose: Ensure endpoints work with real pipeline.
- Use **TestServer/WebApplicationFactory** in ASP.NET Core.

```csharp
[Fact]
public async Task POST_Login_Should_ReturnToken_When_ValidCredentials()
{
    var client = _factory.CreateClient();
    var response = await client.PostAsJsonAsync("/api/login", new { username = "admin", password = "123" });

    response.EnsureSuccessStatusCode();
    var token = await response.Content.ReadAsStringAsync();
    token.Should().NotBeNullOrEmpty();
}
```

---

## 🧠 Summary Table

| Layer           | Recommended Test       | Mock?        | Touch DB?  |
|----------------|------------------------|--------------|------------|
| Domain         | ✅ Unit Test            | ❌ No        | ❌ No       |
| Application    | ✅ Unit / 🔁 Integration| ✅ Yes (unit)| 🔁 Maybe    |
| Infrastructure | ❌ Unit / ✅ Integration| ❌ No        | ✅ Yes      |
| API            | ✅ End-to-End Test      | ❌ No        | ✅ Yes      |

---



---

## 🧪 Does Integration Test Actually Save to a Database?

Yes — but it's **not a real database**.

### 📌 Example:
```csharp
[Fact]
public async Task GetUserName_Should_ReturnCorrectName_When_UserExists()
{
    using var context = new AppDbContext(_options);
    context.Users.Add(new User { Id = 1, Name = "Sara" });
    await context.SaveChangesAsync();

    var repo = new EfUserRepository(context);
    var name = repo.GetUserName(1);

    Assert.Equal("Sara", name);
}
```

### ✅ What's happening here?
- A new `AppDbContext` is created
- A user is added
- `SaveChangesAsync` is called

It looks like real database interaction — but it uses **In-Memory Database** via EF Core.

### 🔧 Setup for In-Memory:
```csharp
var options = new DbContextOptionsBuilder<AppDbContext>()
    .UseInMemoryDatabase("TestDb")
    .Options;
```

### 🧠 Summary:

| Aspect | Answer |
|--------|--------|
| Does it really save data? | ✅ Yes, in memory only |
| Does it need a physical DB? | ❌ No |
| Is this an Integration Test? | ✅ Yes |
| Is this a Unit Test? | ❌ No, because it depends on EF Core |

This approach helps isolate your infrastructure logic **without the overhead of a real database**.
