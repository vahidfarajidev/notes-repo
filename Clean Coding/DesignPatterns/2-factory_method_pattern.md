# Factory Method Pattern in C#

## Introduction

The Factory Method Pattern is a **creational design pattern** used to encapsulate object creation logic. It's especially useful when:

- A class has a `private` constructor.
- You want to control the way objects are created.
- You aim for **loose coupling** between object creation and usage.

## Why Use It?

If a class has a `private` constructor, it cannot be instantiated using `new` from outside the class. In such cases, a **factory method** offers a controlled and centralized way to create instances.

## Basic Structure

```csharp
public class User
{
    public string Name { get; private set; }

    // Private constructor
    private User(string name)
    {
        Name = name;
    }

    // Factory method
    public static User Create(string name)
    {
        // Additional logic can go here
        return new User(name);
    }
}
```

### Usage

```csharp
User user = User.Create("Alice");
Console.WriteLine(user.Name); // Output: Alice
```

## Benefits

- **Encapsulation**: Internal creation logic is hidden.
- **Validation**: You can validate input before creating the object.
- **Flexibility**: You can return different subtypes if needed.

## Real-Life Analogy

Think of a **coffee shop**. You can't make the coffee yourself (constructor is private), but you can order one from the counter (factory method).

## Conclusion

The Factory Method Pattern is simple but powerful. It gives you full control over instance creation and enforces constraints while keeping your code clean and maintainable.
