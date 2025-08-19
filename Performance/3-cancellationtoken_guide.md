# Understanding CancellationToken in .NET

## Introduction

In modern .NET applications, especially those performing **long-running or asynchronous tasks**, it is often necessary to allow **tasks to be canceled gracefully**. This is where `CancellationToken` comes into play.

`CancellationToken` is a lightweight signaling mechanism used to **communicate a request for cancellation** from a caller to a task or operation.

---

## What is CancellationToken?

A `CancellationToken` does not cancel a task by itself. Instead, it **signals the intention** to cancel. It is up to the **callee** to monitor the token and respond appropriately.

Key properties and methods:

- `IsCancellationRequested` – Returns `true` if cancellation has been requested.  
- `ThrowIfCancellationRequested()` – Throws `OperationCanceledException` if cancellation is requested.  
- Typically obtained from a `CancellationTokenSource`.

---

## Creating and Using a CancellationToken

### Example 1: Manual Cancellation

```csharp
var cts = new CancellationTokenSource();
var token = cts.Token;

Task.Run(async () =>
{
    while (!token.IsCancellationRequested)
    {
        Console.WriteLine("Working...");
        await Task.Delay(1000);
    }

    Console.WriteLine("Task canceled!");
});

// Cancel after 5 seconds
await Task.Delay(5000);
cts.Cancel();
```

**Explanation:**

- `CancellationTokenSource` creates a token.  
- Task checks the token periodically.  
- Calling `cts.Cancel()` signals the task to stop.

---

### Example 2: Cancellation in Async Methods

```csharp
public async Task DownloadFileAsync(string url, string path, CancellationToken token)
{
    using var client = new HttpClient();
    var response = await client.GetAsync(url, HttpCompletionOption.ResponseHeadersRead, token);

    await using var stream = await response.Content.ReadAsStreamAsync(token);
    await using var fileStream = File.Create(path);

    await stream.CopyToAsync(fileStream, 81920, token);
}
```

Here, passing the token to methods like `GetAsync` and `CopyToAsync` ensures the operation can be **interrupted externally**.

---

## CancellationToken in ASP.NET Core

ASP.NET Core automatically provides a token to each HTTP request via `HttpContext.RequestAborted`. This allows long-running requests to **cancel automatically** if the client disconnects.

```csharp
[HttpGet("data")]
public async Task<IActionResult> GetData(CancellationToken token)
{
    var data = await _service.FetchBigData(token);
    return Ok(data);
}
```

> Note: Even if the controller method does not explicitly check the token, passing it to async operations ensures proper cancellation.

---

## When to Check the Token Directly

Direct checking of the token is necessary when:

- The method has a **long-running loop**.  
- Internal API calls **do not accept a CancellationToken**.  

Example:

```csharp
public async Task ProcessDataAsync(CancellationToken token)
{
    while (true)
    {
        token.ThrowIfCancellationRequested();
        await Task.Delay(1000); // Simulate work
    }
}
```

---

## Best Practices

1. **Always pass CancellationToken to async operations** that support it.  
2. **Check the token in long loops** if operations themselves are not token-aware.  
3. **Use CancellationTokenSource for manual cancellation** or combine with timeouts.  
4. **Do not ignore the token**: failing to handle it may result in blocked tasks or wasted resources.  
5. **Dispose CancellationTokenSource** if it is no longer needed to free resources.

---

## Conclusion

`CancellationToken` is a fundamental tool in .NET for **graceful task cancellation**. Proper use of cancellation tokens improves:

- Responsiveness of applications  
- Resource management  
- Stability in long-running or asynchronous operations  

By integrating `CancellationToken` in your methods and loops, you can ensure your .NET applications **respond quickly and safely to cancellation requests**.

---

## References

- [Microsoft Docs: Cancellation in Managed Threads](https://learn.microsoft.com/en-us/dotnet/standard/threading/cancellation-in-managed-threads)  
- [Microsoft Docs: IHostApplicationLifetime](https://learn.microsoft.com/en-us/dotnet/api/microsoft.extensions.hosting.ihostapplicationlifetime)  
- [Async/Await and Cancellation](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/async/cancellation)

