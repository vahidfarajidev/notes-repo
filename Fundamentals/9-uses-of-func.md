# Practical Uses of `Func<>` in C#

`Func<>` is a built-in generic delegate in C# used to represent methods that **return a value**. It's one of the most powerful and flexible tools for writing clean, reusable, and functional-style code.

Below are six practical examples showing how `Func<>` is used in real-world C# scenarios — including callback functions, LINQ, logic injection, event handling, and more.

---

## ✅ Example 1: Basic Calculator

```csharp
Func<int, int, int> add = (a, b) => a + b;
Console.WriteLine(add(5, 6)); // Output: 11
```

Use `Func<int, int, int>` instead of defining a custom delegate.

---

## ✅ Example 2: LINQ Filtering

```csharp
var numbers = new List<int> { 1, 2, 3, 4, 5 };
Func<int, bool> isEven = n => n % 2 == 0;

var evens = numbers.Where(isEven);
foreach (var n in evens)
    Console.WriteLine(n); // Output: 2, 4
```

`Func<int, bool>` is used in `Where` to filter values.

---

## ✅ Example 3: Loose Coupling (Injecting Logic)

```csharp
public class PaymentProcessor
{
    private readonly Func<decimal, bool> _validate;

    public PaymentProcessor(Func<decimal, bool> validateStrategy)
    {
        _validate = validateStrategy;
    }

    public void Pay(decimal amount)
    {
        if (_validate(amount))
            Console.WriteLine("Payment accepted.");
        else
            Console.WriteLine("Payment rejected.");
    }
}

var processor = new PaymentProcessor(amount => amount < 1000);
processor.Pay(500); // Output: Payment accepted.
```

Pass business logic dynamically via `Func<>`.

---

## ✅ Example 4: Callback Function — Progress Reporting

```csharp
public class FileDownloader
{
    public void Download(string fileUrl, Func<int, string> onProgress)
    {
        for (int i = 1; i <= 10; i++)
        {
            Thread.Sleep(300);
            Console.WriteLine(onProgress(i * 10));
        }

        Console.WriteLine("Download complete!");
    }
}

var downloader = new FileDownloader();
downloader.Download("http://example.com/file.zip", percent => $"Progress: {percent}%");
```

Here, `Func<int, string>` lets the caller define how progress messages should be formatted.

---

## ✅ Example 5: Callback Function — Validation and Result

```csharp
public class InputValidator
{
    public string Validate(string input, Func<string, bool> isValid)
    {
        return isValid(input) ? "Valid input." : "Invalid input.";
    }
}

var validator = new InputValidator();
string result = validator.Validate("hello123", value => value.Length >= 5 && value.All(char.IsLetterOrDigit));
Console.WriteLine(result); // Output: Valid input.
```

Encapsulate validation logic and make it customizable.

---

## ✅ Example 6: Event Handling (Action used with Events)

```csharp
public class Button
{
    public event Action<string> Clicked;

    public void SimulateClick()
    {
        Clicked?.Invoke("Button1");
    }
}

var button = new Button();
button.Clicked += name => Console.WriteLine($"{name} clicked!");

button.SimulateClick(); // Output: Button1 clicked!
```

This shows how `Action` (and optionally `Func`) work within event-driven patterns.

---

## 🧠 Summary

| Example # | Use Case                    | Delegate Type              |
|-----------|-----------------------------|----------------------------|
| 1         | Basic operation             | `Func<int, int, int>`      |
| 2         | LINQ filtering              | `Func<int, bool>`          |
| 3         | Logic injection             | `Func<decimal, bool>`      |
| 4         | Callback (progress format)  | `Func<int, string>`        |
| 5         | Callback (input validation) | `Func<string, bool>`       |
| 6         | Event handling              | `Action<string>`           |

Using `Func<>` (and `Action<>`) helps you write **more flexible, testable, and expressive code** without needing custom delegates.
