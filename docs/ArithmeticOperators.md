## Arithmetic Operators
Simple arithmetic expressions can be written differently to force the execution plan to change from using a table or index Scan to Index Seek. The following example is on AdventurWorks2022. Since the SalesOrderID is multiplied, the condition SalesOrderID * 3 < 1200 may prevent SQL Server from effectively using the existing PK index. That’s because SQL Server evaluates the multiplication result and not the column data, ‘hiding’ the indexed data. It compares each single multiplication result to 1200, leading to a Scan on the column.

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
