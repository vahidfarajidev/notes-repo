# Testing ASP.NET Core Web APIs and Controllers: A Practical Guide

When developing Web APIs in ASP.NET Core, ensuring correctness and robustness through automated testing is crucial. There are **four main approaches** to testing Controllers and Web APIs, each serving a unique purpose.

---

## ✅ 1. Unit Testing

### 🎯 Goal
Test the controller logic in isolation, without involving any ASP.NET Core infrastructure like routing, middleware, or dependency injection.

### 🛠️ Tools
- Mocking libraries like **Moq**, **NSubstitute**, or **FakeItEasy**
- Testing frameworks: **xUnit**, **NUnit**, **MSTest**

### 🧪 What it tests
- Does the controller invoke the correct method from the service?
- Does it return the expected response type (`Ok`, `BadRequest`, etc.)?

### ✅ Example
```csharp
[Fact]
public void Add_Should_Call_Service_And_Return_Ok()
{
    // Arrange
    var mockService = Substitute.For<IUserService>();
    var controller = new UsersController(mockService);
    var dto = new AddUserDto { Name = "Test", Email = "test@example.com" };

    // Act
    var result = controller.Add(dto);

    // Assert
    result.Should().BeOfType<OkResult>();
    mockService.Received(1).AddUser(dto);
}

---

## ✅ 2. Integration Testing

### 🎯 Goal
Test the integration of multiple layers: **Routing**, **Model Binding**, **Controller logic**, **Dependency Injection**, etc., typically using an **in-memory host**.

### 🛠️ Tools
- `WebApplicationFactory<TEntryPoint>`
- `HttpClient`
- In-memory server

### 🧪 What it tests
- Can the endpoint be reached (`/api/users`)?
- Is the JSON request body bound to the DTO?
- Are services correctly injected?

### ✅ Example
```csharp
var response = await client.PostAsJsonAsync("/api/users", dto);
response.StatusCode.Should().Be(HttpStatusCode.OK);
```

> ⚠️ **Database Note**: Integration tests typically use **In-Memory** databases like:
```csharp
services.AddDbContext<AppDbContext>(options =>
{
    options.UseInMemoryDatabase("TestDb");
});
```

---

## ✅ 3. Functional Testing

### 🎯 Goal
Validate **controller behavior** under specific business scenarios such as invalid input, mismatched route/body IDs, authentication failures, etc.

### 🛠️ Tools
- Same as Integration Test (in-memory server, `HttpClient`)
- Focused test scenarios

### 🧪 What it tests
- Does the controller return `400 BadRequest` if `id` in route != `id` in body?
- Does it handle `ModelState` errors properly?
- Does it return `401 Unauthorized` for unauthenticated users?

### ✅ Example
```csharp
var response = await client.PutAsJsonAsync("/api/users/1", dtoWithDifferentId);
response.StatusCode.Should().Be(HttpStatusCode.BadRequest);
```

---

## ✅ 4. End-to-End (E2E) Testing

### 🎯 Goal
Test the system from the **user’s perspective**—from UI (or HTTP client) through controllers, services, and the **real database**.

### 🛠️ Tools
- **Playwright** or **Selenium** for UI-driven tests
- `HttpClient`, `TestServer`
- Real databases: **SQL Server**, **PostgreSQL**, **SQLite**
- Often run in **Docker** containers

### 🧪 What it tests
- Does the full request pipeline work?
- Is data correctly stored and retrieved from the real database?
- Are user interactions successful?

---

## ⚠️ Managing Test Data in E2E with Real Databases

When using a **real database**, test data must be carefully managed. Here are common strategies:

### 1. Use Transaction + Rollback
Start a transaction before the test, and roll it back afterward.
```csharp
var transaction = dbContext.Database.BeginTransaction();
// run test
transaction.Rollback();
```
> ✅ Keeps DB clean  
> ⚠️ May fail with multiple parallel connections (like EF Core + `HttpClient`)

---

### 2. Cleanup After Test
Manually delete inserted data in `[TearDown]`, `Dispose`, or `[TestCleanup]`.

```csharp
public async Task DisposeAsync()
{
    await _dbContext.Users.ExecuteDeleteAsync(); // EF Core 7+
}
```

---

### 3. Use a Dedicated Test Database
Create a test-only database like `MyApp_Test`.  
Reset and seed it before running test suites.

> ✅ Isolated  
> ⚠️ Requires DB provisioning and maintenance

---

### 4. Run Databases via Docker
Spin up a throwaway database container (PostgreSQL, SQL Server, etc.) using Docker.

```bash
docker run --rm -e POSTGRES_PASSWORD=pass -p 5432:5432 postgres
```

> ✅ Clean and disposable  
> ⚠️ Slower setup, more DevOps effort

---

## 🧠 Summary Table

| Test Type        | Speed      | Scope            | Framework Dependent | Uses Database     | Handles Business Logic |
|------------------|------------|------------------|----------------------|-------------------|-------------------------|
| Unit Test        | ⚡ Fast     | Just controller  | ❌ No                | ❌ No             | ❌ Limited              |
| Integration Test | ⚡ Fast     | Controller + DI  | ✅ Yes               | ✅ In-Memory      | ❌ Basic only           |
| Functional Test  | ⚖️ Medium  | End-to-End logic | ✅ Yes               | ✅ In-Memory      | ✅ Yes                  |
| E2E Test         | 🐢 Slow     | Full system      | ✅ Yes               | ✅ Real DB/Docker | ✅ Full                 |

---

## ✅ Final Thoughts

Choosing the right testing approach depends on your **goals, speed, and accuracy** requirements:

- Use **Unit Tests** to validate controller logic quickly.
- Use **Integration Tests** to ensure routing, DI, and input binding work correctly.
- Use **Functional Tests** to verify business rules and edge cases.
- Use **E2E Tests** to simulate real user flows and confirm end-to-end correctness.

> For robust systems, **combine multiple test types** for confidence and coverage.
