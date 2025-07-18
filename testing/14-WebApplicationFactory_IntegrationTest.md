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

```
