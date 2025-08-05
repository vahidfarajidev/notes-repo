# API Key Authentication in ASP.NET Core 9

## Overview

API Key authentication is one of the simplest yet effective ways to secure your APIs in **.NET 9**. Unlike more complex schemes like OAuth or JWT, API Keys provide a straightforward mechanism for validating client requests while maintaining good security practices.

This document explains:

- What API Key authentication is
- How it compares with JWT
- How ASP.NET Core 9 improves API Key authentication with Authentication Handlers
- A complete example implementation of an API Key Authentication Handler
- When and how the handler is executed in the request pipeline
- A flow diagram of the authentication process

---

## What is API Key Authentication?

API Key authentication is a method where clients send a unique identifier (the API key) with each request to prove they are authorized to access API endpoints. The server validates this key against a stored list of valid keys before processing the request.

### How API Keys Are Sent

API Keys are commonly sent in one of these ways:

- HTTP Header (most common):  
  ```
  X-API-KEY: your-api-key
  ```
- Query String:  
  ```
  GET /api/data?api_key=your-api-key
  ```
- Request Body (less common for GET requests)

### Server-side Validation

The server checks the key from the request against a list/database of valid keys:

- If the key is valid, the request proceeds.
- If invalid or missing, the server returns `401 Unauthorized` or `403 Forbidden`.

### Pros and Cons

| Pros                        | Cons                                  |
|-----------------------------|-------------------------------------|
| Simple and easy to implement | Lower security compared to JWT/OAuth|
| Stateless (no session needed)| API Key can be leaked and reused     |
| Fast validation             | Does not identify actual user       |

---

## API Key vs JWT — What’s the Difference?

| Feature               | API Key                              | JWT                                      |
|-----------------------|------------------------------------|------------------------------------------|
| Nature                | Static unique key                   | Signed token containing claims           |
| Complexity            | Simple string comparison            | Token creation, signing, expiration       |
| Contains user info?   | No                                 | Yes (user identity, roles, permissions)  |
| Expiration            | Usually no expiration unless rotated| Has expiration time and refresh mechanisms|
| Typical Use Cases     | Service-to-service or simple auth   | User authentication with claims & roles  |

### Are They Replacements or Can They Work Together?

- **Replacement:** In simple scenarios, API Key can replace JWT.
- **Together:** In complex systems, API Key can identify the client app, and JWT can authenticate users inside that app. Both tokens can be sent together.

---

## Improved API Key Authentication in ASP.NET Core 9

Instead of manually checking API Keys with Middleware or Filters, .NET 9 allows you to implement API Key authentication using **Authentication Handlers** that integrate fully with the ASP.NET Core Authentication system.

### Benefits of Using Authentication Handlers

- Uses the built-in ASP.NET Core authentication system
- Supports `[Authorize]` attributes and policies
- Clean and modular code
- Easier integration with other authentication schemes (e.g., JWT)
- Standardized 401/403 responses

---

## Sample Code: API Key Authentication Handler in .NET 9

### 1. Define Options

```csharp
public class ApiKeyOptions : AuthenticationSchemeOptions
{
    public const string DefaultScheme = "ApiKeyScheme";
    public string Scheme => DefaultScheme;
    public string ApiKeyHeaderName { get; set; } = "X-API-KEY";
    public string ValidApiKey { get; set; } = "my-secret-key";
}
```

### 2. Create Authentication Handler

```csharp
using Microsoft.AspNetCore.Authentication;
using Microsoft.Extensions.Options;
using System.Security.Claims;
using System.Text.Encodings.Web;

public class ApiKeyAuthenticationHandler : AuthenticationHandler<ApiKeyOptions>
{
    public ApiKeyAuthenticationHandler(
        IOptionsMonitor<ApiKeyOptions> options,
        ILoggerFactory logger,
        UrlEncoder encoder,
        ISystemClock clock
    ) : base(options, logger, encoder, clock) { }

    protected override Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        if (!Request.Headers.TryGetValue(Options.ApiKeyHeaderName, out var apiKey))
        {
            return Task.FromResult(AuthenticateResult.Fail("API Key is missing."));
        }

        if (apiKey != Options.ValidApiKey)
        {
            return Task.FromResult(AuthenticateResult.Fail("Invalid API Key."));
        }

        var claims = new[]
        {
            new Claim(ClaimTypes.Name, "ApiKeyUser"),
            new Claim(ClaimTypes.Role, "ApiUser")
        };

        var identity = new ClaimsIdentity(claims, Options.Scheme);
        var principal = new ClaimsPrincipal(identity);
        var ticket = new AuthenticationTicket(principal, Options.Scheme);

        return Task.FromResult(AuthenticateResult.Success(ticket));
    }
}
```

### 3. Register Handler in Program.cs

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services
    .AddAuthentication(ApiKeyOptions.DefaultScheme)
    .AddScheme<ApiKeyOptions, ApiKeyAuthenticationHandler>(ApiKeyOptions.DefaultScheme, options =>
    {
        options.ValidApiKey = "super-secret-key"; // can be loaded from config or DB
    });

builder.Services.AddAuthorization();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

app.MapGet("/secure-data", [Authorize(AuthenticationSchemes = ApiKeyOptions.DefaultScheme)] () =>
{
    return "You have accessed secure data!";
});

app.Run();
```

### 4. Client Request Example

```
GET /secure-data
X-API-KEY: super-secret-key
```

---

## When and How Is `ApiKeyAuthenticationHandler` Invoked?

- ASP.NET Core executes authentication handlers inside the **Authentication Middleware** (`app.UseAuthentication()`).
- When a request targets an endpoint decorated with `[Authorize(AuthenticationSchemes = "ApiKeyScheme")]`, the system selects the `ApiKeyAuthenticationHandler`.
- The handler's `HandleAuthenticateAsync()` method runs.
- If the API Key is valid, a `ClaimsPrincipal` is created and assigned to `HttpContext.User`.
- Otherwise, a 401 Unauthorized response is returned.
- Then authorization policies are checked (`app.UseAuthorization()`).

---

## Request Pipeline Flow Diagram

```
┌──────────────────────────┐
│  HTTP Request Received   │
└─────────────┬────────────┘
              │
              ▼
     ┌───────────────────┐
     │  Other Middleware  │
     │ (Logging, CORS...) │
     └─────────┬─────────┘
               │
               ▼
     ┌───────────────────┐
     │ UseAuthentication │
     └─────────┬─────────┘
               │
               ▼
  Does Endpoint Require Authentication?
               │
       ┌───────┴────────┐
       │  Yes           │
       │                │
       ▼                │
┌────────────────────────────────────┐
│ Select Authentication Scheme (ApiKeyScheme) │
└─────────────┬──────────────────────┘
              │
              ▼
┌────────────────────────────────────────┐
│ Run ApiKeyAuthenticationHandler        │
│ → HandleAuthenticateAsync()            │
└─────────────┬──────────────────────────┘
              │
     Is API Key valid?
     ┌───────┴───────────┐
     │ Yes               │ No
     ▼                   ▼
┌───────────────┐   ┌─────────────────┐
│ ClaimsPrincipal│   │ Unauthorized(401)│
│ created       │   └─────────────────┘
└───────┬───────┘
        │
        ▼
┌───────────────────┐
│ UseAuthorization  │
└─────────┬─────────┘
          │
          ▼
┌──────────────────────┐
│ Endpoint Executed     │
└──────────────────────┘
```

---

## Summary

Using API Key Authentication Handlers in ASP.NET Core 9 provides a clean, modular, and standard way to secure your APIs. It integrates seamlessly with the ASP.NET Core authentication and authorization pipeline, making your security code maintainable and extensible.

---

## Advanced Example: Role-based API Key Authentication from Database

In more complex systems, you may need to store multiple API Keys with different permissions (roles). Here’s how to extend the basic example to support role-based authorization.

### 1. Define the Model and Repository

```csharp
public class ApiKeyRecord
{
    public string Key { get; set; }
    public string ClientName { get; set; }
    public string Role { get; set; }
}

public interface IApiKeyRepository
{
    ApiKeyRecord GetApiKey(string key);
}

public class InMemoryApiKeyRepository : IApiKeyRepository
{
    private readonly List<ApiKeyRecord> _apiKeys = new()
    {
        new ApiKeyRecord { Key = "key-service-a", ClientName = "ServiceA", Role = "Reader" },
        new ApiKeyRecord { Key = "key-service-b", ClientName = "ServiceB", Role = "Admin" }
    };

    public ApiKeyRecord GetApiKey(string key)
    {
        return _apiKeys.FirstOrDefault(k => k.Key == key);
    }
}
```

### 2. Authentication Handler with Dynamic Claims

```csharp
public class ApiKeyAuthenticationHandler : AuthenticationHandler<ApiKeyOptions>
{
    private readonly IApiKeyRepository _apiKeyRepo;

    public ApiKeyAuthenticationHandler(
        IOptionsMonitor<ApiKeyOptions> options,
        ILoggerFactory logger,
        UrlEncoder encoder,
        ISystemClock clock,
        IApiKeyRepository apiKeyRepo
    ) : base(options, logger, encoder, clock)
    {
        _apiKeyRepo = apiKeyRepo;
    }

    protected override Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        if (!Request.Headers.TryGetValue(Options.ApiKeyHeaderName, out var apiKeyValue))
        {
            return Task.FromResult(AuthenticateResult.Fail("API Key is missing."));
        }

        var apiKeyRecord = _apiKeyRepo.GetApiKey(apiKeyValue);

        if (apiKeyRecord == null)
        {
            return Task.FromResult(AuthenticateResult.Fail("Invalid API Key."));
        }

        var claims = new[]
        {
            new Claim(ClaimTypes.Name, apiKeyRecord.ClientName),
            new Claim(ClaimTypes.Role, apiKeyRecord.Role)
        };

        var identity = new ClaimsIdentity(claims, Options.Scheme);
        var principal = new ClaimsPrincipal(identity);
        var ticket = new AuthenticationTicket(principal, Options.Scheme);

        return Task.FromResult(AuthenticateResult.Success(ticket));
    }
}
```

### 3. Program.cs Setup

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddSingleton<IApiKeyRepository, InMemoryApiKeyRepository>();

builder.Services
    .AddAuthentication(ApiKeyOptions.DefaultScheme)
    .AddScheme<ApiKeyOptions, ApiKeyAuthenticationHandler>(ApiKeyOptions.DefaultScheme, options => { });

builder.Services.AddAuthorization();

var app = builder.Build();

app.UseAuthentication();
app.UseAuthorization();

// Secure endpoint (role-based)
app.MapGet("/admin-data", [Authorize(Roles = "Admin", AuthenticationSchemes = ApiKeyOptions.DefaultScheme)] () =>
{
    return "Only Admin role can see this data.";
});

app.MapGet("/read-data", [Authorize(Roles = "Reader", AuthenticationSchemes = ApiKeyOptions.DefaultScheme)] () =>
{
    return "Only Reader role can see this data.";
});

app.Run();
```

### 4. Testing

- Request with Admin role key:
```
GET /admin-data
X-API-KEY: key-service-b

✅ Works (Admin role matches endpoint requirement)
```

- Request with Reader role key:
```
GET /admin-data
X-API-KEY: key-service-a

❌ 403 Forbidden (API Key is valid but role does not match "Admin")
```

---

## Securing Endpoints with Roles

In `Program.cs`, you can restrict access based on roles assigned in claims:

```csharp
// Secure endpoint (role-based)
app.MapGet("/admin-data", [Authorize(Roles = "Admin", AuthenticationSchemes = ApiKeyOptions.DefaultScheme)] () =>
{
    return "Only Admin role can see this data.";
});

app.MapGet("/read-data", [Authorize(Roles = "Reader", AuthenticationSchemes = ApiKeyOptions.DefaultScheme)] () =>
{
    return "Only Reader role can see this data.";
});
```

---

## Understanding Role-based Access and API Keys

- `key-service-b` might be associated with the "Admin" role and can access `/admin-data`.
- `key-service-a` might be associated with the "Reader" role and cannot access `/admin-data`, resulting in `403 Forbidden`.
- Both keys can have different roles that affect what endpoints they can access.

---

## Additional Notes on Claims

You can add any claims you want in the handler, not just `Name` and `Role`. For example:

```csharp
var claims = new[]
{
    new Claim(ClaimTypes.Name, apiKeyRecord.ClientName),
    new Claim(ClaimTypes.Role, apiKeyRecord.Role),
    new Claim("CustomClaimType", "CustomValue")
};
```

This lets you pass extra metadata about the client or permissions.

## Question

In the API Key authentication method where we add Claims, previously Claims didn’t exist, right? So, API Key was simpler compared to JWT, and now by adding Claims it became richer? 

Previously, when only API Key was used, it was usually just a simple string checked for validity, with no user details or extra information. So basically, API Key was a "simple key" that just said "Is this request authorized or not?" but usually didn’t contain user identity or roles.

But when we combine this with ASP.NET Core Authentication Handlers and add Claims inside the `ClaimsPrincipal`, we are effectively adding a richer authentication step, similar to JWT which is full of various Claims.

Meaning:

- Previously API Key was just simple validation
- Now, by adding Claims in the `ApiKeyAuthenticationHandler`, we can include role, client name, and other info
- This makes API Key behave more like JWT, while still keeping its simplicity and speed

So we can say ASP.NET Core 9 has enhanced API Key from a simple key to a more "informative" authentication mechanism, without full JWT complexity.

### Summary:

API Key was simple before, now with Claims it gets closer to JWT but still lighter and faster.

---

## Follow-up Question

Now that it’s like this, what’s the need for JWT anymore?

Even though API Key with added Claims can meet many simple authentication needs, JWT still has strong reasons to be used:

### Why JWT is still needed?

- **Real user identification and richer info:**  
  JWT usually contains detailed user identity info (username, email, user ID) and complex permission levels. API Key is mostly for identifying services or clients.

- **Higher security:**  
  JWT is signed and sometimes encrypted, which prevents tampering or forgery. API Key is usually a fixed string that if leaked, can be misused.

- **Token management (Expiration & Refresh):**  
  JWT usually has expiration and supports refresh tokens. API Keys often don’t expire unless manually rotated.

- **Distributed, stateless authentication:**  
  JWT can be validated without a central database on multiple servers/services. API Key usually requires checking against a central store.

- **More flexibility:**  
  JWT can carry complex info easily and can be adapted to various policies and systems.

### In conclusion:

- **API Key:** simple, fast, suitable for service-to-service or client identification with simple auth  
- **JWT:** complex, secure, suitable for user authentication with detailed roles and claims

They can also be used together; for example, API Key to identify the service and JWT to identify the user within that service.
