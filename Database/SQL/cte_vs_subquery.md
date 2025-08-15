# Understanding CTE and Subquery in SQL Server

In SQL Server, there are two common ways to structure intermediate query results: **CTE (Common Table Expression)** and **Subquery**. Both can be used for paging, ranking, or complex calculations.

## 1️⃣ Common Table Expression (CTE)

A CTE is a temporary result set that exists only within the execution of a single query. It is defined using `WITH` and is often used to improve readability and manage complex queries.

### Example:
```sql
WITH OrderedEmployees AS (
    SELECT *,
           ROW_NUMBER() OVER (ORDER BY EmployeeID) AS RowNum
    FROM Employees
)
SELECT *
FROM OrderedEmployees
WHERE RowNum BETWEEN 11 AND 15;  -- Fetch rows for the third page if page size is 5
```

- `OrderedEmployees` is the CTE name.
- Inside the parentheses, we select from the main table (`Employees`) and add a row number (`ROW_NUMBER`) for paging.
- Then the main query selects only the desired rows.

**Advantages:**
- High readability
- Can be reused multiple times in the same query
- Supports recursive queries and step-by-step calculations

---

## 2️⃣ Subquery

A Subquery is a query written **inside another query**. Its output is used by the outer query.

### Equivalent Subquery for the above CTE:
```sql
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER (ORDER BY EmployeeID) AS RowNum
    FROM Employees
) AS OrderedEmployees
WHERE RowNum BETWEEN 11 AND 15;
```

- The inner query (`SELECT ... FROM Employees`) does the same as the CTE but inline.
- Alias (`AS OrderedEmployees`) is required to reference it in the outer query.

---

## 3️⃣ Differences Between CTE and Subquery

| Feature               | CTE                                | Subquery                        |
|----------------------|-----------------------------------|---------------------------------|
| Readability          | Higher                            | Lower, especially in long queries |
| Reusability          | Can be used multiple times        | Usually used once               |
| Recursive Support    | Yes                               | Usually no                      |
| Scope                | Within the current query only     | Only where it is written        |

**Note:**
- Performance-wise, CTE and Subquery are often similar because SQL Server converts both into execution plans.
- The choice between CTE and Subquery is mostly about **code readability and maintainability**, not performance.
