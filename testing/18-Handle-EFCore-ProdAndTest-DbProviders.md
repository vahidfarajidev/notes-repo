
# Using Multiple EF Core Database Providers for Production and Integration Testing

This guide explains how to configure your ASP.NET Core application to use **SQL Server** as the production database provider and **InMemory** database for integration tests, avoiding conflicts caused by registering multiple EF Core providers in the same service provider.

---

## Problem

Entity Framework Core throws the following error if multiple database providers are registered in the same service provider:

```
System.InvalidOperationException: Services for database providers 'Microsoft.EntityFrameworkCore.SqlServer', 'Microsoft.EntityFrameworkCore.InMemory' have been registered in the service provider. Only a single database provider can be registered in a service provider.
```

---

## Solution Overview

- Use **SQL Server** provider normally in production.
- In the integration test project, **replace** the `DbContext` registration with an **InMemory** provider.
- Remove the original `DbContext` registrations in the test `WebApplicationFactory`.
- Optionally, set a special environment name (e.g., `"IntegrationTest"`) to distinguish test environment.

---

## Example

### 1. Production `InfrastructureBootstrapper`

```csharp
public static void RegisterDependency(IServiceCollection services, IConfiguration configuration)
{
    services.AddScoped<IUserRepository, EfUserRepository>();
    services.AddScoped<IUnitOfWork, EfUnitOfWork>();

    // Register SQL Server provider for production
    services.AddDbContext<AppDbContext>(options =>
        options.UseSqlServer(configuration.GetConnectionString("DefaultConnection")));
}
```

---

### 2. Integration Test `CustomWebApplicationFactory`

```csharp
public class CustomWebApplicationFactory<TStartup> : WebApplicationFactory<TStartup>
    where TStartup : class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        // Set custom environment for integration tests
        builder.UseEnvironment("IntegrationTest");

        builder.ConfigureServices(services =>
        {
            // Remove existing DbContext registrations to avoid conflicts
            var descriptors = services
                .Where(d => d.ServiceType == typeof(DbContextOptions<AppDbContext>) ||
                            d.ServiceType == typeof(AppDbContext))
                .ToList();

            foreach (var descriptor in descriptors)
            {
                services.Remove(descriptor);
            }

            // Register InMemory database provider for tests
            services.AddDbContext<AppDbContext>(options =>
            {
                options.UseInMemoryDatabase("TestDb");
            });

            // Optional: Initialize database before tests run
            var sp = services.BuildServiceProvider();
            using var scope = sp.CreateScope();
            var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();
            db.Database.EnsureDeleted();
            db.Database.EnsureCreated();
        });
    }
}
```

---

## Notes

- **Why remove registrations?**  
  Because EF Core does not allow multiple database providers registered at the same time in the same service provider.

- **Environment variable `"IntegrationTest"`**  
  Setting a special environment helps conditionally configure services or apply special behaviors in your app during testing.

- **Database Initialization**  
  Calling `EnsureDeleted` and `EnsureCreated` guarantees a clean, isolated test database for each test run.

---

## Summary

This approach cleanly separates the database providers between production and integration tests, eliminating `InvalidOperationException` and allowing you to test against an in-memory database while running SQL Server in production.

---

If you have any questions or issues, feel free to ask!
