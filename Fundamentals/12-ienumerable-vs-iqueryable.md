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



---

## 🔍 What Does "Executes on the Data Source (Server-Side)" Mean?

When we say `IQueryable<T>` executes on the data source, we mean:

- The LINQ query is **not executed immediately**
- Instead, it's **converted into a SQL query**
- The SQL is sent to the **database**, and **only the results** are returned

### ✅ Example:

```csharp
var users = dbContext.Users
    .Where(u => u.Age > 18)
    .Select(u => u.Name);
```

This doesn't fetch data yet — it just builds the query.

Later, when you call:

```csharp
var result = users.ToList();
```

Only then the query runs, like:

```sql
SELECT Name FROM Users WHERE Age > 18
```

This means **filtering is done by the database**, not in memory.

---

## 🧠 In Simple Words

> When you write:
```csharp
var users = dbContext.Users
    .Where(u => u.Age > 18)
    .Select(u => u.Name);
```

- It’s just **building a query**
- No data is loaded into memory yet
- Query executes **only when you enumerate** (`ToList()`, `First()`, `Count()`, etc.)

This helps improve performance and memory usage significantly — especially with large datasets.
