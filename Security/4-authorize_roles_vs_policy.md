# Understanding `[Authorize(Roles = "...")]` vs `[Authorize(Policy = "...")]` in ASP.NET Core

ASP.NET Core provides flexible mechanisms for controlling access to resources through **Authorization**. Two primary methods are:

- Role-based Authorization
- Policy-based Authorization

This article explains the differences, usage, and scenarios for `[Authorize(Roles = "...")]` and `[Authorize(Policy = "...")]`.

---

## 🔐 Role-based Authorization

Role-based authorization checks whether the authenticated user belongs to a specific **role**.

### ✅ Usage

```csharp
[Authorize(Roles = "Admin")]
public IActionResult AdminOnly()
{
    return Ok("You are an Admin!");
}
```

You can specify multiple roles using a comma:

```csharp
[Authorize(Roles = "Admin,Manager")]
```

### ✅ How it works

- Roles are added as `ClaimTypes.Role` claims in the JWT token.
- ASP.NET Core will check for this claim when accessing the endpoint.

---

## 🔐 Policy-based Authorization

Policy-based authorization is more **flexible** and **extensible**, and can check for:

- Specific claim values
- Multiple claims
- Custom requirements and conditions

### ✅ Defining a Policy in `Program.cs`

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("HRPolicy", policy =>
        policy.RequireClaim("Department", "HR"));

    options.AddPolicy("Over18", policy =>
        policy.RequireAssertion(context =>
            context.User.HasClaim(c =>
                c.Type == "Age" && int.Parse(c.Value) >= 18)));
});
```

### ✅ Usage in Controllers

```csharp
[Authorize(Policy = "HRPolicy")]
public IActionResult HROnly()
{
    return Ok("Welcome, HR employee!");
}
```

---

## 🔍 Role vs Policy Comparison

| Feature                 | `[Authorize(Roles = "...")]`     | `[Authorize(Policy = "...")]`         |
|------------------------|----------------------------------|----------------------------------------|
| Simplicity             | Easy                             | Flexible, more powerful               |
| Based on               | Roles                            | Claims, logic, custom handlers        |
| Custom conditions      | ❌ Not supported                  | ✅ Fully supported                     |
| Multiple conditions    | ❌ Limited to roles               | ✅ Combine multiple requirements       |
| Use cases              | Basic role checks                | Complex authorization logic           |

---

## 🧠 Best Practices

- Use **Roles** when you only need simple access control based on job titles or user types.
- Use **Policies** when you need to evaluate multiple claims or implement complex authorization logic.

---

## ✅ Example: JWT with Roles and Claims

When generating a JWT:

```csharp
var claims = new List<Claim>
{
    new Claim(ClaimTypes.Name, username),
    new Claim(ClaimTypes.Role, "Manager"),
    new Claim("Department", "HR")
};
```

This enables you to use both `[Authorize(Roles = "Manager")]` and `[Authorize(Policy = "HRPolicy")]` on protected endpoints.

---

## Conclusion

Both `[Authorize(Roles = "...")]` and `[Authorize(Policy = "...")]` are powerful tools in ASP.NET Core's authorization system. Choose the one that best fits the complexity of your access control needs.
