# JWT Authentication in ASP.NET Core

## Introduction

JWT (JSON Web Token) is a compact, URL-safe means of representing claims to be transferred between two parties. In ASP.NET Core, JWT authentication is widely used to secure APIs in a stateless way.

This article will guide you through implementing JWT authentication in an ASP.NET Core Web API.

---

## Table of Contents

1. [What is JWT?](#what-is-jwt)
2. [Why Use JWT?](#why-use-jwt)
3. [Setting Up JWT in ASP.NET Core](#setting-up-jwt-in-aspnet-core)
4. [Generating a JWT Token](#generating-a-jwt-token)
5. [Securing Endpoints](#securing-endpoints)
6. [Validating Tokens](#validating-tokens)
7. [Best Practices](#best-practices)
8. [Conclusion](#conclusion)

---

## What is JWT?

A **JSON Web Token (JWT)** consists of three parts:

- **Header**: Contains the algorithm and token type.
- **Payload**: Includes claims such as user information and roles.
- **Signature**: Used to verify that the token was not tampered with.

**Example:**

```json
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

---

## Why Use JWT?

- **Stateless**: No session is stored on the server. Each request carries all necessary information, making the system easier to scale and deploy across multiple servers. All the required user information is encoded in the token itself and sent with each request. This eliminates the need for server-side session storage, allowing easy deployment across multiple servers without session synchronization.
- **Scalable**: Perfect for microservices and distributed systems. Since there's no dependency on a central session store, JWT is ideal for distributed architectures such as microservices or cloud-native applications.
- **Compact**: Efficient for transmission over HTTP. JWTs are small in size, encoded in Base64URL format, and can be easily transmitted via HTTP headers. This makes them efficient for APIs and mobile applications.

---

## Setting Up JWT in ASP.NET Core

### 1. Install Required Packages

```bash
dotnet add package Microsoft.AspNetCore.Authentication.JwtBearer
```

### 2. Configure `Program.cs` or `Startup.cs`

```csharp
builder.Services.AddAuthentication("Bearer")
    .AddJwtBearer("Bearer", options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = "yourapp.com",
            ValidAudience = "yourapp.com",
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("Your_Secret_Key"))
        };
    });

builder.Services.AddAuthorization();
```

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

---

## Generating a JWT Token

Here’s an example controller method to generate a token:

```csharp
[HttpPost("login")]
public IActionResult Login([FromBody] LoginModel user)
{
    if (user.Username == "admin" && user.Password == "password")
    {
        var claims = new[]
        {
            new Claim(ClaimTypes.Name, user.Username),
            new Claim(ClaimTypes.Role, "Admin")
        };

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes("Your_Secret_Key"));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: "yourapp.com",
            audience: "yourapp.com",
            claims: claims,
            expires: DateTime.Now.AddHours(1),
            signingCredentials: creds
        );

        return Ok(new
        {
            token = new JwtSecurityTokenHandler().WriteToken(token)
        });
    }

    return Unauthorized();
}
```

---

## Securing Endpoints

Add the `[Authorize]` attribute to secure any controller or action:

```csharp
[Authorize]
[HttpGet("secure-data")]
public IActionResult GetSecureData()
{
    return Ok("This is protected data.");
}
```

You can also secure by role:

```csharp
[Authorize(Roles = "Admin")]
```

---

## Validating Tokens

The middleware automatically validates the JWT on each request. If validation fails (e.g., invalid signature, expired token), the API will return a `401 Unauthorized`.

---

## Best Practices

- **Use HTTPS**: Always transmit tokens over HTTPS.
- **Keep Secret Keys Safe**: Store secrets in environment variables or secure vaults.
- **Token Expiry**: Set short expiration times and implement refresh tokens.
- **Limit Claims**: Avoid putting sensitive or unnecessary data in the token payload.
- **Use Strong Keys**: Minimum 256-bit key for HMAC.

---

## Conclusion

JWT is a powerful mechanism for securing APIs in ASP.NET Core. With proper implementation and best practices, it enables scalable and stateless authentication.

---

**Resources:**

- [Microsoft Docs: JWT Bearer Authentication](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/jwt)
- [jwt.io](https://jwt.io)
