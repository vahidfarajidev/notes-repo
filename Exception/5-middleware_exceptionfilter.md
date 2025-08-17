# Exception Handling in ASP.NET Core: Middleware vs ExceptionFilter

In ASP.NET Core applications, handling exceptions consistently is crucial for maintaining clean architecture, proper logging, and reliable responses to clients. There are two main approaches:

1. **Exception Handling Middleware** (global/general)
2. **Exception Filters** (controller/action-level)

Below we explore their behavior, differences, and integration with examples.

---

## 1. Exception Handling Middleware

Middleware can catch any exception that occurs in the request pipeline. A common pattern is to centralize exception-to-HTTP response mapping here.

### Example Middleware:

```csharp
using System.Net;
using System.Text.Json;
using BankingApi.Domain;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.Logging;

public class ExceptionHandlingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ExceptionHandlingMiddleware> _logger;

    public ExceptionHandlingMiddleware(RequestDelegate next, ILogger<ExceptionHandlingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (DomainException ex) when (ex is AccountNotFoundException)
        {
            // This log complements the central MediatR logging pipeline
            // Logs resource-not-found domain exceptions with warning level
            _logger.LogWarning(
                ex,
                "Resource not found for HTTP request {Method} {Path} with status {StatusCode} and message: {Message}",
                context.Request.Method,
                context.Request.Path,
                (int)HttpStatusCode.NotFound, // HTTP 404
                ex.Message
            );

            // Set HTTP response for client: HTTP 404 Not Found
            await WriteErrorResponseAsync(context, HttpStatusCode.NotFound, ex.Message);
        }
        catch (DomainException ex)
        {
            _logger.LogWarning(
                ex,
                "Domain exception occurred for HTTP request {Method} {Path} with status {StatusCode} and message: {Message}",
                context.Request.Method,
                context.Request.Path,
                (int)HttpStatusCode.BadRequest, // HTTP 400
                ex.Message
            );

            // Set HTTP response for client: HTTP 400 Bad Request
            await WriteErrorResponseAsync(context, HttpStatusCode.BadRequest, ex.Message);
        }
        catch (Exception ex)
        {
            _logger.LogError(
                ex,
                "Unhandled exception occurred for HTTP request {Method} {Path} with status {StatusCode}",
                context.Request.Method,
                context.Request.Path,
                (int)HttpStatusCode.InternalServerError // HTTP 500
            );

            // Set HTTP response for client: HTTP 500 Internal Server Error
            await WriteErrorResponseAsync(context, HttpStatusCode.InternalServerError, "An unexpected error occurred");
        }
    }

    private static async Task WriteErrorResponseAsync(HttpContext context, HttpStatusCode statusCode, string message)
    {
        // Set HTTP response status code and content type
        context.Response.ContentType = "application/json";
        context.Response.StatusCode = (int)statusCode;

        var errorResponse = new
        {
            error = message,
            status = context.Response.StatusCode
        };

        await context.Response.WriteAsync(JsonSerializer.Serialize(errorResponse));
    }
}
```

### Key Points about Middleware

- Middleware **catches exceptions** globally.
- It typically logs exceptions **as a complement to centralized logging**, e.g., MediatR pipeline.
- **Maps exceptions to HTTP responses**:
  - 400 for domain violations
  - 404 for resource not found
  - 500 for system errors
- **Once the middleware handles an exception, it stops the propagation**, so exception filters downstream will not see it unless the middleware rethrows the exception.

#### Passing Exceptions to Filters

If you want exception filters to handle it instead:

```csharp
catch (DomainException ex)
{
    // Allow filter to handle it
    throw;
}
```

---

## 2. Exception Filters

Exception Filters (`ExceptionFilterAttribute`) are **controller/action-level** filters that catch unhandled exceptions in MVC controllers.

### Example Filter:

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Filters;
using System.Net;

public class ApiExceptionFilter : ExceptionFilterAttribute
{
    public override void OnException(ExceptionContext context)
    {
        if (context.Exception is DomainException)
        {
            context.Result = new ObjectResult(new { error = context.Exception.Message })
            {
                StatusCode = (int)HttpStatusCode.BadRequest
            };
        }
        else
        {
            context.Result = new ObjectResult(new { error = "Internal server error" })
            {
                StatusCode = (int)HttpStatusCode.InternalServerError
            };
        }
        context.ExceptionHandled = true;
    }
}
```

### Key Points about Exception Filters

- Filters are executed **after controller/action** code throws an exception.
- They can **return custom HTTP responses**.
- **They do not handle exceptions globally**—only those that propagate to controllers.
- **If middleware already handles exceptions**, the filter **never runs**, unless middleware allows exceptions to propagate (e.g., via `throw;`).

---

## 3. Middleware vs Exception Filters

| Feature               | Middleware                        | Exception Filter                                                               |
| --------------------- | --------------------------------- | ------------------------------------------------------------------------------ |
| Scope                 | Global (entire pipeline)          | Controller / action                                                            |
| Handles               | All exceptions                    | Exceptions reaching controller                                                 |
| Logging               | Often complements central logging | Can log, but usually not used in modern architectures with centralized logging |
| HTTP Response Mapping | Yes (common)                      | Yes                                                                            |
| Propagation           | Stops pipeline unless rethrown    | Only sees exceptions passed down                                               |

---

## 4. Recommendations

- **Use Middleware for global exception handling** in modern ASP.NET Core applications.
- **Use Exception Filters** only for **controller-specific or legacy scenarios**.
- Ensure **centralized logging** (e.g., via MediatR pipeline behavior) rather than logging in both middleware and filters.
- If you want a mix, **middleware must allow exceptions to propagate** by rethrowing them for filters to handle.

---

## 5. Summary

- Middleware and filters both can convert exceptions to HTTP responses.
- Middleware is **global** and should usually handle all exceptions and log them **as a complement**.
- Exception filters are **more local** and mostly used in MVC controllers.
- Exception propagation is key: once middleware handles an exception fully, filters never see it.

This ensures **clean separation of concerns**, **consistent logging**, and **reliable HTTP responses** in ASP.NET Core applications.

