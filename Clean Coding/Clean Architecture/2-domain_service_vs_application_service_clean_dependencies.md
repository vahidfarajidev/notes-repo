# Domain Service vs Application Service in DDD

**Why Domain Services should not depend on Repositories (even interfaces)**

---

- **Domain Service = pure business rules** (stateless, deterministic, no I/O).
- **Application Service = orchestration** (fetch data via repositories, manage transactions, call domain).
- Keep the **Domain layer independent** of data access; pass *data-in* and get *decisions-out*.

---

## Layered Architecture at a Glance

```
[UI / API]
    ↓
[Application Layer]  ← coordinates use cases, transactions, retries
    ↓
[Domain Layer]       ← entities, value objects, domain services (pure rules)
    ↓
[Infrastructure]     ← EF Core, SQL, HTTP, queues (repository implementations)
```

**Rule of thumb:** Domain knows nothing about data access. Application knows both Domain and Repositories. Infrastructure knows how to persist domain objects.

---

## Roles & Responsibilities

### Domain Service

- Encapsulates **business rules** that don’t belong to a single entity/value object.
- Should be **pure**: given inputs → validate/compute → output or throw a **DomainException**.
- **No** repository/DB access; **no** I/O, **no** network calls.

### Application Service

- Orchestrates **use cases**: loads data via repositories, calls domain rules, persists outcomes, handles transactions/retries.
- Can perform **cross-aggregate** checks (with data from multiple repositories) *before* calling domain services.

### Repository (Interface vs Implementation)

- **Interfaces** (contracts) are referenced by the Application (and sometimes declared in Domain or a shared contracts assembly).
- **Implementations** live in Infrastructure (EF Core, Dapper, HTTP, etc.).

> Even if the repository **interface** is visible to the Domain, a Domain Service depending on it mixes responsibilities (rules + data access). Keep Domain Services pure; let Application fetch the data and pass it in.

---

## The Daily Transfer Limit Example

### Bad (Domain Service depends on repository)

```csharp
public class TransferDomainService
{
    private readonly ITransactionRepository _repository;
    private const decimal DailyLimit = 10000m;

    public TransferDomainService(ITransactionRepository repository)
    {
        _repository = repository;
    }

    public void ValidateDailyLimit(string fromAccountId, decimal amount)
    {
        var total = _repository.GetTotalTransfersToday(fromAccountId);
        if (total + amount > DailyLimit)
            throw new BusinessRuleViolationException("Daily transfer limit exceeded.");
    }
}
```

**Problems**: mixes business rule with data fetching; harder to test; violates separation of concerns.

### Good (Domain Service is pure)

```csharp
public class TransferDomainService
{
    private const decimal DailyLimit = 10000m;

    public void ValidateDailyLimit(decimal totalTransfersToday, decimal amount)
    {
        if (totalTransfersToday + amount > DailyLimit)
            throw new BusinessRuleViolationException("Daily transfer limit exceeded.");
    }
}
```

**Benefits**: pure business logic, easy to unit-test, reusable.

### Application Service orchestrates

```csharp
// CQRS-style handler
public class TransferCommandHandler : IRequestHandler<TransferCommand>
{
    private readonly IAccountRepository _repository;
    private readonly TransferDomainService _domainService;
    private readonly BankingDbContext _dbContext;

    public TransferCommandHandler(
        IAccountRepository repository,
        TransferDomainService domainService,
        BankingDbContext dbContext)
    {
        _repository = repository;
        _domainService = domainService;
        _dbContext = dbContext;
    }

    public async Task<Unit> Handle(TransferCommand request, CancellationToken ct)
    {
        if (request.Amount <= 0)
            throw new InvalidAmountException(request.Amount);

        // Cross-aggregate fact collected via repository (I/O in Application layer)
        var totalTransfersToday = await _repository.GetTotalTransfersTodayAsync(request.FromAccountId, DateTime.UtcNow.Date);

        // Pure rule in Domain Service (no I/O)
        _domainService.ValidateDailyLimit(totalTransfersToday, request.Amount);

        // Retry + transaction for reliability
        const int retries = 3;
        for (int attempt = 1; attempt <= retries; attempt++)
        {
            try
            {
                await using var tx = await _dbContext.Database.BeginTransactionAsync(ct);

                var from = await _repository.GetAccountAsync(request.FromAccountId)
                           ?? throw new AccountNotFoundException(request.FromAccountId);
                var to   = await _repository.GetAccountAsync(request.ToAccountId)
                           ?? throw new AccountNotFoundException(request.ToAccountId);

                from.Withdraw(request.Amount);
                to.Deposit(request.Amount);

                await _repository.UpdateBalanceAsync(from);
                await _repository.UpdateBalanceAsync(to);

                var transfer = Transaction.Create(request.FromAccountId, request.ToAccountId, request.Amount);
                await _repository.SaveTransactionAsync(transfer);

                await tx.CommitAsync(ct);
                return Unit.Value;
            }
            catch (DbUpdateException) when (attempt < retries)
            {
                await Task.Delay(500 * attempt, ct); // simple backoff
            }
        }

        throw new DbUpdateException("Failed to save transaction after retries.", (Exception?)null);
    }
}
```

---

## Repository Contract & Implementation

**Contract (used by Application):**

```csharp
public interface IAccountRepository
{
    Task<Account?> GetAccountAsync(string id);
    Task UpdateBalanceAsync(Account account);
    Task SaveTransactionAsync(Transaction transaction);
    Task<decimal> GetTotalTransfersTodayAsync(string accountId, DateTime? onDate = null);
}
```

**EF Core Implementation (Infrastructure):**

```csharp
public class AccountRepository : IAccountRepository
{
    private readonly BankingDbContext _dbContext;

    public AccountRepository(BankingDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public async Task<Account?> GetAccountAsync(string id)
        => await _dbContext.Accounts.FindAsync(id);

    public async Task UpdateBalanceAsync(Account account)
    {
        _dbContext.Update(account);
        await _dbContext.SaveChangesAsync();
    }

    public async Task SaveTransactionAsync(Transaction transaction)
    {
        await _dbContext.Transactions.AddAsync(transaction);
        await _dbContext.SaveChangesAsync();
    }

    public async Task<decimal> GetTotalTransfersTodayAsync(string accountId, DateTime? onDate = null)
    {
        var date = (onDate ?? DateTime.UtcNow.Date).Date;

        var total = await _dbContext.Transactions
            .Where(t => t.FromAccountId == accountId && t.CreatedAt >= date && t.CreatedAt < date.AddDays(1))
            .SumAsync(t => (decimal?)t.Amount) ?? 0m;

        return total;
    }
}
```

---

## Testing Benefits

- **Domain Service** is trivial to unit test: call `ValidateDailyLimit(total, amount)` and assert throws/does not throw.
- **Application Service** can be tested with mocked repositories and a real `TransferDomainService`.
- No need to mock EF Core inside Domain tests.

---

## Decision Checklist

Ask these before placing logic:

1. **Does it need I/O or repository access?**
   - Yes → Application Service.
   - No  → Domain (Entity/Value Object/Domain Service).
2. **Is it a business rule spanning multiple aggregates?**
   - Yes → Domain Service (pure), data supplied by Application.
3. **Does it coordinate steps, transactions, or retries?**
   - Yes → Application Service.

---

## Common Anti‑Patterns

- **Repository inside Domain Service** → mixes rules with I/O.
- **Re-validating domain rules in Application** → duplication and drift.
- **Throwing infrastructure exceptions from Domain** (e.g., `SqlException`) → leaks implementation details.

---

## Conclusion

Keep **Domain Services pure** and free from data access. Let **Application Services** orchestrate repositories and pass the required facts into the domain. This separation yields cleaner code, easier tests, and architectures that scale with complexity.
