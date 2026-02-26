# Entity Framework Core: Eager vs Lazy vs Explicit Loading (Revised & Corrected)

When working with related data in Entity Framework Core, you have three primary strategies for loading related entities:

- **Eager Loading**
- **Lazy Loading**
- **Explicit Loading**

The key difference between them is **when** and **how many times** the database is queried.

This guide explains each strategy using equivalent scenarios so the comparison is fair and technically accurate.

---

## 🧱 Example Entity Classes

```csharp
public class Blog
{
    public int BlogId { get; set; }
    public string Name { get; set; }

    public virtual ICollection<Post> Posts { get; set; }
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }

    public int BlogId { get; set; }
    public virtual Blog Blog { get; set; }
}
```

> ✅ For Lazy Loading, navigation properties must be `virtual` and proxies must be enabled.

---

# 🎯 Fair Comparison Scenario

Assume we want:

👉 "Load all blogs and their related posts"

Now we compare how each strategy behaves in the SAME scenario.

---

# 🔹 1️⃣ Eager Loading

```csharp
var blogs = context.Blogs
    .Include(b => b.Posts)
    .ToList();
```

### What happens?

- EF generates a SQL JOIN.
- Blogs and Posts are loaded together.
- Only **one database round trip**.

### SQL (simplified)

```sql
SELECT *
FROM Blogs
LEFT JOIN Posts ON Blogs.BlogId = Posts.BlogId
```

### ✅ Pros
- Single query
- No N+1 problem
- Best for bulk related data

### ❌ Cons
- May over-fetch data
- Large joins can increase payload size

---

# 🔹 2️⃣ Lazy Loading

```csharp
var blogs = context.Blogs.ToList();

foreach (var blog in blogs)
{
    var posts = blog.Posts;
}
```

### What happens?

1️⃣ One query loads all Blogs.
2️⃣ Each time `blog.Posts` is accessed, EF fires a separate query.

If you have 100 blogs:

- 1 query for Blogs
- 100 queries for Posts

Total = 101 queries ❌ (N+1 problem)

### When is it acceptable?
- When accessing related data rarely
- When working with a single entity

### ⚠️ Performance Risk
Lazy loading is convenient but dangerous in loops.

---

# 🔹 3️⃣ Explicit Loading

```csharp
var blogs = context.Blogs.ToList();

foreach (var blog in blogs)
{
    context.Entry(blog)
        .Collection(b => b.Posts)
        .Load();
}
```

### What happens?

Same query behavior as Lazy Loading:

- 1 query for Blogs
- N queries for Posts

### Difference from Lazy
- You manually control when loading happens
- No proxies required

### Better Explicit Pattern (Optimized)

Instead of loading inside a loop, you can batch load:

```csharp
var blogs = context.Blogs.ToList();

var blogIds = blogs.Select(b => b.BlogId).ToList();

var posts = context.Posts
    .Where(p => blogIds.Contains(p.BlogId))
    .ToList();
```

Now:
- 1 query for Blogs
- 1 query for Posts

Total = 2 queries ✅ (optimized explicit loading)

---

# 🟨 Key Differences (Accurate Comparison)

| Feature | Eager Loading | Lazy Loading | Explicit Loading |
|----------|---------------|--------------|------------------|
| Query Count (bulk scenario) | 1 | 1 + N | 1 + N (unless optimized) |
| When Data Loads | Immediately | On property access | On manual `.Load()` |
| Setup Required | None | Proxies + virtual | None |
| Risk of N+1 | No | Yes | Yes |
| Control Level | Medium | Low | High |

---

# 🧠 When to Use What?

### ✅ Use Eager Loading when:
- You know related data is required
- You are loading collections
- You want predictable performance

### ✅ Use Lazy Loading when:
- Working with single entities
- Related data is optional
- Simplicity is preferred over performance guarantees

### ✅ Use Explicit Loading when:
- You want full control
- You need conditional loading
- You want to optimize manually

---

# ⚙️ Enabling Lazy Loading

Install:

```bash
dotnet add package Microsoft.EntityFrameworkCore.Proxies
```

Configure DbContext:

```csharp
optionsBuilder
    .UseSqlServer("YourConnectionString")
    .UseLazyLoadingProxies();
```

---

# 📌 Final Takeaway

The real difference is not about syntax — it is about:

- How many database round trips happen
- When related data is loaded
- Whether you risk the N+1 problem

For bulk relational data, **Eager Loading is usually the safest default**.

Lazy and Explicit loading require awareness and performance discipline.
