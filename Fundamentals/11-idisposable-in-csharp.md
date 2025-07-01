# Understanding `IDisposable` in C#

`IDisposable` is an important interface in C# that is used to **release unmanaged resources** like files, database connections, sockets, memory handles, etc.

---

## 🧾 Definition

```csharp
public interface IDisposable
{
    void Dispose();
}
```

Any class that holds unmanaged resources should implement `IDisposable` and define cleanup logic in the `Dispose()` method.

---

## 🛠️ Why Use `IDisposable`?

Managed resources (like strings, lists) are automatically cleaned up by the .NET garbage collector.  
But **unmanaged resources** (e.g., file handles, DB connections) need to be cleaned up manually.

Failing to do this can cause:
- Memory leaks
- File locks
- Database connection exhaustion

---

## ✅ Common Use Cases

| Resource Type      | IDisposable? | Why?                         |
|--------------------|--------------|------------------------------|
| `FileStream`       | ✅ Yes       | Holds unmanaged file handle |
| `DbConnection`     | ✅ Yes       | Manages native connections  |
| `HttpClient`       | ✅ Yes       | Uses sockets internally     |
| Custom Classes     | ✅ If needed | When managing native things |

---

## 🧪 Example: Manual Dispose

```csharp
public class MyFileReader : IDisposable
{
    private FileStream _fileStream;

    public MyFileReader(string path)
    {
        _fileStream = new FileStream(path, FileMode.Open);
    }

    public void Dispose()
    {
        _fileStream?.Dispose(); // Free the resource
    }
}
```

---

## ✅ Better Way: `using` Statement

```csharp
using (var reader = new MyFileReader("data.txt"))
{
    // use reader
}
// Automatically calls Dispose()
```

Or with modern C#:

```csharp
using var reader = new MyFileReader("data.txt");
// reader.Dispose() will be called at end of scope
```

---

## 🧠 Summary

| Concept     | Description                                   |
|-------------|-----------------------------------------------|
| IDisposable | Interface for cleanup of unmanaged resources  |
| Dispose()   | Releases native/non-.NET resources manually   |
| `using`     | Ensures Dispose is called automatically       |
| When needed | When using files, DB, sockets, custom handles |

---

Using `IDisposable` properly ensures your app doesn't leak memory or hold system resources longer than necessary.
