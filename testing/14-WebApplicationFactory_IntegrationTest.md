# Why Tests Using WebApplicationFactory Are Considered Integration Tests

## ✅ What Is WebApplicationFactory?

`WebApplicationFactory<TEntryPoint>` is a helper class provided by `Microsoft.AspNetCore.Mvc.Testing`. It spins up your entire ASP.NET Core application **in-memory** using a `TestServer`.

It allows you to write tests that:

- Use the real middleware pipeline
- Hit real controllers via HTTP (not just method calls)
- Use actual routing and dependency injection
- Can use a real or in-memory database
- Behave like a real client interacting with the app

---

## 🔍 What Is an Integration Test?

An **Integration Test** checks how multiple parts of a system work together.

In contrast to unit tests (which isolate components and mock dependencies), integration tests:

- Do **not** mock the internals
- Cover the real execution pipeline
- Validate actual inputs and outputs across multiple layers (e.g., Controller + Middleware + Routing + Filters)

---

## ✅ Why WebApplicationFactory Tests Are Integration Tests

| Feature | Present in WebApplicationFactory Tests |
|--------|----------------------------------------|
| Middleware execution | ✅ |
| Routing and endpoint resolution | ✅ |
| Controller and action invocation | ✅ |
| Dependency Injection (DI) | ✅ |
| Real HTTP requests via HttpClient | ✅ |
| Custom headers, filters, etc. | ✅ |

➡️ All of these are components that work **together**, not in isolation. This is the core of what integration testing means.

---

## ❌ Why These Are Not Unit Tests

- The whole application runs (not just one isolated method)
- No mocking is used by default
- Tests rely on the full HTTP pipeline
- Often include config, startup logic, and global filters

---

## 📌 Summary

> Tests that use `WebApplicationFactory<T>` are classified as **Integration Tests** because they validate how multiple components (middleware, routing, controllers, DI, etc.) behave **together** in a near-real environment.

---

## 🧪 Bonus Tip

If you're using `WebApplicationFactory` with an in-memory EF Core database and real HTTP calls, you're getting **very close to end-to-end** (E2E) behavior — just without a browser.

---

## 📎 Note: Minimal Hosting and WebApplicationFactory

In .NET 6+ (including .NET 9), if you're using Minimal Hosting (`var builder = WebApplication.CreateBuilder(args);`), the `Program` class becomes **implicit** and cannot be found by `WebApplicationFactory<T>`.

✅ **Fix**: Add the following to the end of your `Program.cs` file:

```csharp
public partial class Program { } // for WebApplicationFactory


# Overriding Services in Integration Testing: `Program.cs` vs `CustomWebApplicationFactory`

In ASP.NET Core applications, dependency injection (DI) is configured in `Program.cs` for the actual application runtime. However, in integration testing scenarios, we often need to override these services—for example, to use an in-memory database instead of a real one.

This article explains the difference between defining services in `Program.cs` and overriding them in `CustomWebApplicationFactory`.

---

## 🎯 Purpose

| File | Purpose |
|------|---------|
| `Program.cs` | Configures services for the actual app (Development, Production) |
| `CustomWebApplicationFactory` | Overrides certain services for testing (e.g., swap SQL Server with InMemory DB) |

---

## ✅ What Goes in `Program.cs`

```csharp
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddScoped<IUserRepository, EfUserRepository>();
builder.Services.AddScoped<IUnitOfWork, EfUnitOfWork>();

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(configuration.GetConnectionString("Default")));
```

> These are the real services used in the live application.

---

## 🔁 What Happens in `CustomWebApplicationFactory`

```csharp
builder.ConfigureServices(services =>
{
    var descriptor = services.SingleOrDefault(
        d => d.ServiceType == typeof(DbContextOptions<AppDbContext>));
    if (descriptor != null)
        services.Remove(descriptor);

    services.AddDbContext<AppDbContext>(options =>
    {
        options.UseInMemoryDatabase("TestDb");
    });
});
```

> This ensures that during integration tests, a lightweight in-memory database is used.

---

## 💡 Why This Separation is Important

- Prevents production code from using test-only configurations.
- Keeps tests isolated and fast.
- Follows the single-responsibility principle: production code remains clean, while test configuration is handled separately.

---

## ✅ Summary

- Always define your actual services in `Program.cs`.
- Use `CustomWebApplicationFactory` to override them for test purposes.
- This makes your application clean, testable, and maintainable.
