# Exception Handling & Logging in ASP.NET Core with CQRS

In simple Web API projects, middleware often handles both **logging** and **exception-to-response mapping**.  
However, in more advanced architectures (e.g., **CQRS** or distributed systems), logging is usually separated from the UI middleware:

- **Middleware responsibility**: Focuses only on mapping exceptions to consistent HTTP responses (e.g., `InvalidOperationException → 400`, `DbUpdateException → 500`).  
- **Centralized logging**: Exceptions are logged in a dedicated **cross-cutting pipeline** (e.g., MediatR behaviors, global filters, or infrastructure services). This ensures consistent logging across different entry points (API, background jobs, message handlers) instead of duplicating it in the UI layer.  
- **Observability tools**: Logs are often forwarded to centralized systems (Serilog sinks, ELK, Seq, Application Insights, etc.) where they can be correlated with commands, queries, and domain events.  

👉 **The key idea:** UI middleware communicates errors to clients, while logging is handled in a centralized and infrastructure-oriented way for observability and maintainability.

---

## Domain Layer

# Why the Domain Should Not Throw `System.Exception`

In a properly layered Domain-Driven Design (DDD) architecture, the **Domain layer** should never directly throw system-level exceptions such as `System.Exception`, `ArgumentException`, `NullReferenceException`, or database-specific exceptions like `SqlException`. Here’s why:

## 1. Domain Only Manages Business Rules

The Domain layer should focus purely on business logic:

- Is the inventory sufficient?
- Is the payment amount valid?

If the Domain throws exceptions like `ArgumentException` or `NullReferenceException`, it introduces **system-level concerns** into the domain, breaking its purity.

## 2. Greater Flexibility

By using **domain-specific exceptions** (e.g., `DomainException` and its subclasses), the Application layer can decide how to handle errors:

- Show a user-friendly message
- Log the error
- Retry the operation

If the Domain throws system exceptions, the Application layer has to catch them and map them to domain exceptions, which adds unnecessary complexity.

## 3. Alignment with DDD Principles

DDD advocates that the Domain should be **free of infrastructure dependencies**:

- System exceptions are often tied to the runtime or underlying infrastructure.
- The Domain should only use **domain-specific exceptions** to represent business rule violations and exceptional behaviors.

## Summary

✅ **Domain → only `DomainException` and its subclasses**  
❌ **Domain → should not throw `ArgumentException`, `SqlException`, or other system exceptions**  

**Application / Infrastructure layers** are responsible for interacting with the system and mapping infrastructure errors to appropriate behaviors.

```csharp
// -------------------------------
// DOMAIN LAYER
// -------------------------------
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

    public class InvalidAccountIdException : DomainException
    {
        public InvalidAccountIdException(string accountId)
            : base($"Invalid account ID: {accountId}.") { }
    }

    public class InvalidTargetAccountException : DomainException
    {
        public InvalidTargetAccountException()
            : base("Target account cannot be null.") { }
    }

    public class AccountNotFoundException : DomainException
    {
        public AccountNotFoundException(string accountId)
            : base($"Account with ID '{accountId}' was not found.") { }
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
                throw new InvalidAccountIdException(id);

            if (initialBalance < 0)
                throw new InvalidAmountException(initialBalance);

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

        public void TransferTo(Account target, decimal amount)
        {
            if (target == null)
                throw new InvalidTargetAccountException();

            Withdraw(amount);
            target.Deposit(amount);
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
        public Guid Id { get; private set; }
        public string FromAccountId { get; private set; }
        public string ToAccountId { get; private set; }
        public decimal Amount { get; private set; }
        public DateTime CreatedAt { get; private set; }

        private Transaction(string fromAccountId, string toAccountId, decimal amount)
        {
            FromAccountId = fromAccountId;
            ToAccountId = toAccountId;
            Amount = amount;
            CreatedAt = DateTime.UtcNow;
            Id = Guid.NewGuid();
        }

        public static Transaction Create(string fromAccountId, string toAccountId, decimal amount)
        {
            if (string.IsNullOrWhiteSpace(fromAccountId))
                throw new InvalidAccountIdException(fromAccountId);

            if (string.IsNullOrWhiteSpace(toAccountId))
                throw new InvalidAccountIdException(toAccountId);

            if (amount <= 0)
                throw new InvalidAmountException(amount);

            return new Transaction(fromAccountId, toAccountId, amount);
        }
    }
}
```
## Repository Layer

# 📌 Exceptions

## DomainException
- Intrinsically related to domain logic.
- Usually created and managed in the **Domain Layer**, not in the Repository.
- For example, if data is read from the DB that violates domain rules (e.g., a negative balance that should never exist in the domain), the Repository should simply fetch the data. The Domain Layer then validates it, and if there's an issue, it throws a `DomainException`.
- Therefore, the Repository does **not directly throw DomainExceptions**; it returns "raw" data and lets the Domain decide whether it’s valid.

## SystemException
- Originates from infrastructure concerns, such as:
  - `SqlException`
  - `DbUpdateException`
  - `TimeoutException`
- Repositories usually **do not catch these**, except for:
  - Logging, or
  - Wrapping them into an **Infrastructure-layer Exception** before letting them bubble up.

## 📌 Common Behavior
- The Repository is considered part of the **Infrastructure Layer**.
- It should **not be involved in domain logic**.
- System-related errors (DB, Network, API) bubble up to the **Application Layer**, which decides how to handle them (retry, fallback, user message).
- Domain-related errors belong in the **Domain Layer**.

Consider the following code sample:

```csharp
public async Task<Account> GetAccountAsync(string id)
{
    var account = await _dbContext.Accounts.FindAsync(id);
    if (account == null)
        throw new KeyNotFoundException($"Account {id} not found.");
    return account;
}
```

📌 Is this correct?

From a technical perspective, it works fine.

However, from a DDD and layering perspective, there are a few points:

Exception type
KeyNotFoundException is considered a system/infrastructure exception (not a domain exception).
These kinds of exceptions are more suitable for infrastructure or .NET collections, 
rather than expressing domain concepts like "Account not found".

Proper place to indicate "Account does not exist"
It is better if the repository simply returns the data (even null).
Then, in the Application Layer or Domain Service, you decide if it should throw, 
for example, an AccountNotFoundException when null.

Better alternatives
You can create a custom domain exception like:

```csharp
public class AccountNotFoundException : DomainException
{
    public AccountNotFoundException(string accountId) 
        : base($"Account with id {accountId} not found.") {}
}
```
Or even better, instead of throwing an exception, you can use the Result Pattern or Option/Maybe:

```csharp
public async Task<Account?> GetAccountAsync(string id)
{
    return await _dbContext.Accounts.FindAsync(id);
}
```
Then higher up (for example, in the Application Layer) you decide how to handle null.

According to **DDD and layered architecture** principles, the version below is **more correct and standard**:

```csharp
public async Task<Account?> GetAccountAsync(string id)
{
    return await _dbContext.Accounts.FindAsync(id);
}
```

### Why?

1. **Repository is only responsible for data access**

   - It should not decide what it means if the data is missing or which exception to throw.
   - If the Repository itself throws `AccountNotFoundException`, it places part of the **domain rules** into the Infrastructure layer, which goes against DDD principles.

2. **Domain/Application Layer is responsible for domain logic and errors**

   - In the Application Layer or Domain Service, you can check if the result is `null` and, if needed, throw a domain exception or define specific behavior.
   - Example:

```csharp
var account = await _accountRepository.GetAccountAsync(id);
if (account == null)
{
    throw new AccountNotFoundException(id); // Domain decides here
}
```

3. **More flexibility**
   - Later, if you want to use a Result/Option pattern or handle nullable data differently, the first version is easier to adapt.

### Summary

- Repository → only data (`Account?`)
- Application/Domain → rules, validation, exceptions



```csharp
// -------------------------------
// REPOSITORY LAYER
// -------------------------------
using BankingApi.Domain;
using Microsoft.EntityFrameworkCore;

namespace BankingApi.Infrastructure
{
    public interface IAccountRepository
    {
        Task<Account?> GetAccountAsync(string id);
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

        public async Task<Account?> GetAccountAsync(string id)
        {
            // Only returns data; does not throw domain exceptions
            return await _dbContext.Accounts.FindAsync(id);
        }

        public async Task UpdateBalanceAsync(Account account)
        {
            // EF tracks changes automatically
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
// -------------------------------
// APPLICATION / SERVICE LAYER
// -------------------------------
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
        /// 5. Each retry starts a new transaction, and if all retries fail, the exception is rethrown.
        /// </summary>
        public async Task<Unit> Handle(TransferCommand request, CancellationToken cancellationToken)
        {
            if (request.Amount <= 0)
                throw new InvalidAmountException(request.Amount);

            int retries = 3;

            for (int attempt = 1; attempt <= retries; attempt++)
            {
                try
                {
                    // Begin database transaction
                    // Each retry starts a new transaction.
                    // If a DbUpdateException occurs, the previous transaction is automatically rolled back (thanks to await using).
                    // After the last attempt, if it still fails, the exception is re-thrown.

                    await using var transactionScope = await _dbContext.Database.BeginTransactionAsync(cancellationToken);

                    // Load accounts
                    var fromAccount = await _repository.GetAccountAsync(request.FromAccountId);
                    if (fromAccount == null)
                        throw new AccountNotFoundException(request.FromAccountId);

                    var toAccount = await _repository.GetAccountAsync(request.ToAccountId);
                    if (toAccount == null)
                        throw new AccountNotFoundException(request.ToAccountId);

                    // Apply domain rules
                    fromAccount.Withdraw(request.Amount);
                    toAccount.Deposit(request.Amount);

                    // Persist changes in repository
                    await _repository.UpdateBalanceAsync(fromAccount);
                    await _repository.UpdateBalanceAsync(toAccount);

                    // Create transaction using domain factory method
                    var transaction = Transaction.Create(request.FromAccountId, request.ToAccountId, request.Amount);
                    await _repository.SaveTransactionAsync(transaction);

                    // Commit transaction
                    await transactionScope.CommitAsync(cancellationToken);
                    return Unit.Value;
                }
                catch (DbUpdateException) when (attempt < retries)
                {
                    // Retry with simple backoff
                    await Task.Delay(500 * attempt, cancellationToken);
                }
            }

            // If all retries fail, throw exception
            throw new DbUpdateException("Failed to save transaction after retries.", (Exception?)null);
        }
    }
}
```

## MediatR Pipeline Behavior (Centralized Logging)

Instead of logging exceptions separately in each part of the application (API, UI, background jobs, etc.), we define a **centralized pipeline** for logging.

This pipeline can be implemented in one of the following ways:

- **MediatR behaviors**: All requests (Requests/Commands/Queries) pass through a behavior before reaching their handler. Logging and exception handling can be placed there.  
- **Global filters** (e.g., in ASP.NET Core): Filters that are applied to all controllers, capturing exceptions in one place and logging them consistently.  
- **Infrastructure services / middleware**: A layer in the infrastructure where all requests (from APIs, jobs, or message handlers) pass through, allowing centralized exception handling and logging.  

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
                // Log domain exceptions as warnings
                _logger.LogWarning(ex, "Domain exception occurred");
                await WriteErrorResponseAsync(context, HttpStatusCode.BadRequest, ex.Message);
            }
            catch (Exception ex)
            {
                // Log unexpected exceptions as errors
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
