# Setting up EF Core DbContext with SQL Server

This document explains the purpose and role of the following key elements in an ASP.NET Core application using Entity Framework Core with SQL Server.

---

## 📌 DbContext Registration in `Program.cs`

```csharp
// Register EF Core DbContext with SQL Server
builder.Services.AddDbContext<AuthDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));
```

### 🔍 What does this line do?

- **Registers `AuthDbContext` with the dependency injection (DI) container.**
- Configures it to use **SQL Server** as the database provider.
- Reads the **connection string** from `appsettings.json` under the key `"ConnectionStrings:DefaultConnection"`.

> Without this line, the application won’t know how to instantiate `AuthDbContext`, and runtime or migration operations will fail.

---

## ⚙️ Migration Commands

After registering the `DbContext`, we can manage the database schema using EF Core Migrations.

### 🛠 1. Add Migration

```bash
dotnet ef migrations add InitialCreate
```

- Captures the current model (defined by your entity classes and `DbContext`) and generates a migration file.
- This file contains `Up()` and `Down()` methods to apply or rollback schema changes.

### 🗃 2. Update Database

```bash
dotnet ef database update
```

- Applies the pending migrations to the database.
- Automatically creates the database if it doesn't exist.

---

## 📦 Required NuGet Packages

Make sure these packages are installed:

```bash
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet add package Microsoft.EntityFrameworkCore.Design
```

---

## 📝 Example `appsettings.json`

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=.;Database=SampleJwtApiDb;Trusted_Connection=True;TrustServerCertificate=True"
  }
}
```

---

## ✅ Summary

| Component | Purpose |
|----------|---------|
| `AddDbContext` | Registers the EF Core DbContext and configures SQL Server |
| `Add-Migration` | Generates migration files from model changes |
| `Update-Database` | Applies migrations and updates the actual database |

This setup enables EF Core to act as the data access layer for your ASP.NET Core app using SQL Server.
