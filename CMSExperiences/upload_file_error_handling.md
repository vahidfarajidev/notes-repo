# Handling File Upload Validation and Detailed Errors in ASP.NET 4.8

In older ASP.NET applications, it is common to return `500 Internal Server Error` for all file upload issues. This is not ideal because users cannot understand the reason for failure, and it mixes different error scenarios under a single status code.

## The Old Approach (Problematic)

Previously, all errors during file upload, such as invalid file types, oversized files, or duplicates, returned `500`.

```csharp
context.Response.StatusCode = 500;
context.Response.Write("File is invalid or duplicate");
```

This approach is not recommended because:
- `500` indicates a server problem, not a client-side validation error.
- Users cannot distinguish between duplicate files, invalid files, or other errors.

## Updated Approach with Proper Error Codes

We updated the code to use specific HTTP status codes and detailed messages:

- **400 Bad Request**: For invalid file types or failed validation.
- **409 Conflict**: For duplicate files.

```csharp
try
{
    string FilePath = Folder + "/" + filename;
    if (File.Exists(context.Server.MapPath(FilePath)))
    {
        context.Response.StatusCode = 409; // Duplicate file
        context.Response.Write("The file already exists.");
        return;
    }

    if (FType == (int)Core.DomainValue.FileAttach.FileAttachType.Image)
    {
        if (!ImageTools.ImageFileTypes_WhiteList().Contains(MimeMapping.GetMimeMapping(filename)))
            throw new Infrastructure.Exception.CMSException("", Infrastructure.Exception.ErrorCodes.Validation);
        System.Drawing.Image Image = System.Drawing.Image.FromStream(postedFile.InputStream);
        ImageTools.CheckDataImage(Image.Width, Image.Height, postedFile.ContentLength);
    }
    else
    {
        context.Response.StatusCode = 400; // Invalid file type
        context.Response.Write("Invalid file.");
        return;
    }
}
catch (Infrastructure.Exception.CMSException ex)
{
    if (ex.ErrorNo == Infrastructure.Exception.ErrorCodes.Validation)
    {
        context.Response.StatusCode = 400;
        context.Response.Write("The file is invalid.");
    }
    return;
}
```

## IIS Configuration Update

To make sure IIS does not override our custom error messages, we updated the `web.config`:

```xml
<system.webServer>
  <httpErrors errorMode="Custom" existingResponse="PassThrough" />
</system.webServer>
```

This ensures that responses like `"The file is invalid."` or `"The file already exists."` are returned directly to the client, preserving the correct status codes and messages.

## Summary

- Returning `500` for all errors is misleading and not user-friendly.
- Use `400` for validation errors and `409` for duplicates.
- Update IIS with `existingResponse="PassThrough"` to keep custom messages intact.

This approach provides clear, actionable feedback for clients and aligns with HTTP best practices.
