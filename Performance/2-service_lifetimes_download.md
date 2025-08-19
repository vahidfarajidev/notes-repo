# Service Lifetimes in ASP.NET Core

This Markdown document explains **Singleton**, **Scoped**, and **Transient** service lifetimes in ASP.NET Core using a Q&A format with code examples.

---

## ❓ What is a Singleton service?
A **Singleton** service is created **once** during the application’s lifetime and reused everywhere.

**Example:**
```csharp
builder.Services.AddSingleton<IMessageService, MessageService>();
```

**Behavior:**
- Within one request: same instance for all injections.
- Across requests: same instance persists.

**Use cases:** Logging, caching, configuration.

---

## ❓ What is a Scoped service?
A **Scoped** service is created **once per HTTP request**.

**Example:**
```csharp
builder.Services.AddScoped<IMessageService, MessageService>();
```

**Behavior:**
- Within one request: same instance for all injections.
- Across requests: a new instance is created for each request.

**Use cases:** Entity Framework DbContext, request-specific business logic.

---

## ❓ What is a Transient service?
A **Transient** service is created **every time it is requested**.

**Example:**
```csharp
builder.Services.AddTransient<IMessageService, MessageService>();
```

**Behavior:**
- Within one request: each injection gets a new instance.
- Across requests: each injection is a new instance.

**Use cases:** Lightweight, stateless helpers or validators.

---

## ❓ Example Controller
```csharp
[ApiController]
[Route("[controller]")]
public class TestController : ControllerBase
{
    private readonly IMessageService _service1;
    private readonly IMessageService _service2;

    public TestController(IMessageService service1, IMessageService service2)
    {
        _service1 = service1;
        _service2 = service2;
    }

    [HttpGet]
    public string Get()
    {
        return _service1.GetMessage() + " | " + _service2.GetMessage();
    }
}
```

- Singleton: same Guid for both services across all requests.
- Scoped: same Guid within one request, new Guid for the next request.
- Transient: different Guid even within the same request.

---

## ❓ Injecting multiple instances
```csharp
public TestController(IMessageService s1, IMessageService s2, IMessageService s3, IMessageService s4, IMessageService s5) { }
```

- Singleton: all 5 instances are the same across requests.
- Scoped: all 5 instances are the same within a request, new instances in the next request.
- Transient: all 5 instances are different even within the same request.

---

## 📊 Quick Comparison
| Lifetime     | Within One Request | Across Requests |
|--------------|------------------|----------------|
| Singleton    | Same instance     | Same instance   |
| Scoped       | Same instance     | Different       |
| Transient    | Different instances | Different     |

---

✅ **Summary:**
- Singleton: for global services.
- Scoped: for per-request services.
- Transient: for lightweight, stateless services.

