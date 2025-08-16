# Exception Handling in ASP.NET Core Web API

Building robust and maintainable Web APIs requires a well-thought-out strategy for **exception handling**. In banking or financial systems, error handling becomes even more critical because failures directly impact end users. This article demonstrates how to structure exception handling across multiple layers in an **ASP.NET Core Web API** project, including domain exceptions, repository operations, service logic, and global middleware.

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
 │    ├── Account.cs
 │    ├── Transaction.cs
 │    └── Exceptions/
 │         ├── InsufficientFundsException.cs
 │         ├── InvalidTransactionAmountException.cs
 │         └── AccountNotFoundException.cs
 ├── Middleware/
 │    └── ExceptionHandlingMiddleware.cs
 └── Program.cs
```

---

## 1️⃣ Domain Layer

### Domain Models
```csharp
namespace BankingApi.Domain
{
    public class Transaction
    {
        public Guid Id { get; set; } = Guid.NewGuid();
        public string FromAccountId { get; set; }
        public string ToAccountId { get; set; }
        public decimal Amount { get; set; }
        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    }

    public class Account
    {
        public string Id { get; set; }
        public decimal Balance { get; private set; }

        public void Withdraw(decimal amount)
        {
            if (amount <= 0)
                throw new InvalidTransactionAmountException(amount);

            if (Balance < amount)
                throw new InsufficientFundsException(Id, amount);

            Balance -= amount;
        }

        public void Deposit(decimal amount)
        {
            if (amount <= 0)
                throw new InvalidTransactionAmountException(amount);

            Balance += amount;
        }
    }
}
```

### Domain Exceptions
```csharp
namespace BankingApi.Domain.Exceptions
{
    public class InsufficientFundsException : Exception
    {
        public InsufficientFundsException(string accountId, decimal amount)
            : base($"Insufficient funds in account {accountId} for withdrawal of {amount}.") { }
    }

    public class InvalidTransactionAmountException : Exception
    {
        public InvalidTransactionAmountException(decimal amount)
            : base($"Invalid transaction amount: {amount}. Must be positive.") { }
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

```csharp
using BankingApi.Domain;
using BankingApi.Domain.Exceptions;
using Microsoft.EntityFrameworkCore;

namespace BankingApi.Repositories
{
    public class BankingDbContext : DbContext
    {
        public DbSet<Account> Accounts { get; set; }
        public DbSet<Transaction> Transactions { get; set; }

        public BankingDbContext(DbContextOptions<BankingDbContext> options)
            : base(options) { }
    }

    public class AccountRepository
    {
        private readonly BankingDbContext _db;

        public AccountRepository(BankingDbContext db)
        {
            _db = db;
        }

        public async Task<Account> GetAccountAsync(string accountId)
        {
            var account = await _db.Accounts.FindAsync(accountId);
            if (account == null)
                throw new AccountNotFoundException(accountId);
            return account;
        }

        public async Task UpdateBalanceAsync(string accountId, decimal newBalance)
        {
            var account = await _db.Accounts.FindAsync(accountId);
            if (account == null)
                throw new AccountNotFoundException(accountId);

            account.Balance = newBalance;
            await _db.SaveChangesAsync();
        }

        public async Task SaveTransactionAsync(Transaction tx)
        {
            _db.Transactions.Add(tx);
            await _db.SaveChangesAsync();
        }
    }
}
```

---

## 3️⃣ Service Layer with Retry + Logging

```csharp
using BankingApi.Domain;
using BankingApi.Domain.Exceptions;
using BankingApi.Repositories;
using Microsoft.EntityFrameworkCore;

namespace BankingApi.Services
{
    public class AccountService
    {
        private readonly AccountRepository _repository;
        private readonly ILogger<AccountService> _logger;

        public AccountService(AccountRepository repository, ILogger<AccountService> logger)
        {
            _repository = repository;
            _logger = logger;
        }

        public async Task TransferAsync(string fromAccountId, string toAccountId, decimal amount)
        {
            var fromAccount = await _repository.GetAccountAsync(fromAccountId);
            var toAccount = await _repository.GetAccountAsync(toAccountId);

            fromAccount.Withdraw(amount);
            toAccount.Deposit(amount);

            await _repository.UpdateBalanceAsync(fromAccount.Id, fromAccount.Balance);
            await _repository.UpdateBalanceAsync(toAccount.Id, toAccount.Balance);

            var transaction = new Transaction
            {
                FromAccountId = fromAccountId,
                ToAccountId = toAccountId,
                Amount = amount
            };

            int retries = 3;
            for (int attempt = 1; attempt <= retries; attempt++)
            {
                try
                {
                    await _repository.SaveTransactionAsync(transaction);
                    _logger.LogInformation("Transaction saved successfully. TxId={TxId}", transaction.Id);
                    break;
                }
                catch (DbUpdateException ex) when (attempt < retries)
                {
                    _logger.LogWarning(ex, "Database error saving transaction. Attempt {Attempt}/{Retries}. TxId={TxId}", attempt, retries, transaction.Id);
                    await Task.Delay(500);
                }
            }
        }
    }
}
```

---

## 4️⃣ Controller Layer

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

        [HttpPost("transfer")]
        public async Task<IActionResult> Transfer([FromQuery] string fromAccount, [FromQuery] string toAccount, [FromQuery] decimal amount)
        {
            await _service.TransferAsync(fromAccount, toAccount, amount);
            return Ok(new { message = "Transfer successful." });
        }
    }
}
```

---

## 5️⃣ Exception Handling Middleware

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
            catch (InsufficientFundsException ex)
            {
                _logger.LogWarning(ex, "Business rule violation: insufficient funds.");
                await HandleExceptionAsync(context, HttpStatusCode.BadRequest, ex.Message);
            }
            catch (InvalidTransactionAmountException ex)
            {
                _logger.LogWarning(ex, "Business rule violation: invalid transaction amount.");
                await HandleExceptionAsync(context, HttpStatusCode.BadRequest, ex.Message);
            }
            catch (AccountNotFoundException ex)
            {
                _logger.LogWarning(ex, "Account not found.");
                await HandleExceptionAsync(context, HttpStatusCode.NotFound, ex.Message);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Unhandled exception.");
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

builder.Services.AddDbContext<BankingDbContext>(opt => opt.UseInMemoryDatabase("BankingDb"));
builder.Services.AddScoped<AccountRepository>();
builder.Services.AddScoped<AccountService>();
builder.Services.AddControllers();

var app = builder.Build();
app.UseExceptionHandling();
app.MapControllers();
app.Run();
```

---

## ✅ Summary

- **Domain Layer**: Business rules throw domain-specific exceptions.  
- **Repository Layer**: Handles CRUD; throws domain exceptions for missing entities; system exceptions bubble up.  
- **Service Layer**: Orchestrates domain methods, handles retries, logs critical info.  
- **Controller Layer**: Keeps endpoints clean; relies on middleware.  
- **Middleware**: Centralizes logging and maps exceptions to proper HTTP responses.  

This approach ensures maintainability, consistent error handling, and clear semantics for API consumers.

## 💡 Exception “Bubble Up”

When we say an exception **“bubbles up”**, it means the exception is **not caught in the current layer** and is propagated to the higher layers.  

In our example:  

- If the **Repository** encounters a system exception such as `SqlException` or `DbUpdateException`, it **does not catch it**.  
- The exception **bubbles up** to the **Service Layer**.  
- If the Service Layer also does not catch it (which is usually the case for system exceptions), the exception continues to propagate until it reaches the **Middleware**.  
- The **Middleware** catches the exception, logs it, and returns an appropriate HTTP response.  

So, **“bubble up”** means the exception passes through the current layer until it finds a handler (like the Middleware).  

In simple terms: the exception moves up from the lowest layer until someone is responsible for handling it, without the Repository interfering with it.

