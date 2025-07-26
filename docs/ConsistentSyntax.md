## 1. SELECT *
When writing SQL queries, using SELECT * should be avoided. This statement reads all columns, increasing I/O and memory usage due to potentially unnecessary data retrieval. Instead, only select the columns really needed in the code. This approach reduces data load, improves query performance, and keeps your code cleaner. In the example below, the query has been changed and the ‘*’ has been replaced with only the necessary columns: ProductID and LineTotal. 

<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Not optimized</h4>
      <pre><code>
--discouraged
WITH alpha AS (
SELECT *
FROM Sales.SalesOrderDetail
WHERE SalesOrderID > 1500
)
SELECT ProductID, LineTotal
FROM Alpha
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Optimized</h4>
      <pre><code>
--good practice
WITH alpha AS (
SELECT ProductID, LineTotal
FROM Sales.SalesOrderDetail
WHERE SalesOrderID > 1500
)
SELECT ProductID, LineTotal
FROM Alpha
      </code></pre>
    </td>
  </tr>
</table>

   
## 2. OLD JOIN syntax
Old-style implicit joins combine join and filter conditions in the WHERE clause, making the query harder to read and maintain. The second version below uses explicit INNER JOIN syntax, which clearly separates join logic from filtering, enhancing clarity and structure. SQL Server process both with the same plan, but explicit JOINs are best practice because they make queries easier to understand, maintain, and extend.
   
## 3. ORDER BY / GROUP BY
When using ORDER BY or GROUP BY clauses, it is recommended to explicitly use column names instead of column position numbers. In ORDER BY, relying on numeric positions can lead to errors if the SELECT clause is later modified, changing the order of selected columns without updating the ORDER BY clause. This sorts the results set by unintended columns, potentially resulting in incorrect results and silent bug. Similar concept for GROUP BY. [Rules 10.3/4]
