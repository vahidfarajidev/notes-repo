# SQL Server Paging Guide

Paging is a common requirement in database queries to fetch a subset of results, often used in applications for pagination.

In SQL Server, `LIMIT` is **not used** (it's specific to MySQL). There are two main methods to implement paging:

---

## 1️⃣ Using `OFFSET ... FETCH NEXT`

This method is supported from **SQL Server 2012 onwards** and is the most common approach.

```sql
SELECT *
FROM Employees
ORDER BY EmployeeID
OFFSET 10 ROWS      -- Number of rows to skip
FETCH NEXT 5 ROWS ONLY;  -- Number of rows to fetch
```

- `OFFSET` specifies how many rows to skip (e.g., for the second page if each page has 10 rows, use `OFFSET 10 ROWS`).  
- `FETCH NEXT ... ROWS ONLY` specifies how many rows to retrieve for the current page.

---

## 2️⃣ Using `ROW_NUMBER()` for paging (before SQL Server 2012)

Before SQL Server 2012, `TOP` and `ROW_NUMBER()` in a CTE or subquery were commonly used. Below are two equivalent ways.

### Using CTE

```sql
WITH OrderedEmployees AS (
    SELECT *,
           ROW_NUMBER() OVER (ORDER BY EmployeeID) AS RowNum
    FROM Employees
)
SELECT *
FROM OrderedEmployees
WHERE RowNum BETWEEN 11 AND 15;  -- Fetch rows for the second page if page size is 5
```

### Using Subquery

```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER (ORDER BY EmployeeID) AS RowNum
    FROM Employees
) AS OrderedEmployees
WHERE RowNum BETWEEN 11 AND 15;  -- Fetch rows for the second page if page size is 5
```

- `ROW_NUMBER()` assigns a sequential number to rows based on the desired order.  
- The `WHERE` clause filters the rows for the desired page.  
- Subquery method is equivalent to CTE, just without using `WITH`.

---

💡 **Note on professional usage in 2025:**

In modern projects, `OFFSET ... FETCH NEXT` is generally preferred because it is:
- Officially supported and standard in SQL Server 2012+
- Cleaner and easier to read
- Well-optimized for typical paging scenarios

The `ROW_NUMBER()` method is still used for complex ranking or multi-level filtering, but for simple pagination, `OFFSET ... FETCH NEXT` is the common choice.

💡 **Note:** SQL Server does **not** support `LIMIT`. The modern and standard approach is `OFFSET ... FETCH NEXT`.
