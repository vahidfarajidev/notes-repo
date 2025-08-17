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
    // -------------------------------
    // Base Domain Exception
    // -------------------------------
    /// <summary>
    /// Base class for all domain-related exceptions.
    /// Domain exceptions represent violations of business rules.
    /// </summary>
    public abstract class DomainException : Exception
    {
        protected DomainException(string message) : base(message) { }
    }

    // -------------------------------
    // Specific Domain Exceptions
    // -------------------------------
    public class InsufficientFundsException : DomainException
    {
        public InsufficientFundsException(string accountId, decimal balance, decimal attempted)
            : base($"Account {accountId} has insufficient funds. Balance={balance}, Attempted={attempted}") { }
    }

    public class InvalidAmountException : DomainException
    {
        public InvalidAmountException(decimal amount)
            : base($"Invalid amount: {amount}. Must be greater than zero.") { }
    }

    // -------------------------------
    // Account Aggregate
    // -------------------------------
    /// <summary>
    /// Represents a bank account and enforces domain rules for withdrawals and deposits.
    /// Domain enforces all input validation and business logic.
    /// </summary>
    public class Account
    {
        public string Id { get; private set; }
        public decimal Balance { get; private set; }

        public Account(string id, decimal initialBalance = 0)
        {
            if (string.IsNullOrWhiteSpace(id))
                throw new ArgumentException("Account ID cannot be empty.", nameof(id));
            if (initialBalance < 0)
                throw new ArgumentException("Initial balance cannot be negative.", nameof(initialBalance));

            Id = id;
            Balance = initialBalance;
        }

        public void Withdraw(decimal amount)
        {
            if (amount <= 0)
                throw new InvalidAmountException(amount);

            if (Balance < amount)
                throw new InsufficientFundsException(Id, Balance, amount);

            Balance -= amount;
        }

        public void Deposit(decimal amount)
        {
            if (amount <= 0)
                throw new InvalidAmountException(amount);

            Balance += amount;
        }
    }

    // -------------------------------
    // Transaction Entity
    // -------------------------------
    /// <summary>
    /// Represents a money transfer transaction between two accounts.
    /// All domain validation should occur before creating a transaction instance.
    /// </summary>
    public class Transaction
    {
        public Guid Id { get; set; } = Guid.NewGuid();
        public string FromAccountId { get; set; }
        public string ToAccountId { get; set; }
        public decimal Amount { get; set; }
        public DateTime CreatedAt { get; set; } = DateTime.UtcNow;

        public Transaction(string fromAccountId, string toAccountId, decimal amount)
        {
            if (string.IsNullOrWhiteSpace(fromAccountId))
                throw new ArgumentException("FromAccountId cannot be empty.", nameof(fromAccountId));
            if (string.IsNullOrWhiteSpace(toAccountId))
                throw new ArgumentException("ToAccountId cannot be empty.", nameof(toAccountId));
            if (amount <= 0)
                throw new ArgumentException("Transaction amount must be positive.", nameof(amount));

            FromAccountId = fromAccountId;
            ToAccountId = toAccountId;
            Amount = amount;
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
        Task UpdateBalanceAsync(Account account);
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
                throw new KeyNotFoundException($"Account {id} not found.");
            return account;
        }

        public async Task UpdateBalanceAsync(Account account)
        {
            // EF will track changes to the account entity automatically
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
        /// 3. Retry for transient DbUpdateException including all SaveChangesAsync operations.
        /// 4. Logging delegated to pipeline/middleware.
        /// </summary>
        public async Task<Unit> Handle(TransferCommand request, CancellationToken cancellationToken)
        {
            int retries = 3;
            for (int attempt = 1; attempt <= retries; attempt++)
            {
                try
                {
                    await using var transactionScope = await _dbContext.Database.BeginTransactionAsync(cancellationToken);

                    var fromAccount = await _repository.GetAccountAsync(request.FromAccountId);
                    var toAccount = await _repository.GetAccountAsync(request.ToAccountId);

                    fromAccount.Withdraw(request.Amount);
                    toAccount.Deposit(request.Amount);

                    await _repository.UpdateBalanceAsync(fromAccount);
                    await _repository.UpdateBalanceAsync(toAccount);

                    var transaction = new Transaction(request.FromAccountId, request.ToAccountId, request.Amount);
                    await _repository.SaveTransactionAsync(transaction);

                    await transactionScope.CommitAsync(cancellationToken);
                    return Unit.Value;
                }
                catch (DbUpdateException) when (attempt < retries)
                {
                    await Task.Delay(500, cancellationToken); // simple backoff
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
using Microsoft.Extensions.Logging;

namespace BankingApi.Application
{
    /// <summary>
    /// Pipeline behavior for logging all MediatR requests and exceptions.
    /// Logs before execution and on exceptions.
    /// </summary>
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
            // Log before executing the handler
            _logger.LogInformation(
                "Handling {RequestType} with data: {@Request}",
                typeof(TRequest).Name,
                request
            );

            try
            {
                var response = await next();
                
                // Optional: Log after successful execution
                _logger.LogInformation(
                    "Handled {RequestType} successfully with response: {@Response}",
                    typeof(TRequest).Name,
                    response
                );

                return response;
            }
            catch (Exception ex)
            {
                // Log exception with full context
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
using BankingApi.Domain;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;

namespace BankingApi.Api.Middleware
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

        public async Task InvokeAsync(HttpContext context)
        {
            try
            {
                await _next(context);
            }
            catch (DomainException ex)
            {
                _logger.LogWarning(ex, "Domain exception occurred");
                await WriteErrorResponseAsync(context, HttpStatusCode.BadRequest, ex.Message);
            }
            catch (KeyNotFoundException ex)
            {
                _logger.LogWarning(ex, "Resource not found");
                await WriteErrorResponseAsync(context, HttpStatusCode.NotFound, ex.Message);
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Unhandled exception occurred");
                await WriteErrorResponseAsync(context, HttpStatusCode.InternalServerError, "An unexpected error occurred");
            }
        }

        private static async Task WriteErrorResponseAsync(HttpContext context, HttpStatusCode statusCode, string message)
        {
            context.Response.ContentType = "application/json";
            context.Response.StatusCode = (int)statusCode;

            var errorResponse = new
            {
                error = message,
                status = context.Response.StatusCode
            };

            await context.Response.WriteAsync(JsonSerializer.Serialize(errorResponse));
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
