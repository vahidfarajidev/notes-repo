
# Returning Multiple Values from a Method in C#

In C#, returning more than one value from a method can be done in several ways, depending on the language version and coding style. This article covers four common approaches:

---

## ✅ 1. Modern Way (Named Tuples)

Starting from **C# 7.0**, you can use named tuples to return multiple values from a method.

### Example:

```csharp
(string Name, int Age) GetPerson() => ("Ali", 28);
```

or

```csharp
(string Name, int Age) GetPerson()
{
    return ("Vahid", 40);
}
```

### Usage:

```csharp
var person = GetPerson();
Console.WriteLine(person.Name); // Ali
Console.WriteLine(person.Age);  // 28
```

### Advantages:
- Shorter, more readable code
- No need to define a separate class or struct

---

## ✅ 2. Using a Class (Traditional OOP Style)

This method works in **all versions of C#**, and is useful when more structure or object-oriented features are needed.

### Define a class:

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
}
```

### Return method:

```csharp
public Person GetPerson()
{
    Person person = new Person();
    person.Name = "Ali";
    person.Age = 28;
    return person;
}
```

### Usage:

```csharp
var person = GetPerson();
Console.WriteLine(person.Name);
Console.WriteLine(person.Age);
```


### Alternative: Using a `struct`

For lightweight data containers, you can also use a `struct` instead of a class. This is often used for performance in value-type scenarios.

```csharp
public struct Person
{
    public string Name;
    public int Age;
}

public Person GetPerson()
{
    Person person = new Person();
    person.Name = "Ali";
    person.Age = 28;
    return person;
}
```

### Usage:

```csharp
var person = GetPerson();
Console.WriteLine(person.Name);
Console.WriteLine(person.Age);
```

### Notes:
- `struct` is a value type and typically used for small, immutable data.
- Avoid using `struct` if you plan to modify it frequently or need inheritance.


---

## ✅ 3. Older Way with `out` Parameters

In earlier versions of C#, `out` parameters were used to return multiple values.

### Example:

```csharp
void GetPerson(out string name, out int age)
{
    name = "Ali";
    age = 28;
}
```

### Usage:

```csharp
string name;
int age;
GetPerson(out name, out age);
Console.WriteLine(name);
Console.WriteLine(age);
```

### Drawbacks:
- Less readable
- Harder to maintain

---

## ✅ 4. Classic Way with `Tuple<T1, T2>`

Before C# 7.0 introduced value tuples, developers could return multiple values using the `System.Tuple` class. This approach works in older versions like **C# 4.0 and later**.

### Example:

```csharp
Tuple<string, int> GetPerson()
{
    return new Tuple<string, int>("Ali", 28);
}

// Usage:
var person = GetPerson();
Console.WriteLine(person.Item1); // Ali
Console.WriteLine(person.Item2); // 28
```

### Advantages:
- Works in older versions of C# (C# 4+)
- No need to define a separate class

### Drawbacks:
- Members are accessed via `Item1`, `Item2`, etc., which makes the code less readable
- Not as flexible or expressive as named tuples

---

## Summary

| Method                | Required C# Version | Pros                                   | Cons                             |
|-----------------------|---------------------|----------------------------------------|----------------------------------|
| Named Tuples          | C# 7.0 and later     | Highly readable, no need for classes   | Requires newer version           |
| Using a Class         | All versions         | Extendable, object-oriented            | More verbose code                |
| `out` Parameters      | All versions         | Simple for small data                  | Less readable, limited use cases |
| `Tuple<T1, T2>`       | C# 4.0 and later     | Works in older versions, no class req. | Poor readability                 |

---

## 🧠 Explanation of This Syntax

The following line of code is a concise and modern way of returning multiple values using named tuples and an expression-bodied method:

```csharp
(string Name, int Age) GetPerson() => ("Ali", 28);
```

### Breakdown:

- `(string Name, int Age)`: The return type is a tuple with named elements `Name` and `Age`.
- `GetPerson()`: This is the method name.
- `=> ("Ali", 28);`: This is an expression-bodied method that returns the tuple `("Ali", 28)` directly.

This syntax is available from **C# 7.0** onwards and is great for returning lightweight, structured data without creating a separate class.

### Benefits:
- **Concise**: One-liner definition.
- **Readable**: Named tuple elements make the code self-documenting.
- **Efficient**: No need to define a new class or struct just to return a pair of values.
