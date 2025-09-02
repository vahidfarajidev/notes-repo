# Handling File Upload Validation and Detailed Error Messages in ASP.NET 4.8

When developing file upload functionality in ASP.NET, it’s common to run into issues where invalid files cause generic server errors. For instance, attempting to upload a file that is too large, has an unsupported type, or contains invalid data may result in a **500 Internal Server Error**. This can be confusing for end users and makes debugging difficult.

This article explains how to handle file validation properly, display detailed error messages, and configure ASP.NET/IIS to respect your custom messages.

---

## Common Issues

1. **File Size Limits**  
   ASP.NET has default request size limits:
   ```xml
   <httpRuntime maxRequestLength="4096" /> <!-- size in KB -->
   ```
   IIS also has its own limits:
   ```xml
   <requestFiltering>
       <requestLimits maxAllowedContentLength="30000000" /> <!-- size in bytes -->
   </requestFiltering>
   ```

2. **Generic 500 Error**  
   By default, IIS may show:
   ```
   500 - Internal server error
   ```
   even if the file failed validation due to type or content.

3. **Bad Request (400) Responses**  
   Returning `StatusCode = 400` sometimes results in the browser displaying a generic "Bad Request" instead of your custom message.

---

## Handling File Validation in Code

Wrap your file upload logic in a `try/catch` block and throw custom exceptions for validation failures:

```csharp
try
{
    string filePath = Folder + "/" + filename;

    if (File.Exists(context.Server.MapPath(filePath)))
    {
        context.Response.StatusCode = 500;
        context.Response.Write("The file already exists.");
        return;
    }

    // Example: Validate image files
    if (FType == (int)Core.DomainValue.FileAttach.FileAttachType.Image)
    {
        if (!ImageTools.ImageFileTypes_WhiteList().Contains(MimeMapping.GetMimeMapping(filename)))
            throw new Infrastructure.Exception.CMSException(
                "Invalid file format",
                Infrastructure.Exception.ErrorCodes.Validation
            );

        var image = System.Drawing.Image.FromStream(postedFile.InputStream);
        ImageTools.CheckDataImage(image.Width, image.Height, postedFile.ContentLength);
    }
    // Add similar blocks for Doc, Audio, Video, Infographic types
}
catch (Infrastructure.Exception.CMSException ex)
{
    if (ex.ErrorNo == Infrastructure.Exception.ErrorCodes.Validation)
    {
        context.Response.StatusCode = 500;
        context.Response.Write("The file is invalid.");
        context.Response.End(); // ensure IIS does not overwrite response
    }
    return;
}
```

### Key Points:
- Validate **file type**, **size**, and **content** before saving.  
- Use `Response.Write()` and `Response.End()` to return custom messages.  
- Catch your custom validation exceptions to avoid exposing raw server errors.

---

## Configuring web.config for Detailed Errors

To prevent IIS from overriding your messages, set `httpErrors` with `existingResponse="PassThrough"`:

```xml
<system.webServer>
    <httpErrors errorMode="Custom" existingResponse="PassThrough" />
</system.webServer>
```

Also, make sure `httpRuntime` and `requestLimits` are set to allow the desired file size:

```xml
<system.web>
    <httpRuntime maxRequestLength="102400" executionTimeout="3600" targetFramework="4.8" />
</system.web>

<system.webServer>
    <security>
        <requestFiltering>
            <requestLimits maxAllowedContentLength="104857600" />
        </requestFiltering>
    </security>
</system.webServer>
```

---

## Summary

1. Wrap file upload code in `try/catch`.  
2. Throw meaningful exceptions for invalid files.  
3. Write custom error messages to `Response`.  
4. Configure `web.config` to let your messages pass through IIS.  

With these steps, users see clear validation errors like:

```
The file is invalid.
```

instead of generic `500 Internal Server Error` or browser-specific "Bad Request" messages.

---

By following this approach, you can provide a robust file upload experience in ASP.NET 4.8 while keeping both users and developers informed.

