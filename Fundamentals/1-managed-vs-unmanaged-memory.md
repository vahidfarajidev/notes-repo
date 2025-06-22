
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

## 🧹 Why It Matters

| Feature                   | Managed        | Unmanaged       |
|---------------------------|----------------|-----------------|
| GC managed                | ✅ Yes         | ❌ No           |
| Needs manual cleanup      | ❌ No          | ✅ Yes          |
| Risk of memory leaks      | ❌ Low         | ✅ High         |

---

## 🔄 Safe Usage Example (Unmanaged Resource)
```csharp
using (var file = new FileStream("file.txt", FileMode.Open))
{
    // Use the file
} // Dispose() is called automatically
```

---

## 🧾 Summary

| Memory Type | Controlled By     | Cleanup Style     | Examples                       |
|-------------|-------------------|-------------------|--------------------------------|
| Managed     | .NET CLR & GC     | Automatic         | List, string, custom classes   |
| Unmanaged   | Developer/Natives | Manual (Dispose)  | FileStream, SqlConnection      |
