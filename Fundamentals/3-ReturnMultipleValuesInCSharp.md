
# Returning Multiple Values from a Method in C#

In C#, returning more than one value from a method can be done in several ways, depending on the language version and coding style. This article covers three common approaches:

---

## ✅ 1. Modern Way (Named Tuples)

Starting from **C# 7.0**, you can use named tuples to return multiple values from a method.

### Example:

```csharp
(string Name, int Age) GetPerson() => ("Ali", 28);

// Usage:
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

// Usage:
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

## Summary

| Method | Required C# Version | Pros | Cons |
|--------|---------------------|------|------|
| Named Tuples | C# 7.0 and later | Highly readable, no need for custom types | Requires newer version |
| Using a Class | All versions | Extendable, object-oriented | More verbose code |
| `out` Parameters | All versions | Simple for small data | Less readable, limited use cases |

---

For modern projects, using tuples is recommended unless object-oriented modeling is necessary.
