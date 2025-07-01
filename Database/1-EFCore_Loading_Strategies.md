# Entity Framework Core: Eager vs Lazy vs Explicit Loading

When working with related data in Entity Framework Core, you have three primary strategies for loading related entities:

- **Eager Loading**
- **Lazy Loading**
- **Explicit Loading**

Each method has its own use cases, trade-offs, and configuration requirements. This guide walks you through each one, their differences, and when to use them.

---

## 🧱 Example Entity Classes

```csharp
public class Blog
{
    public int BlogId { get; set; }
    public string Name { get; set; }

    public virtual ICollection<Post> Posts { get; set; }  // Note: 'virtual' is needed for Lazy Loading
}

public class Post
{
    public int PostId { get; set; }
    public string Title { get; set; }

    public int BlogId { get; set; }
    public virtual Blog Blog { get; set; }

    public virtual Author Author { get; set; }  // For reference loading
}

public class Author
{
    public int AuthorId { get; set; }
    public string Name { get; set; }

    public virtual ICollection<Post> Posts { get; set; }
}
```

---

## 🔹 Eager Loading

### ✅ Definition
Eager Loading loads related data **along with the main entity** in a single query using the `.Include()` method.

### 🧪 Example

```csharp
var blogs = context.Blogs
    .Include(b => b.Posts)
    .ToList();
```

### 👍 Pros
- Single database query.
- Efficient when you know you’ll need the related data.

### 👎 Cons
- Might load more data than necessary (over-fetching).

---

## 🔹 Lazy Loading

### ✅ Definition
Lazy Loading automatically loads related data **only when you access** the navigation property.

### 🧪 Example

```csharp
var blog = context.Blogs.First();
var posts = blog.Posts; // EF triggers a separate query here
```

### 🛠 Requirements
- Install `Microsoft.EntityFrameworkCore.Proxies`
- Use `.UseLazyLoadingProxies()` in your `DbContext`
- Navigation properties must be marked as `virtual`

### 👍 Pros
- Loads data only when needed.
- Cleaner code without `.Include()`

### 👎 Cons
- Can cause **N+1 query problem** (many small DB calls).
- Requires extra setup.

### ❓ Common Question:  
**"What does it mean that EF loads data automatically when you access a property?"**

> EF uses dynamic proxy classes that override your virtual properties. When you access something like `blog.Posts`, EF detects the access and fires a SQL query behind the scenes to load that data.

---

## 🔹 Explicit Loading

### ✅ Definition
Explicit Loading allows you to **manually load** related data by calling `.Load()` on a navigation property.

### 🧪 Example

```csharp
var blog = context.Blogs.First();
context.Entry(blog)
    .Collection(b => b.Posts)
    .Load();
```

For reference properties:

```csharp
context.Entry(post)
    .Reference(p => p.Author)
    .Load();
```

### 👍 Pros
- Full control over what gets loaded and when.
- Can avoid unnecessary DB calls.

### 👎 Cons
- Requires more code.
- Can still result in **N+1 problem** if not optimized.

### ❓ Common Question:
**"Does Explicit Loading also cause multiple queries like Lazy Loading?"**

> Yes — if you load related entities in a loop:

```csharp
foreach (var blog in context.Blogs)
{
    context.Entry(blog).Collection(b => b.Posts).Load();
}
```

This results in multiple queries: one for each `Blog` (N+1 problem). To avoid this, use `Include()` when you need to bulk load.

---

## 🟨 Key Differences

| Feature               | Eager Loading     | Lazy Loading        | Explicit Loading      |
|----------------------|-------------------|---------------------|-----------------------|
| Query Count          | Usually 1         | Many (per property) | Many (per entity)     |
| When Data Loads      | Immediately       | On access           | On manual `.Load()`   |
| Setup Required       | None              | Yes (proxies + virtual) | No                |
| Control Level        | Medium            | Low                 | High                  |
| Risk of Over-fetching| Yes               | No                  | No                    |
| Risk of N+1 Problem  | No                | Yes                 | Yes                   |

---

## 🟢 Recommendations

- Use **Eager Loading** when you know the related data is always needed.
- Use **Explicit Loading** when you want full control and are loading conditionally.
- Be **cautious with Lazy Loading** — it can be convenient but may harm performance.

---

## 📦 NuGet Installation for Lazy Loading

```bash
dotnet add package Microsoft.EntityFrameworkCore.Proxies
```

In your `DbContext`:

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer("YourConnectionString")
        .UseLazyLoadingProxies();
}
```

---

## 📌 Summary

Understanding the loading strategies in EF Core is key to writing efficient, clean, and scalable data access code. Use the right approach depending on your performance needs and coding style.
