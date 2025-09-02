# Handling File Upload Validation and Detailed Errors in ASP.NET 4.8

A common but flawed approach in older ASP.NET applications is to return `500 Internal Server Error` for all file upload issues. This is not recommended because it mixes different error scenarios under a single status code and does not inform the user about the specific problem.

## Why Returning 500 for All Errors is a Wrong Approach

Previously, developers often did something like this:

```csharp
context.Response.StatusCode = 500;
context.Response.Write("File is invalid or duplicate");
```

This approach is problematic because:
- `500` indicates a server-side failure, but most file upload issues are client-side validation errors.
- Users cannot distinguish between duplicate files, invalid files, or other errors.
- It does not follow HTTP best practices for meaningful status codes.

## Updated Approach with Proper Error Codes

Instead of returning `500`, we now use:
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

To ensure IIS does not override these custom error messages, we update `web.config`:

```xml
<system.webServer>
  <httpErrors errorMode="Custom" existingResponse="PassThrough" />
</system.webServer>
```

This preserves the correct status codes and error messages sent to the client.

## Summary

- Returning `500` for all errors is a flawed approach.
- Use `400` for validation errors and `409` for duplicates.
- Configure IIS with `existingResponse="PassThrough"` to maintain custom messages.

This method provides clear, actionable feedback for clients and adheres to HTTP best practices.
