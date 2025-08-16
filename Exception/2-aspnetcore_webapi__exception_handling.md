# Exception Handling in ASP.NET Core Web API

Building robust and maintainable Web APIs requires a well-thought-out strategy for **exception handling**. In banking or financial systems, error handling becomes even more critical because failures directly impact end users. This article demonstrates how to structure exception handling across multiple layers in an **ASP.NET Core Web API** project.

---

## 📂 Project Structure

```
BankingApi/
 ├── Controllers/
 │    └── AccountsController.cs
 ├── Services/
 │    └── AccountService.cs
 ├── Repositories/
 │    └── AccountRepository.cs
 ├── Domain/
 │    └── Exceptions/
 │         ├── InsufficientFundsException.cs
 │         └── AccountNotFoundException.cs
 ├── Middleware/
 │    └── ExceptionHandlingMiddleware.cs
 └── Program.cs
```

---

## 1️⃣ Domain Exceptions

Domain exceptions represent **business errors** such as insufficient funds or a missing account. These exceptions should be explicit and meaningful.

```csharp
namespace BankingApi.Domain.Exceptions
{
    public class InsufficientFundsException : Exception
    {
        public InsufficientFundsException(string accountId, decimal amount)
            : base($"Insufficient funds in account {accountId} for withdrawal of {amount}.") { }
    }

    public class AccountNotFoundException : Exception
    {
        public AccountNotFoundException(string accountId)
            : base($"Account with ID {accountId} not found.") { }
    }
}
```

---

## 2️⃣ Repository Layer

The Repository layer deals with **data access**. It may throw both **Domain Exceptions** (when a business-related issue is detected) and **System Exceptions** (SQL errors, timeouts, etc.).

```csharp
using BankingApi.Domain.Exceptions;
using Microsoft.Data.SqlClient;
using Microsoft.EntityFrameworkCore;

namespace BankingApi.Repositories
{
    public class BankingDbContext : DbContext
    {
        public DbSet<Account> Accounts { get; set; }

        public BankingDbContext(DbContextOptions<BankingDbContext> options)
            : base(options) { }
    }

    public class Account
    {
        public string Id { get; set; }
        public decimal Balance { get; set; }
    }

    public class AccountRepository
    {
        private readonly BankingDbContext _db;

        public AccountRepository(BankingDbContext db)
        {
            _db = db;
        }

        public async Task<decimal> GetBalanceAsync(string accountId)
        {
            try
            {
                var account = await _db.Accounts.FindAsync(accountId);

                if (account == null)
                    throw new AccountNotFoundException(accountId);

                return account.Balance;
            }
            catch (SqlException ex)
            {
                throw new Exception("Database error occurred while fetching balance.", ex);
            }
            catch (TimeoutException ex)
            {
                throw new Exception("Database request timed out.", ex);
            }
        }

        public async Task UpdateBalanceAsync(string accountId, decimal newBalance)
        {
            try
            {
                var account = await _db.Accounts.FindAsync(accountId);

                if (account == null)
                    throw new AccountNotFoundException(accountId);

                account.Balance = newBalance;
                await _db.SaveChangesAsync();
            }
            catch (DbUpdateConcurrencyException ex)
            {
                throw new Exception("Concurrency conflict while updating account balance.", ex);
            }
            catch (SqlException ex)
            {
                throw new Exception("Database error occurred while updating balance.", ex);
            }
        }
    }
}
```

---

## 3️⃣ Service Layer

The Service layer contains **business logic**. It does not swallow exceptions but simply enforces domain rules (e.g., insufficient funds).

```csharp
using BankingApi.Domain.Exceptions;
using BankingApi.Repositories;

namespace BankingApi.Services
{
    public class AccountService
    {
        private readonly AccountRepository _repository;

        public AccountService(AccountRepository repository)
        {
            _repository = repository;
        }

        public async Task WithdrawAsync(string accountId, decimal amount)
        {
            var balance = await _repository.GetBalanceAsync(accountId);

            if (balance < amount)
                throw new InsufficientFundsException(accountId, amount);

            await _repository.UpdateBalanceAsync(accountId, balance - amount);
        }
    }
}
```

---

## 4️⃣ Controller Layer

The Controller exposes endpoints but does not need explicit `try/catch` because global middleware will handle exceptions.

```csharp
using BankingApi.Services;
using Microsoft.AspNetCore.Mvc;

namespace BankingApi.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class AccountsController : ControllerBase
    {
        private readonly AccountService _service;

        public AccountsController(AccountService service)
        {
            _service = service;
        }

        [HttpPost("{accountId}/withdraw")]
        public async Task<IActionResult> Withdraw(string accountId, [FromQuery] decimal amount)
        {
            await _service.WithdrawAsync(accountId, amount);
            return Ok(new { message = "Withdrawal successful." });
        }
    }
}
```

---

## 5️⃣ Exception Handling Middleware

The Middleware centralizes exception handling, logging, and mapping to proper HTTP responses.

```csharp
using BankingApi.Domain.Exceptions;
using System.Net;

namespace BankingApi.Middleware
{
    public class ExceptionHandlingMiddleware
    {
        private readonly RequestDelegate _next;
        private readonly ILogger<ExceptionHandlingMiddleware> _logger;

        public ExceptionHandlingMiddleware(RequestDelegate next, ILogger<ExceptionHandlingMiddleware> logger)
        {
            _next = next;
            _logger = logger;
        }

        public async Task Invoke(HttpContext context)
        {
            try
            {
                await _next(context);
            }
            catch (AccountNotFoundException ex)
            {
                _logger.LogWarning(ex, "Account not found");
                await HandleExceptionAsync(context, HttpStatusCode.NotFound, ex.Message);
            }
            catch (InsufficientFundsException ex)
            {
                _logger.LogWarning(ex, "Insufficient funds");
                await HandleExceptionAsync(context, HttpStatusCode.BadRequest, ex.Message);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Unhandled exception");
                await HandleExceptionAsync(context, HttpStatusCode.InternalServerError, "An unexpected error occurred.");
            }
        }

        private static Task HandleExceptionAsync(HttpContext context, HttpStatusCode statusCode, string message)
        {
            context.Response.ContentType = "application/json";
            context.Response.StatusCode = (int)statusCode;
            return context.Response.WriteAsJsonAsync(new { error = message });
        }
    }

    public static class ExceptionHandlingMiddlewareExtensions
    {
        public static IApplicationBuilder UseExceptionHandling(this IApplicationBuilder builder)
        {
            return builder.UseMiddleware<ExceptionHandlingMiddleware>();
        }
    }
}
```

---

## 6️⃣ Program.cs

```csharp
using BankingApi.Repositories;
using BankingApi.Services;
using BankingApi.Middleware;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

// Register DbContext (e.g., InMemory or SQL Server)
builder.Services.AddDbContext<BankingDbContext>(opt => opt.UseInMemoryDatabase("BankingDb"));

// Dependency injection
builder.Services.AddScoped<AccountRepository>();
builder.Services.AddScoped<AccountService>();
builder.Services.AddControllers();

var app = builder.Build();

app.UseExceptionHandling(); // Our custom middleware
app.MapControllers();
app.Run();
```

---

## ✅ Summary

- **Repository Layer**: May throw both **Domain Exceptions** (business-related) and **System Exceptions** (DB/IO errors).  
- **Service Layer**: Applies business rules and throws domain-specific exceptions.  
- **Controller Layer**: Keeps endpoints clean; no `try/catch` needed.  
- **Middleware**: Centralizes logging and exception-to-HTTP mapping.  

This separation ensures maintainability, consistency, and clear error semantics for clients consuming your API.
