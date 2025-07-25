## DATEDIFF()
The DATEDIFF function in SQL Server calculates the difference between two dates based on a specified date part (such as days, months, or years). It can be refactored using DATEADD on the right side of the operator.

<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT UnitPrice, LineTotal
FROM Sales.SalesOrderDetail
WHERE DATEDIFF(DAY, ModifiedDate, GETDATE()) < 21
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Sargable - Index Seek</h4>
      <pre><code>
SELECT UnitPrice, LineTotal
FROM Sales.SalesOrderDetail 
WHERE ModifiedDate >= DATEADD(DAY, -21, GETDATE()
      </code></pre>
    </td>
  </tr>
</table>

<div style="text-align: left;">
  <img src="https://github.com/user-attachments/assets/914fde7b-710f-4c92-87ba-3fbbbcbaa23e" alt="Time_1" style="width: 90%;">
</div>


<div style="background: white; font-family: Courier; padding: 10px; margin: 0;">
Cost=0.496, CPU time=16 ms,  elapsed time=29 ms.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cost=0.006,  CPU time=0 ms,  elapsed time=12 ms
</div>

## DATEADD()

<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT UnitPrice, LineTotal
FROM Sales.SalesOrderDetail
WHERE DATEADD(DAY, -21, ModifiedDate) >= '2025-07-15'
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Sargable - Index Seek</h4>
      <pre><code>
SELECT UnitPrice, LineTotal
FROM Sales.SalesOrderDetail
WHERE ModifiedDate >= DATEADD(DAY, 21, '2025-07-15')
      </code></pre>
    </td>
  </tr>
</table>

<div style="text-align: left;">
<img width="2353" height="462" alt="DateADD" src="https://github.com/user-attachments/assets/8abd1061-7fda-409e-8d8a-b06aff2b7f92" />
</div>

<div style="background: white; font-family: Courier; padding: 10px; margin: 0;">
Cost=0.496, CPU time=16 ms,  elapsed time=29 ms.&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cost=0.006,  CPU time=0 ms,  elapsed time=12 ms
</div>
  
## DATEPART()

## YEAR()


