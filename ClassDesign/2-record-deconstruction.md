# Deconstruction in C#: `record` vs `class`

In C#, **deconstruction** is the ability to split an object into its component parts (i.e., properties) using tuple-like syntax.

## ✅ Deconstruction with `record`

When you define a `record`, C# automatically provides support for deconstruction. That means you can easily extract its properties without writing any extra code.

### Example

```csharp
public record Person(string Name, int Age);

var person = new Person("Alice", 30);
var (name, age) = person;

Console.WriteLine($"{name} is {age} years old.");
