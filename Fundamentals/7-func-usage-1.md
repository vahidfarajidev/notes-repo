
# 🧠 Usage of Multiline `Func` in C#

```csharp
Func<int, int, int> add = (a, b) =>
{
    int result = a + b;
    Console.WriteLine($"Result is: {result}");
    return result;
};
```

---

## ✅ What does this code do?

This anonymous function takes two `int` values, calculates their sum,  
prints the result to the console, and then returns the value.

---

## 🔧 Common Use Cases

### 1. Assigning as a reusable variable (passing to other functions)

```csharp
int DoSomething(Func<int, int, int> operation)
{
    return operation(3, 5);
}

int result = DoSomething(add);  // Output: "Result is: 8"
Console.WriteLine(result);      // Output: 8
```

---

### 2. Using inside LINQ or dynamic method calls

```csharp
var results = new List<int> { 1, 2, 3, 4, 5 }
    .Select(x => add(x, 10))
    .ToList();
```

**Console Output:**
```
Result is: 11
Result is: 12
Result is: 13
Result is: 14
Result is: 15
```

---

### 3. When you need multiline logic

Useful when you need intermediate variables, logs, or conditional logic inside the delegate.

---

## 🔚 Summary

| Use Case                   | Why It's Useful                                     |
|----------------------------|-----------------------------------------------------|
| Passing as a parameter     | Enables flexible and dynamic behavior               |
| Usage in LINQ              | Allows filtering, mapping, and transformation logic |
| Multiline logic definition | Great for complex operations like logging or checks |
