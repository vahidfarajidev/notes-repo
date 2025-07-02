# Expression-bodied vs Block-bodied Methods in C#

In C#, methods can be defined in two main ways: **Expression-bodied** and **Block-bodied**. This article explores the differences, benefits, and use cases of each style.

---

## ✏️ 1. Block-bodied Methods

This is the traditional way to define methods in C#, using curly braces `{}` to enclose the method body.

```csharp
public int Square(int number)
{
    return number * number;
}
```

### ✅ Advantages:
- High readability for multi-line code
- Allows multiple statements
- Suitable for complex logic

---

## ⚡ 2. Expression-bodied Methods

Introduced in C# 6 and improved in later versions, this syntax allows you to define methods using a single expression and the `=>` operator.

```csharp
public int Square(int number) => number * number;
```

### ✅ Advantages:
- Concise and clean for simple methods
- Great for getters and setters
- Improves readability for one-liners

---

## 📌 When to Use Which?

| Condition | Recommendation |
|----------|----------------|
| Single statement | Use expression-bodied |
| Multiple lines or complex logic | Use block-bodied |
| Focus on team readability | Prefer block-bodied for clarity |

---

## ✅ Example: Constructor Comparison

```csharp
// Expression-bodied
public PercentageDiscount(decimal percent) => _percent = percent;

// Block-bodied
public PercentageDiscount(decimal percent)
{
    _percent = percent;
}
```

---

## 🔚 Conclusion

Expression-bodied syntax is a great way to simplify your code, but it shouldn't replace block bodies without consideration. Choose the style that best fits the **simplicity, readability, and context** of your project.
