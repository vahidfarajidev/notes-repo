# Method Group in C#

## ✅ What is a Method Group?

In C#, a **method group** is simply a reference to a method by its name **without parentheses**. It does **not invoke** the method — it just **points to** it.

---

## 📌 Simple Example

```csharp
void SayHello()
{
    Console.WriteLine("Hello!");
}

Action greet = SayHello; // method group reference
greet(); // Output: Hello!
```

Here:
- `SayHello` (without `()`) is a **method group**.
- `greet()` actually **executes** the method.

---

## 📌 Example with Parameters

```csharp
bool IsPositive(int number) => number > 0;

Func<int, bool> check = IsPositive; // method group
Console.WriteLine(check(5)); // Output: True
```

---

## 📌 With LINQ Methods

When using LINQ methods like `All`, `Where`, or `Select`, you can pass:

### A lambda expression:
```csharp
value.All(c => char.IsLetterOrDigit(c));
```

### Or a method group:
```csharp
value.All(char.IsLetterOrDigit);
```

Both are equivalent. The method group version is more concise when the method already matches the required delegate signature.

---

## 🔁 When to Use

Use a method group when:
- You already have a method that matches the required delegate signature.
- You want cleaner and more readable code.
- You don't need to rewrite a lambda.

---

## 🧠 Technical Note

A method group is not a delegate itself. It is a **set of overloaded methods** identified by name. At **compile time**, the compiler picks the appropriate overload and **converts it to a delegate**.

---

## ✅ Summary

| Concept        | Syntax                      | Meaning                          |
|----------------|-----------------------------|----------------------------------|
| Method Group   | `MethodName`                | A reference to the method        |
| Method Call    | `MethodName()`              | Invokes the method               |
| Lambda         | `x => SomeMethod(x)`        | Delegate using a lambda          |
| Method Group in LINQ | `collection.All(SomeMethod)` | Clean alternative to lambda |
