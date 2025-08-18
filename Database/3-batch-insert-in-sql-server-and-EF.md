# Batch Insert in SQL Server and Entity Framework

This article explains how to perform **Batch Insert** in SQL Server and Entity Framework, its benefits, and how transactions affect the process.

---

## Multi-Row INSERT in SQL Server

You can insert multiple rows in a single SQL statement:

```sql
INSERT INTO Users (Name) VALUES ('Ali'), ('Sara'), ('Reza');
```

**Batch Insert** means sending all records in a single INSERT statement instead of executing a separate INSERT for each record.

### Details

* `'Ali'`, `'Sara'`, `'Reza'` → three records to insert into the `Users` table.
* Only **one INSERT statement** is executed covering all three rows.
* This ensures **a single round-trip to the SQL Server**, and all rows are inserted at once.

### Benefits

* Reduces the number of round-trips between the application and the database.
* Faster than executing three separate INSERT statements.
* Consumes fewer server and network resources.

---

## Batch Insert in Entity Framework

In Entity Framework, you can perform Batch Insert by **adding multiple entities to the DbContext and calling a single `SaveChanges()`**.

### Example in C#

```csharp
using (var context = new AppDbContext())
{
    // Create multiple new records
    var users = new List<User>
    {
        new User { Name = "Ali" },
        new User { Name = "Sara" },
        new User { Name = "Reza" }
    };

    // Add all records to the DbContext
    context.Users.AddRange(users);

    // Save all changes to the database at once
    await context.SaveChangesAsync();
}
```

### Step-by-Step Explanation

1. `AddRange(users)` → adds all entities to the DbContext, not yet in the database.
2. `SaveChangesAsync()` → sends all INSERTs as a **Batch** to the database.
3. **Benefits**: fewer round-trips, faster response times, lower resource usage.

### Transaction Behavior

* EF usually creates an **internal transaction** during `SaveChanges()`.
* **All-or-Nothing**: if one insert fails, EF rolls back all inserts to maintain database consistency.
* Without explicit transaction, SQL Server still ensures all-or-nothing for a multi-row INSERT.

---

## Multi-Row INSERT Version Support

* Introduced in **SQL Server 2008**.
* Before SQL Server 2008, multiple rows required multiple INSERT statements:

```sql
INSERT INTO Users (Name) VALUES ('Ali');
INSERT INTO Users (Name) VALUES ('Sara');
INSERT INTO Users (Name) VALUES ('Reza');
```

* After 2008, Multi-Row INSERT reduces round-trips and improves performance.

---

## Transaction Behavior in SQL Server

* If executed **inside a transaction**: all rows are inserted or none if any error occurs.
* If executed **without an explicit transaction**:
  * SQL Server uses an **internal autocommit transaction** for the statement.
  * If any row violates constraints (e.g., NULL in NOT NULL column), the entire statement fails.

Example:

```sql
INSERT INTO Users (Name, Age) 
VALUES ('Ali', 25), ('Sara', NULL), ('Reza', 30);
```

* If `Age` is NOT NULL, **no rows are inserted**.
* Ensures database remains consistent with no partial data inserted.

---

## Batch Insert Using ADO.NET (No Explicit Transaction)

Even without `BeginTransaction`:

* Sending a multi-row INSERT to SQL Server uses an **internal short transaction**.
* If any row fails, the **whole statement fails**.
* Behavior is **all-or-nothing**, no manual transaction required.

---

This approach ensures:

* Fewer database round-trips
* Faster performance
* Database consistency
