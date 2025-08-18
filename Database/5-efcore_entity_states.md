# Understanding Entity States in EF Core

In **Entity Framework Core (EF Core)**, every entity tracked by the `DbContext` has a **state**. This state determines what actions EF Core will take when `SaveChanges()` is called, such as adding, updating, deleting, or ignoring the entity.

## 1. Main Entity States

EF Core defines the following states in the `EntityState` enum:

| State         | Description                                                                 |
| ------------- | --------------------------------------------------------------------------- |
| **Added**     | The entity is new and will be inserted into the database.                   |
| **Unchanged** | The entity exists in the database and has not been modified.                |
| **Modified**  | The entity has been changed and will be updated in the database.            |
| **Deleted**   | The entity will be removed from the database.                               |
| **Detached**  | The entity is not tracked by the context, and EF Core does nothing with it. |

## 2. Example Code

Consider a simple model:

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

And a `DbContext`:

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Product> Products { get; set; }
}
```

### Practical Example

```csharp
using (var context = new AppDbContext())
{
    var product = new Product { Name = "Laptop", Price = 1000 };

    // State: Added
    context.Products.Add(product);
    Console.WriteLine(context.Entry(product).State); // Output: Added

    context.SaveChanges();
    Console.WriteLine(context.Entry(product).State); // Output: Unchanged

    // Modify the product
    product.Name = "Gaming Laptop";
    Console.WriteLine(context.Entry(product).State); // Output: Modified

    // Delete the product
    context.Products.Remove(product);
    Console.WriteLine(context.Entry(product).State); // Output: Deleted
}
```

## 3. Key Points

- Loaded entities are tracked as **Unchanged** by default.
- Changing any property updates the state to **Modified**.
- Using `Add` or `Remove` sets the state to **Added** or **Deleted**, respectively.
- **Detached** means the entity is not tracked, for example after being removed from the context.

---

Understanding entity states helps manage the **lifecycle of your entities** and ensures EF Core performs the correct database operations efficiently.

