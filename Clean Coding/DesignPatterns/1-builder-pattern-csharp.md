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


---

## 🔍 Why Not Just Use a Big Constructor?

You might wonder: why not just define a constructor with all fields?

```csharp
public UserProfile(string name, string email, string phone, string picture, string bio)
```

This might seem fine at first, but it has serious downsides:

### 1. ❌ Poor Readability

```csharp
new UserProfile("Ali", "ali@example.com", "0912...", "pic.jpg", "bio...");
```

It’s unclear what each parameter means — especially with many `string`s.

---

### 2. ❌ Optional Parameters Become Mandatory

You are forced to pass values (like `null`) even when you don’t need them:

```csharp
new UserProfile("Ali", "ali@example.com", null, null, null); // ugly and error-prone
```

---

### 3. ❌ Violates Open/Closed Principle

If you add a new field (e.g., `TwitterHandle`), you must:

- Change the constructor:
  ```csharp
  public UserProfile(string name, string email, ..., string twitter)
  ```

- Update all usages of that constructor across the codebase

➡️ This breaks existing code = not closed for modification.

---

## ✅ Builder Solves These

With a builder, only new builder methods are added:

```csharp
public UserProfileBuilder WithTwitter(string handle)
{
    _twitter = handle;
    return this;
}
```

Old code keeps working, new fields are easy to add:

```csharp
var user = new UserProfileBuilder("Ali", "ali@example.com")
               .WithBio("C# Dev")
               .Build(); // Twitter not needed here
```

---

## 🧠 Final Comparison

| Feature                     | Constructor Approach          | ✅ Builder Pattern          |
|-----------------------------|-------------------------------|-----------------------------|
| Readability                 | ❌ Poor                        | ✅ Fluent & readable        |
| Optional parameter support  | ❌ Hard to handle              | ✅ Natural                  |
| Extensibility (Open/Closed) | ❌ Violates principle          | ✅ Fully preserved          |
| Maintenance                 | ❌ Requires global changes     | ✅ Add methods safely       |

---

## 🔧 Why Constructor for Name/Email?

In the builder:

```csharp
public UserProfileBuilder(string name, string email) { ... }
```

These fields are passed in the constructor because they're **required** — unlike the optional ones like `Phone`, `Bio`, etc., which are set using fluent methods.

It gives a good balance between:

- Ensuring required fields are set
- Providing flexibility for optional ones



---

## 🔍 Why Are There Dots After Each Line?

You might wonder why this syntax is used:

```csharp
var user = new UserProfileBuilder("Ali", "ali@example.com")
               .WithPhone("09120000000")
               .WithProfilePicture("ali.jpg")
               .WithBio("C# enthusiast")
               .Build();
```

### ✅ This style is called Fluent Interface or Method Chaining

It means that each method (like `.WithPhone()`) returns the builder itself (`this`), allowing the next method to be called on the same object.

### Example method:
```csharp
public UserProfileBuilder WithPhone(string phone)
{
    _user.Phone = phone;
    return this; // returns itself for chaining
}
```

So this line:

```csharp
new UserProfileBuilder("Ali", "ali@example.com")
    .WithPhone("0912...")
```

Means:

1. A `UserProfileBuilder` object is created
2. `.WithPhone()` is called, modifying the object and returning it
3. The returned builder allows `.WithBio()` to be called next

---

## 🧠 Why Use This?

| Feature | Benefit |
|--------|---------|
| Fluent Style | Code reads like a sentence |
| No repeated variables | More concise |
| Scales well | Easy to add optional values |

### 🧼 Clean and readable:

This:

```csharp
var user = new UserProfileBuilder("Ali", "ali@example.com").WithPhone("0912...").WithBio("Dev").Build();
```

Is identical to:

```csharp
var user = new UserProfileBuilder("Ali", "ali@example.com")
               .WithPhone("0912...")
               .WithBio("Dev")
               .Build();
```

Only difference: readability and formatting.

---

## 🔎 What Does `new UserProfileBuilder(...)` Return?

It returns a `UserProfileBuilder` instance. So calling `.WithPhone()` immediately after works, because the object supports that method and returns itself again.

```csharp
UserProfileBuilder builder = new UserProfileBuilder(...);
builder.WithPhone(...).WithBio(...).Build();
```

Is the same as chaining it directly.

---

## 📦 Summary

| Concept | Explanation |
|--------|-------------|
| `new UserProfileBuilder(...)` | Creates a builder object |
| `.WithXxx()` | Returns the same builder for further chaining |
| `Build()` | Produces the final `UserProfile` object |



---

## 📌 Note: What If There Are More Required Fields?

If additional fields (besides `Name` and `Email`) are **required**, they should also go in the constructor of the builder.

Example:

```csharp
public UserProfileBuilder(string name, string email, DateTime birthDate)
{
    _user = new UserProfile
    {
        Name = name,
        Email = email,
        BirthDate = birthDate
    };
}
```

This ensures that all essential fields are provided at the time of building the object and prevents the creation of incomplete instances.

| Field Type  | How to Provide   | Why?                             |
|-------------|------------------|----------------------------------|
| Required    | Constructor      | Object is invalid without them   |
| Optional    | Fluent methods   | Included only when needed        |
