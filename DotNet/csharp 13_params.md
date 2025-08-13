# Using `params` in C#

In C#, the `params` keyword allows a method to accept a variable number of arguments without requiring you to create an array explicitly.

In simple German (for reference):

> `params` ist ein Schlüsselwort in C#, das es einer Methode erlaubt, eine variable Anzahl von Argumenten zu akzeptieren, ohne dass man sie als Array erstellen muss.

## Example: Basic `params`

```csharp
// Method with params
void PrintNumbers(params int[] numbers)
{
    foreach (var n in numbers)
        Console.WriteLine(n);
}

// Call
PrintNumbers(1, 2, 3, 4);  // No need to create an array
```

💡 Note:

- Without `params`, you would have to write:  
  ```csharp
  PrintNumbers(new int[] {1, 2, 3, 4});
  ```
- With `params`, you can pass individual numbers directly, and the method receives them as an array.
- **Advantage:** `params` makes method calls cleaner and more readable, especially when passing multiple values, without explicitly creating an array.

**Example showing the advantage:**

```csharp
// Cleaner calls with params
PrintNumbers(5, 10, 15);      // Directly passing numbers
PrintNumbers(1, 2, 3, 4, 5);  // No need to create array each time

// Without params, you'd need:
PrintNumbers(new int[] {5, 10, 15});
PrintNumbers(new int[] {1, 2, 3, 4, 5});
```

## Example: `params` with `IEnumerable<T>` (C# 13)

```csharp
// Method with params and IEnumerable<T>
void PrintNumbers(params IEnumerable<int>[] numberCollections)
{
    foreach (var collection in numberCollections)
    {
        foreach (var n in collection)
            Console.WriteLine(n);
    }
}

// Call
PrintNumbers(new List<int> { 1, 2 }, new int[] { 3, 4 });
```

💡 Notes:

- Before C# 13, `params` could only accept arrays (e.g., `int[]`). 
- In C# 13, you can pass collections like `List<int>` or `IEnumerable<int>` directly.
- This reduces memory allocations and improves performance since you don't have to convert everything to an array.
