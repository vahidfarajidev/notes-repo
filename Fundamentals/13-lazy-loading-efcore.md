# Lazy Loading in Entity Framework Core

**Lazy Loading** in Entity Framework Core is a feature that allows related entities to be loaded **only when you access them**.  
This can improve performance by avoiding unnecessary database queries — until they're actually needed.

---

## 🧠 What is Lazy Loading?

Lazy loading means that EF Core **does not load navigation properties** when the entity is first retrieved.  
Instead, it waits until you try to use the property — then it automatically queries the database.

---

## ✅ How to Enable Lazy Loading in EF Core

EF Core doesn’t enable lazy loading by default. To use it:

### 1. Install the proxy package:

```bash
dotnet add package Microsoft.EntityFrameworkCore.Proxies
```

---

### 2. Enable proxies in `DbContext`:

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer("YourConnectionString")
        .UseLazyLoadingProxies();
}
```

---

### 3. Make navigation properties `virtual`:

```csharp
public class Post
{
    public int Id { get; set; }
    public string Title { get; set; }

    public virtual ICollection<Comment> Comments { get; set; } // Lazy loaded
}

public class Comment
{
    public int Id { get; set; }
    public string Text { get; set; }

    public int PostId { get; set; }
    public virtual Post Post { get; set; } // Lazy loaded
}
```

---

## ⚡ How It Works

```csharp
var post = context.Posts.First();      // Only loads Post
var comments = post.Comments;          // Triggers SQL query to load Comments
```

The query for `Comments` is only executed **when you access the property** — not before.

---

## 🔍 Compare: Lazy vs Eager vs Explicit

| Approach      | Description                                       |
|---------------|---------------------------------------------------|
| Lazy Loading  | Load on property access (deferred)                |
| Eager Loading | Load with `.Include()` when querying              |
| Explicit      | Load manually with `.Entry().Collection().Load()` |

---

## ⚠️ Gotchas & When to Avoid

- 🔄 **Too many queries**: Can lead to N+1 problems if used carelessly
- 🐌 **Hidden performance hits**: May make performance debugging harder
- 🧪 **Proxies only work on virtual & public properties**

---

## 🧾 Summary

| Feature           | Lazy Loading (EF Core)                      |
|-------------------|---------------------------------------------|
| Requires package  | `Microsoft.EntityFrameworkCore.Proxies`     |
| Requires keyword  | `virtual` on navigation properties           |
| Trigger           | Accessing the property                      |
| Benefit           | Delay loading until needed                  |
| Downside          | Risk of too many SQL calls                  |

Lazy loading in EF Core helps you **delay expensive queries** — but use it with care and monitor performance.
