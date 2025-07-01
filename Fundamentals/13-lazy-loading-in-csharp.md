# Lazy Loading in C#

**Lazy Loading** is a design pattern that defers the creation or loading of an object until it is actually needed.

This helps improve performance and reduce memory usage, especially when dealing with heavy or resource-intensive objects.

---

## 🧠 What is Lazy Loading?

- Instead of loading everything upfront, the object is only created **on first access**
- It's useful when loading data is expensive (e.g., database queries, large files, or complex calculations)

---

## ✅ Use Case Example

Suppose you have a large dataset, but it's only needed **if** a user takes a certain action.  
Instead of loading it in advance, you wait until it’s requested.

---

## 🔧 Lazy<T> in C#

C# provides a built-in generic type called `Lazy<T>` to handle lazy loading.

```csharp
Lazy<HeavyObject> lazyObj = new Lazy<HeavyObject>(() => new HeavyObject());

...

if (needHeavyStuff)
{
    var obj = lazyObj.Value; // Now it gets created
}
```

- The object is **not created until `.Value` is accessed**
- You can wrap **any class or method** inside `Lazy<T>`

---

## 📦 Example: Without Lazy

```csharp
public class Report
{
    public HeavyData Data { get; } = new HeavyData(); // Loaded immediately
}
```

### With Lazy:

```csharp
public class Report
{
    private readonly Lazy<HeavyData> _lazyData = new Lazy<HeavyData>(() => new HeavyData());

    public HeavyData Data => _lazyData.Value; // Created only when accessed
}
```

---

## 🧪 Use in Entity Framework (EF Core)

In EF Core, lazy loading means navigation properties (like related entities) are loaded **automatically and on-demand** when you access them.

Enable lazy loading by:

1. Installing the package:  
```bash
Microsoft.EntityFrameworkCore.Proxies
```

2. Enabling it in `DbContext`:

```csharp
optionsBuilder.UseLazyLoadingProxies();
```

3. Making navigation properties `virtual`:

```csharp
public class Post
{
    public int Id { get; set; }

    public virtual ICollection<Comment> Comments { get; set; } // Lazy loaded
}
```

---

## ⚠️ When NOT to Use Lazy Loading

| Situation                             | Better Approach        |
|--------------------------------------|------------------------|
| You always need the data             | Use eager loading      |
| You want full control on performance | Use explicit loading   |
| You're in high-performance systems   | Avoid hidden loading   |

---

## 🧾 Summary

| Concept        | Description                                |
|----------------|--------------------------------------------|
| Lazy<T>        | Built-in C# type to delay object creation  |
| .Value         | Accessor that triggers creation            |
| EF Core usage  | Auto-load related entities on access       |
| Benefit        | Performance & memory optimization          |

Lazy loading can make your code more efficient — **just load what you really need, when you need it**.
