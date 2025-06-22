
# 🧠 Managed vs Unmanaged Memory in .NET

Understanding the difference between managed and unmanaged memory is crucial for writing efficient and safe C# applications.

---

## ✅ What is Managed Memory?

- Managed memory is automatically handled by the **.NET CLR (Common Language Runtime)**.
- You **don't need to manually free** memory.
- Garbage Collector (GC) finds and frees memory when objects are no longer in use.

### ✔ Examples
```csharp
var list = new List<int>();
var str = "hello";
var obj = new MyClass();
```

---

## ❌ What is Unmanaged Memory?

- Memory not tracked or cleaned by .NET.
- You must **manually release** it, typically via `Dispose()` or native APIs.
- Can lead to **memory leaks** if not cleaned properly.

### ✔ Examples
- `FileStream`, `SqlConnection`, `Bitmap`
- Interop with native code (`DllImport`, COM)

---

## 🛠 How to Handle Unmanaged Resources

### Using `IDisposable`
If a class holds unmanaged resources, implement `IDisposable` and release the resources in `Dispose()`:

```csharp
public class MyResource : IDisposable
{
    private FileStream _file = new FileStream("data.txt", FileMode.Open);

    public void Dispose()
    {
        _file.Dispose();
    }
}
```

### Using the `using` Statement
This automatically calls `Dispose()` when the block exits:

```csharp
using (var stream = new FileStream("file.txt", FileMode.Open))
{
    // use stream
} // stream.Dispose() is called automatically
```

---

## 🧹 Why It Matters

| Feature                   | Managed        | Unmanaged       |
|---------------------------|----------------|-----------------|
| GC managed                | ✅ Yes         | ❌ No           |
| Needs manual cleanup      | ❌ No          | ✅ Yes          |
| Risk of memory leaks      | ❌ Low         | ✅ High         |

---

## 🧾 Summary

| Memory Type | Controlled By     | Cleanup Style     | Examples                       |
|-------------|-------------------|-------------------|--------------------------------|
| Managed     | .NET CLR & GC     | Automatic         | List, string, custom classes   |
| Unmanaged   | Developer/Natives | Manual (Dispose)  | FileStream, SqlConnection      |
