# Dictionary vs Hashtable in C#

In C#, both `Dictionary<TKey, TValue>` and `Hashtable` are used to store key-value pairs.  
But they have **important differences** in type safety, performance, and usage.

---

## 📦 Dictionary<TKey, TValue>

- Generic collection (`System.Collections.Generic`)
- Type-safe — you specify the type of keys and values
- Faster and more modern
- Should be your default choice

### ✅ Example:

```csharp
var dict = new Dictionary<string, int>();
dict["apple"] = 3;
dict["banana"] = 5;

Console.WriteLine(dict["apple"]); // Output: 3
```

---

## 🧳 Hashtable

- Non-generic (`System.Collections`)
- Keys and values are `object`, so you need casting
- Older — kept for backward compatibility
- Slower than `Dictionary` in most scenarios

### ⚠️ Example:

```csharp
var table = new Hashtable();
table["apple"] = 3;
table["banana"] = 5;

Console.WriteLine((int)table["apple"]); // Must cast
```

---

## 🧠 Key Differences

| Feature        | Dictionary              | Hashtable               |
|----------------|--------------------------|--------------------------|
| Type-safe      | ✅ Yes (Generic)         | ❌ No (Object-based)     |
| Namespace      | `System.Collections.Generic` | `System.Collections` |
| Performance    | ✅ Faster                | 🚫 Slower               |
| Null Keys      | ❌ No (throws exception) | ✅ Yes (one null allowed)|
| Thread-safe    | ❌ No (use ConcurrentDict) | ❌ No                   |

---

## ✅ When to Use What?

| Use Case                              | Recommended |
|---------------------------------------|-------------|
| Modern .NET applications              | `Dictionary`|
| Legacy or interop with old APIs       | `Hashtable` |
| Multi-threaded dictionary             | `ConcurrentDictionary` |

---

## 🧾 Summary

- Use `Dictionary<TKey, TValue>` for most scenarios
- Avoid `Hashtable` unless you have legacy needs
- For thread-safe operations, use `ConcurrentDictionary`
