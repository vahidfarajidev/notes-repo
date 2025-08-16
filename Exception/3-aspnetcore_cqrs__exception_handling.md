# Exception Handling & Logging in ASP.NET Core with CQRS

In simple Web API projects, middleware often handles both **logging** and **exception-to-response mapping**.  
However, in more advanced architectures (e.g., **CQRS** or distributed systems), logging is usually separated from the UI middleware:

- **Middleware responsibility**: Focuses only on mapping exceptions to consistent HTTP responses (e.g., `InvalidOperationException → 400`, `DbUpdateException → 500`).  
- **Centralized logging**: Exceptions are logged in a dedicated **cross-cutting pipeline** (e.g., MediatR behaviors, global filters, or infrastructure services). This ensures consistent logging across different entry points (API, background jobs, message handlers) instead of duplicating it in the UI layer.  
- **Observability tools**: Logs are often forwarded to centralized systems (Serilog sinks, ELK, Seq, Application Insights, etc.) where they can be correlated with commands, queries, and domain events.  

👉 **The key idea:** UI middleware communicates errors to clients, while logging is handled in a centralized and infrastructure-oriented way for observability and maintainability.

---

## Domain Layer

```csharp
namespace BankingApi.Domain
{
    // Custom domain exception for insufficient funds
    public class InsufficientFundsException : Exception
    {
        public InsufficientFundsException(string accountId, decimal balance, decimal amount)
            : base($"Account {accountId} has insufficient funds. Balance: {balance}, Attempted withdraw: {amount}")
        {
        }
    }

    public class Account
    {
        public string Id { get; set; }
        public decimal Balance { get; private set; }

        public void Withdraw(decimal amount)
        {
            if (amount <= 0)
                throw new ArgumentException("Amount must be positive.");

            if (Balance < amount)
                throw new InsufficientFundsException(Id, Balance, amount);

            Balance -= amount;
        }

        public void Deposit(decimal amount)
        {
            if (amount <= 0)
                throw new ArgumentException("Amount must be positive.");

            Balance += amount;
        }
    }
}
```
## Repository Layer

```csharp
using BankingApi.Domain;
using Microsoft.EntityFrameworkCore;

namespace BankingApi.Infrastructure
{
    public interface IAccountRepository
    {
        Task<Account> GetAccountAsync(string id);
        Task UpdateBalanceAsync(string id, decimal newBalance);
        Task SaveTransactionAsync(Transaction transaction);
    }

    public class AccountRepository : IAccountRepository
    {
        private readonly BankingDbContext _dbContext;

        public AccountRepository(BankingDbContext dbContext)
        {
            _dbContext = dbContext;
        }

        public async Task<Account> GetAccountAsync(string id)
        {
            var account = await _dbContext.Accounts.FindAsync(id);
            if (account == null)
                throw new KeyNotFoundException($"Account {id} not found");
            return account;
        }

        public async Task UpdateBalanceAsync(string id, decimal newBalance)
        {
            var account = await GetAccountAsync(id);
            account.Deposit(0); // force domain validation
            account.GetType().GetProperty("Balance")!.SetValue(account, newBalance);
            _dbContext.Update(account);
            await _dbContext.SaveChangesAsync();
        }

        public async Task SaveTransactionAsync(Transaction transaction)
        {
            await _dbContext.Transactions.AddAsync(transaction);
            await _dbContext.SaveChangesAsync();
        }
    }
}
```

## Service Layer (CQRS Command Handler)

```csharp
using BankingApi.Domain;
using BankingApi.Infrastructure;
using MediatR;
using Microsoft.EntityFrameworkCore;

namespace BankingApi.Application
{
    public class TransferCommand : IRequest
    {
        public string FromAccountId { get; init; }
        public string ToAccountId { get; init; }
        public decimal Amount { get; init; }
    }

    public class TransferCommandHandler : IRequestHandler<TransferCommand>
    {
        private readonly IAccountRepository _repository;
        private readonly BankingDbContext _dbContext;

        public TransferCommandHandler(IAccountRepository repository, BankingDbContext dbContext)
        {
            _repository = repository;
            _dbContext = dbContext;
        }

        /// <summary>
        /// Performs a money transfer between two accounts in an atomic and reliable manner.
        /// Key points:
        /// 1. Transactional consistency with BeginTransaction.
        /// 2. Domain rules enforced by Account entities.
        /// 3. Retry for transient DbUpdateException.
        /// 4. Logging delegated to pipeline (not here).
        /// </summary>
        public async Task<Unit> Handle(TransferCommand request, CancellationToken cancellationToken)
        {
            await using var transactionScope = await _dbContext.Database.BeginTransactionAsync(cancellationToken);

            var fromAccount = await _repository.GetAccountAsync(request.FromAccountId);
            var toAccount = await _repository.GetAccountAsync(request.ToAccountId);

            fromAccount.Withdraw(request.Amount);
            toAccount.Deposit(request.Amount);

            await _repository.UpdateBalanceAsync(fromAccount.Id, fromAccount.Balance);
            await _repository.UpdateBalanceAsync(toAccount.Id, toAccount.Balance);

            var transaction = new Transaction
            {
                FromAccountId = request.FromAccountId,
                ToAccountId = request.ToAccountId,
                Amount = request.Amount
            };

            int retries = 3;
            for (int attempt = 1; attempt <= retries; attempt++)
            {
                try
                {
                    await _repository.SaveTransactionAsync(transaction);
                    await transactionScope.CommitAsync(cancellationToken);
                    return Unit.Value;
                }
                catch (DbUpdateException) when (attempt < retries)
                {
                    await Task.Delay(500, cancellationToken);
                }
            }

            throw new DbUpdateException("Failed to save transaction after retries.", (Exception?)null);
        }
    }
}
```

## MediatR Pipeline Behavior (Centralized Logging)

```csharp
using MediatR;

namespace BankingApi.Application
{
    public class LoggingBehavior<TRequest, TResponse> : IPipelineBehavior<TRequest, TResponse>
    {
        private readonly ILogger<LoggingBehavior<TRequest, TResponse>> _logger;

        public LoggingBehavior(ILogger<LoggingBehavior<TRequest, TResponse>> logger)
        {
            _logger = logger;
        }

        public async Task<TResponse> Handle(
            TRequest request,
            RequestHandlerDelegate<TResponse> next,
            CancellationToken cancellationToken)
        {
            try
            {
                return await next();
            }
            catch (Exception ex)
            {
                _logger.LogError(
                    ex,
                    "Error handling {RequestType} with data: {@Request}",
                    typeof(TRequest).Name,
                    request
                );
                throw;
            }
        }
    }
}
```

## Middleware (Only Maps Exceptions → HTTP Responses)

```csharp
using System.Net;
using System.Text.Json;

namespace BankingApi.Api.Middleware
{
    public class ExceptionHandlingMiddleware
    {
        private readonly RequestDelegate _next;

        public ExceptionHandlingMiddleware(RequestDelegate next)
        {
            _next = next;
        }

        public async Task InvokeAsync(HttpContext context)
        {
            try
            {
                await _next(context);
            }
            catch (KeyNotFoundException ex)
            {
                context.Response.StatusCode = (int)HttpStatusCode.NotFound;
                await context.Response.WriteAsync(JsonSerializer.Serialize(new { error = ex.Message }));
            }
            catch (InsufficientFundsException ex)
            {
                context.Response.StatusCode = (int)HttpStatusCode.BadRequest;
                await context.Response.WriteAsync(JsonSerializer.Serialize(new { error = ex.Message }));
            }
            catch (Exception)
            {
                context.Response.StatusCode = (int)HttpStatusCode.InternalServerError;
                await context.Response.WriteAsync(JsonSerializer.Serialize(new { error = "An unexpected error occurred" }));
            }
        }
    }
}
```

Summary

- Domain Layer: Validates business rules with domain-specific exceptions.

- Repository Layer: CRUD only; system exceptions bubble up.

- Service Layer: Enforces transactional consistency, retry logic.

- Pipeline Behavior: Centralized logging across commands/queries.

- Middleware: Maps exceptions to HTTP responses only (no logging here).

This ensures clean separation of concerns, centralized logging, and reliable error handling in a CQRS-style ASP.NET Core application.
