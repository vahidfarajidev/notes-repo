# Handling Multiple EF Core Providers for Production and Integration Testing

In real-world .NET projects, we typically use SQL Server for production and `InMemory` database for integration testing. However, using both EF Core providers at the same time in one service provider causes the following error:

```
System.InvalidOperationException: Services for database providers 'Microsoft.EntityFrameworkCore.SqlServer', 'Microsoft.EntityFrameworkCore.InMemory' have been registered in the service provider. Only a single database provider can be registered in a service provider.
```

To resolve this, follow these steps:

---

## ✅ Step 1: Conditional Registration Based on Environment

Modify your `InfrastructureBootStraper` to only register SQL Server in non-test environments:

```csharp
public static void RegisterDependency(IServiceCollection services, IConfiguration configuration, IWebHostEnvironment env)
{
    services.AddScoped<IUserRepository, EfUserRepository>();
    services.AddScoped<IUnitOfWork, EfUnitOfWork>();

    // ✅ Avoid registering SQL Server in test environment
    if (!env.EnvironmentName.Equals("IntegrationTest", StringComparison.OrdinalIgnoreCase))
    {
        services.AddDbContext<AppDbContext>(options =>
            options.UseSqlServer(configuration.GetConnectionString("DefaultConnection")));
    }
}
```

Make sure `IWebHostEnvironment env` is passed when calling this method (e.g., from `Program.cs` or `Startup.cs`).

---

## ✅ Step 2: Setup InMemory DB in Tests

Use a custom `WebApplicationFactory<T>` in your integration tests to override the database provider:

```csharp
public class CustomWebApplicationFactory<TStartup> : WebApplicationFactory<TStartup> where TStartup : class
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        // ✅ Set a specific environment to detect in DI logic
        builder.UseEnvironment("IntegrationTest");

        builder.ConfigureServices(services =>
        {
            // ✅ Remove all registrations related to AppDbContext
            var dbContextDescriptors = services
                .Where(d => d.ServiceType == typeof(DbContextOptions<AppDbContext>) ||
                            d.ServiceType == typeof(AppDbContext))
                .ToList();

            foreach (var descriptor in dbContextDescriptors)
            {
                services.Remove(descriptor);
            }

            // ✅ Register InMemory provider
            services.AddDbContext<AppDbContext>(options =>
            {
                options.UseInMemoryDatabase("TestDb");
            });

            // ✅ Optionally create and seed the database
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

## 💡 Summary

- ❗ EF Core does not allow multiple providers (like `UseSqlServer` and `UseInMemoryDatabase`) in the same DI container.
- ✅ Use `UseEnvironment("IntegrationTest")` in your `CustomWebApplicationFactory`.
- ✅ Add environment checks in your main DB registration logic (`InfrastructureBootStraper`) to avoid registering `UseSqlServer` during tests.
- ✅ Override the database in tests using `UseInMemoryDatabase` and remove any existing `DbContext` registrations.

This setup cleanly separates test and production configurations without clashing EF Core providers.
