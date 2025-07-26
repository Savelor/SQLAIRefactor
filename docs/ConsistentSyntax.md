## 1. SELECT *
When writing SQL queries, using SELECT * should be avoided. This statement reads all columns, increasing I/O and memory usage due to potentially unnecessary data retrieval. Instead, only select the columns really needed in the code. This approach reduces data load, improves query performance, and keeps your code cleaner. In the example below, the ‘*’ has been replaced with only the necessary columns: ProductID and LineTotal. 

<table>
  <tr>
    <td style="vertical-align: top; ">
      <h4 style = "margin: 2 px;">🔹 Discouraged</h4>
      <pre><code>
WITH alpha AS (
SELECT *
FROM Sales.SalesOrderDetail
WHERE SalesOrderID > 1500
)
SELECT ProductID, LineTotal
FROM Alpha
      </code></pre>
    </td>
    <td style="vertical-align: top;">
      <h4 style = "margin: 2 px;">🔹 Good practice</h4>
      <pre><code>
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

<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Old syntax</h4>
      <pre><code>
SELECT P.Name, P.ProductNumber, PM.ModifiedDate
FROM Production.Product P, Production.ProductModel PM
WHERE P.ProductModelID = PM.ProductModelID
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Good pratice</h4>
      <pre><code>
SELECT P.Name, P.ProductNumber, PM.ModifiedDate
FROM Production.Product P
INNER JOIN Production.ProductModel PM
ON P.ProductModelID = PM.ProductModelID
      </code></pre>
    </td>
  </tr>
</table>


## 3. ORDER BY / GROUP BY
When using ORDER BY or GROUP BY clauses, it is recommended to explicitly use column names instead of column position numbers. In ORDER BY, relying on numeric positions can lead to errors if the SELECT clause is later modified, changing the order of selected columns without updating the ORDER BY clause. This sorts the results set by unintended columns, potentially resulting in incorrect results and silent bug. Similar concept for GROUP BY. 

<table style="vertical-align: top;">
  <tr>
    <td style="vertical-align: top;">
      <h4>🔹 Discouraged</h4>
      <pre><code>
SELECT
 SOD.SalesOrderID,
 SOH.CustomerID,
 SOD.UnitPrice,
 SOH.OrderDate,
 SOD.OrderQty
FROM Sales.SalesOrderDetail SOD
INNER JOIN Sales.SalesOrderHeader SOH 
ON SOD.SalesOrderID = SOH.SalesOrderID
ORDER BY 
1, 2, 4 DESC
      </code></pre>
    </td>
    <td style="vertical-align: top;">
      <h4>🔹 Recommended</h4>
      <pre><code>
SELECT
 SOD.SalesOrderID,
 SOH.CustomerID,
 SOD.UnitPrice,
 SOH.OrderDate,
 SOD.OrderQty
FROM Sales.SalesOrderDetail SOD
INNER JOIN Sales.SalesOrderHeader SOH 
ON SOD.SalesOrderID = SOH.SalesOrderID
ORDER BY 
 SOD.SalesOrderID,
 SOH.CustomerID,
 SOH.OrderDate DESC
      </code></pre>
    </td>
  </tr>
</table>
