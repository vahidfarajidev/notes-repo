# EF Core 8: Working with Stored Procedures, UDFs, and Views (2025)

In 2025, using the latest **EF Core 8**, you can efficiently work with **Stored Procedures (SP)**, **User-Defined Functions (UDFs)**, and **Views**. Here's a clean guide with examples.

---

## 1. Stored Procedures

- EF Core 8 still supports ``** and **`` to execute SPs.
- Reading and writing data with SPs is straightforward, no need for old EDMX or Function Imports.

**Reading data from a SP:**

```csharp
var topCustomers = await context.Customers
    .FromSqlRaw("EXEC GetTopCustomers")
    .ToListAsync();
```

**Writing or updating data using a SP:**

```csharp
await context.Database.ExecuteSqlRawAsync(
    "EXEC UpdateCustomerStatus @Id = {0}, @Status = {1}", customerId, status);
```

- EF Core 8 also supports **async/await and cancellation tokens**, making SP execution scalable and performant.

---

## 2. User-Defined Functions (UDFs)

- EF Core 8 allows mapping UDFs using ``** or mapping conventions**.
- SQL functions can be used in LINQ, and EF Core translates them to SQL queries.

```csharp
public class AppDbContext : DbContext
{
    [DbFunction("GetFullName", "dbo")]
    public string GetFullName(string firstName, string lastName) =>
        throw new NotImplementedException();
}

// Using in LINQ
var query = await context.Users
    .Select(u => new {
        FullName = context.GetFullName(u.FirstName, u.LastName)
    })
    .ToListAsync();
```

- This ensures calculations happen **directly in the database**, reducing data transfer.

---

## 3. Views

- EF Core 8 can map **Views as read-only DbSets**.
- Direct updates on Views are usually not allowed unless the View has **INSTEAD OF triggers**.

```csharp
public DbSet<ActiveCustomer> ActiveCustomers { get; set; }

// Using in LINQ
var active = await context.ActiveCustomers
    .Where(a => a.TotalPurchase > 1000)
    .ToListAsync();
```

- Views are excellent for **reducing query complexity in code** and **optimizing heavy database processing**.

---

## 💡 Summary (EF Core 8)

- **Stored Procedure:** Reading/writing data with SPs using async/await
- **Functions:** Perform calculations inside the database via LINQ
- **Views:** Read precomputed data, reduce data volume, simplify queries

This approach helps build **high-performance, scalable applications** and is perfect to mention in interviews when discussing **database and API optimization skills**.

