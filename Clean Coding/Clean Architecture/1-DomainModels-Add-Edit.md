# 🧱 Designing Add vs Edit Methods in Domain Models (Clean Architecture)

In object-oriented design, **how** you expose methods for creating or updating data matters — both for clarity and maintainability. Let's look at how `Add` (i.e., object creation) differs from `Edit` (i.e., mutation), and why you handle them differently in your domain models.

---

## ✨ Add = Create = Initialization

### Example:

```csharp
public class User
{
    public Guid Id { get; private set; }
    public string Name { get; private set; }
    public string Email { get; private set; }

    private User() { } // For EF or serialization

    private User(string name, string email)
    {
        Id = Guid.NewGuid();

        if (string.IsNullOrWhiteSpace(name))
            throw new ArgumentException("Name is required");

        if (string.IsNullOrWhiteSpace(email))
            throw new ArgumentException("Email is required");

        Name = name;
        Email = email;
    }

    public static User Create(string name, string email) => new User(name, email);
}
```

### ✅ Why `Create()` is enough?

- At creation time, **everything is being initialized at once**.
- There's no existing state to mutate.
- Therefore, you **don't need methods like `CreateName()` or `CreateEmail()`**.
- It’s more expressive and makes the intent clear: “build a full object”.

---

## 🔁 Edit = Change = Mutation

### Example:

```csharp
public void ChangeName(string newName)
{
    if (string.IsNullOrWhiteSpace(newName))
        throw new ArgumentException("Name cannot be empty");
    Name = newName;
}

public void ChangeEmail(string newEmail)
{
    if (string.IsNullOrWhiteSpace(newEmail))
        throw new ArgumentException("Email cannot be empty");
    Email = newEmail;
}
```

### ✅ Why `ChangeX()` is needed?

- The object **already exists** and only a **part of it is changing**.
- Separation improves readability, testing, and validation logic.
- Clients clearly express what part is being updated.

---

## 📐 Design Principle

| Action | Method Style | Reason |
|--------|--------------|--------|
| **Add (Initialization)** | `Create()` (with required fields) | All data is being set at once; no state to mutate |
| **Edit (Mutation)** | `ChangeName()`, `ChangeEmail()` | Only one part of an existing object is changing |

This design follows:
- **Encapsulation**: Validations stay close to the data
- **Clarity**: Readers can distinguish between creation and modification
- **Separation of Concerns**: Each method does one thing

---

## ✅ Summary

- Use `Create()` when initializing a complete new object.
- Use `ChangeX()` methods for updating specific fields after creation.
- Do **not** split `CreateName()` or `CreateEmail()` unless you're doing step-by-step builders or complex staged construction.

This approach aligns with **Clean Architecture** and **DDD (Domain-Driven Design)** best practices.

---
