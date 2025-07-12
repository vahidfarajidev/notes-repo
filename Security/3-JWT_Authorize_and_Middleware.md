
# Understanding `[Authorize]` and JWT Middleware in ASP.NET Core

## 🔐 What Does `[Authorize]` Do?

`[Authorize]` is an attribute used in ASP.NET Core to protect endpoints. When you use it above a controller or action method, you tell the framework:

> "Don't allow access to this endpoint unless the user is authenticated."

### ✅ What Does It Check?

Behind the scenes, these two middlewares do the heavy lifting:

```csharp
app.UseAuthentication(); // Checks JWT or cookies, etc.
app.UseAuthorization();  // Checks role or policy
```

They ensure that:

- The request contains a valid token
- The token is properly signed, not expired, and matches the expected `issuer` and `audience`
- The user has permission to access the resource

---

## 📌 Common Use Cases for `[Authorize]`

### 🔹 Basic usage:

```csharp
[Authorize]
```
➡ Requires a valid token.

### 🔹 Role-based restriction:

```csharp
[Authorize(Roles = "Admin")]
```
➡ Only users with the role `Admin` can access this endpoint.

### 🔹 Anonymous access override:

```csharp
[AllowAnonymous]
public IActionResult PublicEndpoint()
```
➡ Allows public access even if the controller has `[Authorize]`.

---

### 🧠 Summary:

| Attribute                  | Description                               |
|---------------------------|-------------------------------------------|
| `[Authorize]`             | Requires authentication                   |
| `[Authorize(Roles="Admin")]` | Requires specific role                  |
| `[AllowAnonymous]`        | Skips auth entirely for that endpoint     |

---

## 🔐 What Does This Do in `Program.cs`?

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = issuer,
            ValidAudience = audience,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(secretKey))
        };
    });
```

### ✳️ What It Does:

This registers JWT authentication in the application's service container.  
It tells ASP.NET Core how to:

- **Read and validate tokens**
- **Verify the signature**
- **Check expiration, issuer, and audience**

---

### 🔍 Line-by-line Explanation:

| Setting                      | What It Does                                              |
|-----------------------------|-----------------------------------------------------------|
| `ValidateIssuer = true`     | Checks `iss` in token matches expected `issuer`           |
| `ValidateAudience = true`   | Checks `aud` in token matches expected `audience`         |
| `ValidateLifetime = true`   | Ensures token is not expired                              |
| `ValidateIssuerSigningKey = true` | Verifies signature with the expected key             |
| `IssuerSigningKey`          | The actual secret key used to validate the token signature|

---

## 🔄 Relationship Between `[Authorize]` and JWT Config

| Component                                | Responsibility                                  |
|------------------------------------------|--------------------------------------------------|
| `AddAuthentication().AddJwtBearer(...)` | Configure how JWT tokens are validated           |
| `app.UseAuthentication()`               | Actually inspects the token in the request       |
| `[Authorize]`                           | Enforces that only authenticated users access it |

---

## ✅ Conclusion

- `[Authorize]` marks which endpoints require login
- `UseAuthentication()` & JWT middleware check and validate the token
- Config in `Program.cs` tells ASP.NET Core **how** to validate tokens

Together, they form a complete and secure JWT-based auth system in ASP.NET Core.
