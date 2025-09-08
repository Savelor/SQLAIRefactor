## Arithmetic Operators
Simple arithmetic expressions can be written differently to force the execution plan to change from using a table or index Scan to Index Seek. The following example is on AdventurWorks2022: table SalesOrderdetail has the Clustered index PK_SalesOrderDetail_SalesOrderID_SalesOrderDetailID. Since the SalesOrderID is multiplied, the condition SalesOrderID * 3 < 1200 may prevent SQL Server from effectively using the existing PK index. That’s because SQL Server evaluates the multiplication result and not the column data, ‘hiding’ the indexed data. It compares each single multiplication result to 1200, leading to a Scan on the column.

<table style="width: 100%;">
  <tr>
    <td style="width: 60%; vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT UnitPrice, OrderQty 
FROM Sales.SalesOrderDetail
WHERE SalesOrderID * 3 < 1200
      </code></pre>
    </td>
    <td style="width: 40%; vertical-align: top; padding: 10px;">
      <h4>🔹 Sargable - Index Seek</h4>
      <pre><code>
SELECT UnitPrice, OrderQty 
FROM Sales.SalesOrderDetail
WHERE SalesOrderID < 1200 / 3
      </code></pre>
    </td>
  </tr>
</table>

<div style="text-align: left;">
<img width="2553" height="455" alt="Arithmetic" src="https://github.com/user-attachments/assets/4bdaf2a6-bbed-439a-b146-45c43708f4e2" />
</div>

The comparison between the two actual execution plans shows exceptional improvements on all execution metrics:
<small>

| Metric            | Original Query | AI Refactored Query | Variation (%) |
|:------------------|---------------:|--------------------:|--------------:|
| Cost              | 0.422          | 0.003               | **-99.29% ✅**  |
| CPU Time [ms]     | 15             | 0                   | **-100.00% ✅** |
| Elapsed Time [ms] | 9              | 0                   | **-100.00% ✅** |
| Logical Reads     | 273            | 3                   | **-98.90% ✅**  |

</small>

This use case also addresses the following conditions in WHERE clauses:

| Not Sargable - Index Scan    | Sargable - Index Seek                      |
|-------------|-------------------------------------|
| WHERE SalesOrderID / 3 < 1200       | WHERE SalesOrderID < 1200 * 3      |
| WHERE SalesOrderID + 3 < 1200      | WHERE SalesOrderID < 1200 – 3          |
| WHERE SalesOrderID – 3 < 1200     | WHERE SalesOrderID < 1200 + 3       |


## Simple equations
In cases where the WHERE predicate is a linear combination of an indexed column and numeric constants, it can be rewritten by applying the basic principles of linear equations: isolating the column on the left-hand side and moving the constant terms to the right-hand side. If properly instructed, the AI model is perfectly able to do that.

<table style="width: 100%;">
  <tr>
    <td style="width: 60%; vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT UnitPrice, OrderQty 
FROM Sales.SalesOrderDetail
WHERE SalesOrderID * 3 - 10 < 1200
      </code></pre>
    </td>
    <td style="width: 40%; vertical-align: top; padding: 10px;">
      <h4>🔹 Sargable - Index Seek</h4>
      <pre><code>
SELECT UnitPrice, OrderQty
FROM Sales.SalesOrderDetail
WHERE SalesOrderID < ((1200 + 10.0) / 3.0)
      </code></pre>
    </td>
  </tr>
</table>

<img width="1851" height="469" alt="LinearEquations" src="https://github.com/user-attachments/assets/30f99eaa-0ca5-4c7c-a8b4-8f52086761e8" />

The comparison between the two actual execution plans shows exceptional improvements on the AI optimized code on all execution metrics:
<small>

| Metric            | Original Query | AI Refactored Query | Variation (%) |
|:------------------|---------------:|--------------------:|--------------:|
| Cost              | 0.446          | 0.003               | **-99.33% ✅**  |
| CPU Time [ms]     | 31             | 0                   | **-100.00% ✅** |
| Elapsed Time [ms] | 22             | 0                   | **-100.00% ✅** |
| Logical Reads     | 273            | 3                   | **-98.90% ✅**  |

</small>



