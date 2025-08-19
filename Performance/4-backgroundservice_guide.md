# Understanding `IHostedService`, `BackgroundService`, and Scoped Services in ASP.NET Core

Background services in **ASP.NET Core** are a powerful way to run tasks outside of the HTTP request pipeline. In this guide, we'll go step by step through the concepts of `IHostedService`, `BackgroundService`, scoped services, and best practices. At the end, you’ll find a **complete runnable sample project**.

---

## 1. `IHostedService`

`IHostedService` is an interface for defining long-running background tasks.

```csharp
public interface IHostedService
{
    Task StartAsync(CancellationToken cancellationToken);
    Task StopAsync(CancellationToken cancellationToken);
}
```

- **StartAsync** → Runs when the application starts.
- **StopAsync** → Runs when the application is shutting down.

You need to implement both methods yourself.

---

## 2. `BackgroundService`

`BackgroundService` is an abstract class that already implements `IHostedService` and makes life easier. You only need to override `ExecuteAsync`:

```csharp
public abstract class BackgroundService : IHostedService, IDisposable
{
    protected abstract Task ExecuteAsync(CancellationToken stoppingToken);
}
```

Example:

```csharp
public class MyBackgroundWorker : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            Console.WriteLine("Background task running...");
            await Task.Delay(1000, stoppingToken);
        }
    }
}
```

Register in `Program.cs`:

```csharp
builder.Services.AddHostedService<MyBackgroundWorker>();
```

---

## 3. Task.Delay with CancellationToken

Instead of `Thread.Sleep`, always use:

```csharp
await Task.Delay(1000, stoppingToken);
```

✅ This allows the task to stop **immediately** when the application shuts down.

---

## 4. Exception Handling and Logging

If exceptions are not caught, the background task might silently stop. Always log them:

```csharp
protected override async Task ExecuteAsync(CancellationToken stoppingToken)
{
    try
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                DoWork();
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error in background task");
            }

            await Task.Delay(1000, stoppingToken);
        }
    }
    catch (OperationCanceledException)
    {
        // Expected when service stops
    }
}
```

---

## 5. Overriding `StartAsync` and `StopAsync`

You normally don't need to override these, unless you want extra behavior before/after execution.

```csharp
public override async Task StartAsync(CancellationToken cancellationToken)
{
    _logger.LogInformation("Service starting...");
    await base.StartAsync(cancellationToken);
}

public override async Task StopAsync(CancellationToken cancellationToken)
{
    _logger.LogInformation("Service stopping...");
    await base.StopAsync(cancellationToken);
}
```

Always call `base.StartAsync`/`base.StopAsync` to preserve default behavior.

---

## 6. Scoped Services inside `BackgroundService`

### ❌ Wrong Way: Inject Scoped Service Directly

```csharp
public class WrongWorker : BackgroundService
{
    private readonly MyDbContext _dbContext; // ❌ DbContext is Scoped

    public WrongWorker(MyDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            var count = await _dbContext.Users.CountAsync(); // ❌ Runtime error
            await Task.Delay(1000, stoppingToken);
        }
    }
}
```

This fails because `BackgroundService` is **Singleton** and cannot hold a **Scoped** service.

---

### ✅ Correct Way: Use `IServiceScopeFactory`

```csharp
public class CorrectWorker : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;
    private readonly ILogger<CorrectWorker> _logger;

    public CorrectWorker(IServiceScopeFactory scopeFactory, ILogger<CorrectWorker> logger)
    {
        _scopeFactory = scopeFactory;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                using (var scope = _scopeFactory.CreateScope())
                {
                    var dbContext = scope.ServiceProvider.GetRequiredService<MyDbContext>();
                    var count = await dbContext.Users.CountAsync();
                    _logger.LogInformation("User count: {Count}", count);
                }
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Error in background service");
            }

            await Task.Delay(1000, stoppingToken);
        }
    }
}
```

Here, every loop creates a **new scope**, which is similar to a **new HTTP request** lifecycle. Scoped services (like `DbContext`) live only inside that scope and are disposed afterwards.

---

## 7. Service Lifetimes Recap

- **Singleton** → one instance for the entire app.
- **Scoped** → one instance per request/scope (e.g., per HTTP request or per `CreateScope()`).
- **Transient** → a new instance every time it’s requested.

| Lifetime   | Needs scope in BackgroundService? |
|------------|----------------------------------|
| Singleton  | No (safe to inject directly)     |
| Scoped     | Yes (use `IServiceScopeFactory`) |
| Transient  | No, but can also be created inside a scope |

---

## 8. Scoped Example: `IMessageService`

```csharp
builder.Services.AddScoped<IMessageService, MessageService>();
```

Using inside a worker:

```csharp
public class MessageWorker : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;

    public MessageWorker(IServiceScopeFactory scopeFactory)
    {
        _scopeFactory = scopeFactory;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            using (var scope = _scopeFactory.CreateScope())
            {
                var service = scope.ServiceProvider.GetRequiredService<IMessageService>();
                service.SendMessage("Hello from BackgroundService!");
            }

            await Task.Delay(1000, stoppingToken);
        }
    }
}
```

Every loop creates a new scope → like a new HTTP request.

---

## ✅ Full Sample Project Structure

Here’s a minimal runnable example you can put in GitHub.

### Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

// Register services
builder.Services.AddScoped<IMessageService, MessageService>();
builder.Services.AddHostedService<MessageWorker>();

var app = builder.Build();

app.MapGet("/", () => "Hello World!");

app.Run();
```

### IMessageService.cs

```csharp
public interface IMessageService
{
    void SendMessage(string message);
}
```

### MessageService.cs

```csharp
public class MessageService : IMessageService
{
    private readonly ILogger<MessageService> _logger;

    public MessageService(ILogger<MessageService> logger)
    {
        _logger = logger;
    }

    public void SendMessage(string message)
    {
        _logger.LogInformation("[MessageService] {Message}", message);
    }
}
```

### MessageWorker.cs

```csharp
public class MessageWorker : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;
    private readonly ILogger<MessageWorker> _logger;

    public MessageWorker(IServiceScopeFactory scopeFactory, ILogger<MessageWorker> logger)
    {
        _scopeFactory = scopeFactory;
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            using (var scope = _scopeFactory.CreateScope())
            {
                var service = scope.ServiceProvider.GetRequiredService<IMessageService>();
                service.SendMessage("Hello from BackgroundService!");
            }

            await Task.Delay(2000, stoppingToken);
        }
    }
}
```

---

## ✅ Summary / Checklist

- Use `BackgroundService` for simpler implementation of long-running tasks.
- Always use `Task.Delay(..., stoppingToken)` instead of `Thread.Sleep`.
- Handle exceptions with logging; don’t let the loop silently crash.
- Override `StartAsync`/`StopAsync` only when extra setup/cleanup is needed.
- Scoped services (e.g., DbContext) must be resolved inside a scope using `IServiceScopeFactory`.
- Each `CreateScope()` is like a new request lifecycle.
- Singleton and Transient services can be injected directly, Scoped cannot.

---

