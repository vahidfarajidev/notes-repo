# Builder Pattern in C#

The **Builder Pattern** is a creational design pattern that separates the construction of a complex object 
from its representation — so the same construction process can create different representations.

---

## ✅ When to Use

- When you need to build an object with many optional parameters
- To avoid telescoping constructors (too many overloaded constructors)
- To make object creation more readable and flexible

---

## 🧱 Basic Structure

- **Product**: The complex object being built
- **Builder**: Provides methods for setting parts of the product
- **ConcreteBuilder**: Implements the step-by-step logic
- **Build()**: Returns the final object

---

## 🧪 Example: Building a User Profile

### 1. Define the Product

```csharp
public class UserProfile
{
    public string Name { get; set; }
    public string Email { get; set; }
    public string Phone { get; set; }
    public string ProfilePicture { get; set; }
    public string Bio { get; set; }

    public override string ToString()
    {
        return $"Name: {Name}, Email: {Email}, Phone: {Phone}, Picture: {ProfilePicture}, Bio: {Bio}";
    }
}
```

### 2. Create the Builder

```csharp
public class UserProfileBuilder
{
    private readonly UserProfile _user;

    public UserProfileBuilder(string name, string email)
    {
        _user = new UserProfile
        {
            Name = name,
            Email = email
        };
    }

    public UserProfileBuilder WithPhone(string phone)
    {
        _user.Phone = phone;
        return this;
    }

    public UserProfileBuilder WithProfilePicture(string path)
    {
        _user.ProfilePicture = path;
        return this;
    }

    public UserProfileBuilder WithBio(string bio)
    {
        _user.Bio = bio;
        return this;
    }

    public UserProfile Build() => _user;
}
```

### 3. Usage

```csharp
var user = new UserProfileBuilder("Ali", "ali@example.com")
               .WithPhone("09120000000")
               .WithProfilePicture("ali.jpg")
               .WithBio("C# enthusiast")
               .Build();

Console.WriteLine(user);
// Output:
// Name: Ali, Email: ali@example.com, Phone: 09120000000, Picture: ali.jpg, Bio: C# enthusiast
```

---

## 🧠 Summary

| Role         | Responsibility                          |
|--------------|------------------------------------------|
| Product      | The object being built                   |
| Builder      | Provides fluent methods for building     |
| Build()      | Returns the final object                 |

✅ This pattern improves clarity and allows step-by-step object creation with clean syntax.

---

## ✅ Real-World Analogy

> Like filling out a profile on a website step-by-step:  
> Name and email are required, while other fields are optional.  
> The builder helps organize and construct the complete profile clearly.
