## Avoiding cursors

The inefficiency of cursors is a classic pain point in SQL Server performance. Cursors allow you to iterate through rows one by one, similar to a loop in procedural languages. While this might feel intuitive for developers coming from imperative programming backgrounds, cursors are usually inefficient for several reasons:
Row-by-row processing: SQL Server is optimized for set-based operations. Cursors break this model.
Resource intensive: They require memory and locks, and often spill to tempdb.
Slow performance: For large result sets, performance degrades dramatically compared to set-based
You can often refactor cursor-based code by rewriting it without explicitly defining a cursor, relying instead on standard T-SQL constructs. Below are some common approaches.

**📝 Set-Based Queries:** This approach is typically valid when the cursor is simple, meaning that it only reads rows from a query result, it performs operations that can be expressed as aggregations, joins, or window functions, it does not depend on row-by-row side effects and the logic for one row does not depend on the state of previous iterations.

```sql
DECLARE @CustomerID INT, @OrderDate DATE, @TotalDue MONEY, @PrevCustomerID INT = NULL, @RunningTotal MONEY = 0;
DECLARE @Results TABLE ( CustomerID INT, OrderDate DATE, TotalDue MONEY, RunningTotal MONEY);
DECLARE order_cursor CURSOR FOR
    SELECT CustomerID, OrderDate, SubTotal AS TotalDue
    FROM Sales.SalesOrderHeader
    WHERE SubTotal > 1000
    ORDER BY CustomerID, OrderDate;

OPEN order_cursor;
FETCH NEXT FROM order_cursor INTO @CustomerID, @OrderDate, @TotalDue;

WHILE @@FETCH_STATUS = 0
BEGIN
    IF @CustomerID <> @PrevCustomerID
        SET @RunningTotal = 0; -- reset when customer changes

    SET @RunningTotal += @TotalDue;

    INSERT INTO @Results (CustomerID, OrderDate, TotalDue, RunningTotal)
    VALUES (@CustomerID, @OrderDate, @TotalDue, @RunningTotal);

    SET @PrevCustomerID = @CustomerID;
    FETCH NEXT FROM order_cursor INTO @CustomerID, @OrderDate, @TotalDue;
END;

CLOSE order_cursor;
DEALLOCATE order_cursor;
-- Return result set
SELECT * FROM @Results;
```
**Refactored version:**
```sql
SELECT  CustomerID, OrderDate, SubTotal AS TotalDue,
    SUM(SubTotal) OVER (
        PARTITION BY CustomerID
        ORDER BY OrderDate
        ROWS UNBOUNDED PRECEDING
    ) AS RunningTotal
FROM Sales.SalesOrderHeader
WHERE SubTotal > 1000
ORDER BY CustomerID, OrderDate;
```

**📝 CTE:** In general, a cursor can be rewritten as a CTE when the cursor’s purpose is primarily row sequencing, grouping, or computing derived columns rather than performing complex procedural operations per row.  The cursor should not perform row-by-row external actions, and the logic for each iteration should not depend on the results of previous iterations.

```sql
DECLARE @CustomerID INT, @OrderDate DATE, @TotalDue MONEY, @PrevCustomerID INT = NULL, @PrevDate DATE = NULL, @RunningTotal MONEY = 0;
DECLARE @Results TABLE ( CustomerID INT, OrderDate DATE, TotalDue MONEY, RunningTotal MONEY );
DECLARE order_cursor CURSOR FOR
    SELECT CustomerID, OrderDate, TotalDue
    FROM Sales.SalesOrderHeader
    WHERE TotalDue > 1000
    ORDER BY CustomerID, OrderDate;

OPEN order_cursor;
FETCH NEXT FROM order_cursor INTO @CustomerID, @OrderDate, @TotalDue;

WHILE @@FETCH_STATUS = 0
BEGIN
    IF @CustomerID <> @PrevCustomerID OR DATEDIFF(DAY, @PrevDate, @OrderDate) > 30
    BEGIN
        SET @RunningTotal = 0;  -- Reset running total if customer changes or gap > 30 days
    END

    SET @RunningTotal += @TotalDue;

    INSERT INTO @Results (CustomerID, OrderDate, TotalDue, RunningTotal)
    VALUES (@CustomerID, @OrderDate, @TotalDue, @RunningTotal);

    -- Save previous values for next row comparison
    SET @PrevCustomerID = @CustomerID;
    SET @PrevDate = @OrderDate;

    FETCH NEXT FROM order_cursor INTO @CustomerID, @OrderDate, @TotalDue;
END
CLOSE order_cursor;
DEALLOCATE order_cursor;

-- Return result set
SELECT * FROM @Results;
```

**Refactored version:**
```sql
-- Created by Copilot in SSMS - review carefully before executing
DECLARE @Results TABLE ( CustomerID INT, OrderDate DATE, TotalDue MONEY, RunningTotal MONEY);

-- Using a set-based approach with a window function to calculate running totals
WITH OrderedData AS (
    SELECT 
        CustomerID,
        OrderDate,
        TotalDue,
        LAG(CustomerID) OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS PrevCustomerID,
        LAG(OrderDate) OVER (PARTITION BY CustomerID ORDER BY OrderDate) AS PrevDate
    FROM  Sales.SalesOrderHeader
    WHERE TotalDue > 1000
),
RunningTotals AS (
    SELECT 
        CustomerID,
        OrderDate,
        TotalDue,
        CASE 
            WHEN CustomerID <> PrevCustomerID OR DATEDIFF(DAY, PrevDate, OrderDate) > 30 THEN TotalDue
            ELSE SUM(TotalDue) OVER (PARTITION BY CustomerID ORDER BY OrderDate ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
        END AS RunningTotal
    FROM  OrderedData
)
INSERT INTO @Results (CustomerID, OrderDate, TotalDue, RunningTotal)
SELECT  CustomerID, OrderDate, TotalDue, RunningTotal
FROM RunningTotals;

-- Return result set
SELECT * FROM @Results;
```



