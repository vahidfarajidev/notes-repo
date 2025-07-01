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



---

## 🔒 What Does "Type-Safe" Mean in Dictionary?

When using `Dictionary<TKey, TValue>`, the compiler knows **exactly** what types you're working with.

```csharp
var dict = new Dictionary<string, int>();
dict["apple"] = 10; // ✅ OK
dict[5] = "oops";   // ❌ Compile-time error
```

✅ This is called **type-safety** — the compiler catches type errors **before the program even runs**.

---

## ⚠️ What About Hashtable?

With `Hashtable`, you store everything as `object`, so you lose type information.

```csharp
var table = new Hashtable();
table["apple"] = 10;
int value = (int)table["apple"]; // ✅ OK

table["banana"] = "oops";
int broken = (int)table["banana"]; // ❌ Runtime error: InvalidCastException
```

⛔ The error is **not detected at compile-time**, and will crash the app **during runtime**.

---

## 🧠 Compile-Time vs Run-Time (Visual Studio)

| Concept        | When?                        | What happens?                      | Example                  |
|----------------|-------------------------------|-------------------------------------|--------------------------|
| Compile-Time   | When you build/run the code   | Compiler checks syntax and types   | `dict["apple"] = "ten";` ❌
| Run-Time       | When the program is executing | Code runs with real data & logic   | Invalid cast from object ❌

In Visual Studio:
- **Compile-time errors** show up as red squiggles or build failures.
- **Run-time errors** crash the app or throw exceptions when it's running.

---

## 🔚 Summary:

- `Dictionary` is **type-safe** — prevents bugs early.
- `Hashtable` requires **casting** — more error-prone.
- Prefer `Dictionary` unless you're dealing with legacy APIs.
