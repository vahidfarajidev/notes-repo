# IEnumerable vs IQueryable in C#

In C#, both `IEnumerable<T>` and `IQueryable<T>` are used for querying collections.  
But they work **differently**, and knowing when to use each one is key for writing efficient and clean code.

---

## 🌿 IEnumerable<T>

- Defined in: `System.Collections.Generic`
- Represents a **sequence of items** you can loop through (`foreach`)
- Executes queries **in memory** (client-side)
- Great for working with **collections already in memory**

### ✅ Example:

```csharp
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };

IEnumerable<int> evens = numbers.Where(n => n % 2 == 0);

foreach (var num in evens)
    Console.WriteLine(num); // Output: 2, 4
```

> LINQ is executed in memory — the full list exists in RAM.

---

## 🌐 IQueryable<T>

- Defined in: `System.Linq`
- Represents a **queryable source**, like a **database**
- Supports **deferred execution** and **expression trees**
- LINQ queries are **translated to SQL** (e.g., Entity Framework)
- Executes queries **on the data source** (server-side)

### ✅ Example with EF Core:

```csharp
IQueryable<User> users = dbContext.Users
    .Where(u => u.IsActive)
    .OrderBy(u => u.LastName);
```

> The query is not executed until you enumerate it (`ToList()`, `First()`, etc.), and it's translated to SQL.

---

## 🧠 Key Differences

| Feature            | IEnumerable<T>         | IQueryable<T>          |
|--------------------|------------------------|-------------------------|
| Namespace          | System.Collections     | System.Linq             |
| Execution          | In-memory              | At data source (e.g. DB)|
| Performance        | May fetch too much     | Optimized SQL queries   |
| Use case           | In-memory collections  | Remote data (DB, API)   |
| Query translation  | Not supported          | Yes (via LINQ Providers)|

---

## 🚀 When to Use Which?

| Scenario                          | Use               |
|-----------------------------------|-------------------|
| Working with in-memory lists      | `IEnumerable<T>`  |
| Querying a database               | `IQueryable<T>`   |
| Need SQL optimization             | `IQueryable<T>`   |
| You want simple filtering in C#   | `IEnumerable<T>`  |

---

## 🔚 Summary

- `IEnumerable<T>` = pull data into memory, then filter
- `IQueryable<T>` = build query, then fetch only what's needed
- Choose based on where your data lives: memory vs server

Using the right interface helps you write **cleaner and more performant** applications — especially when working with databases.
