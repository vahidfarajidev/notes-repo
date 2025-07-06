# Factory Method Pattern in C#

## Introduction

The Factory Method Pattern is a **creational design pattern** used to encapsulate object creation logic. It's especially useful when:

- A class has a `private` constructor.
- You want to control the way objects are created.
- You aim for **loose coupling** between object creation and usage.

## Why Use It?

If a class has a `private` constructor, it cannot be instantiated using `new` from outside the class. In such cases, a **factory method** offers a controlled and centralized way to create instances.

## Basic Structure with Benefits

```csharp
public class User
{
    public string Name { get; private set; }
    public string Role { get; private set; }

    // Private constructor
    private User(string name, string role)
    {
        Name = name;
        Role = role;
    }

    // Factory method with encapsulated logic, validation, and flexibility
    public static User Create(string name, string role)
    {
        // Validation: Ensure role is valid
        var allowedRoles = new[] { "Admin", "Guest", "Member" };
        if (!allowedRoles.Contains(role))
        {
            throw new ArgumentException("Invalid role");
        }

        // Flexibility: Different logic per role
        if (role == "Admin")
        {
            // Custom logic for Admin
            Console.WriteLine("Admin privileges granted.");
        }

        return new User(name, role); // Encapsulation: Creation logic is hidden
    }
}
```

### Usage

```csharp
try
{
    User user = User.Create("Alice", "Admin");
    Console.WriteLine($"{user.Name} - {user.Role}");
}
catch (Exception ex)
{
    Console.WriteLine(ex.Message);
}
```

## Benefits Demonstrated

- **Encapsulation**: The constructor is private; object creation is done only through the factory.
- **Validation**: Role is checked before the object is created.
- **Flexibility**: Custom logic is executed based on the role provided (e.g., logging or permission setup).

## Real-Life Analogy

Think of a **coffee shop**. You can't make the coffee yourself (constructor is private), but you can order one from the counter (factory method). Depending on what you order (role), the barista may make it differently.

## Conclusion

The Factory Method Pattern is simple but powerful. It gives you full control over instance creation and enforces constraints while keeping your code clean and maintainable.
