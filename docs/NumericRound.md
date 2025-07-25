When used in a WHERE clause, these functions can prevent the SQL Server engine from utilizing indexes effectively, often resulting in an Index Scan instead of a more efficient Index Seek. From a performance standpoint, these predicates should be rewritten using equivalent arithmetic conditions that preserve index usage and allow the optimizer to choose an Index Seek. In the following example in **WorldWideImportersDW** database, the table City has a PK index on [City Key] column. Refactoring the WHERE condition shows a query cost, I/O, CPU and elapsed time decreases by 99%, changing the plan from an Index Scan to an Index Seek. [Rules 10.5/6]

## CEILING()
<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT [Valid From], [Valid To]
FROM Dimension.City
WHERE CEILING([City Key]) = 350
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Sargable - Index Seek</h4>
      <pre><code>
SELECT [Valid From], [Valid To]
FROM Dimension.City
WHERE [City Key] > 350-1 AND [City Key] <= 350
      </code></pre>
    </td>
  </tr>
</table>

<div style="text-align: left;">
<img width="2122" height="279" alt="CEILING" src="https://github.com/user-attachments/assets/207018fb-84a5-40a5-ae14-d23ff6de3def" />
</div>


<div style="background: white; font-family: Courier; padding: 10px; margin: 0;">
Cost=0.45, CPU time=16 ms,  elapsed time=9 ms. LogicalReads=438 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cost=0.00,  CPU time=0 ms,  elapsed time=0 ms LogicalReads=3
</div>



## FLOOR()



## ROUND()



## SIGN()
