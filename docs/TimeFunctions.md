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
The comparison between the two actual execution plans shows exceptional reduction in all execution metrics:
<small>

| Metric            | Original Query | AI Refactored Query | Variation (%) |
|:------------------|---------------:|--------------------:|--------------:|
| Cost              | 0.496          | 0.006               | **-98.79% ✅**  |
| CPU Time [ms]     | 15             | 0                   | **-100.00% ✅** |
| Elapsed Time [ms] | 36             | 27                  | **-25.00% ✅**  |
| Logical Reads     | 358            | 3                   | **-99.16% ✅**  |

</small>


## DATEADD()
The DATEADD() function is often used to shift dates when filtering, but applying it directly to an indexed date column makes the query non-SARGable, since SQL Server must evaluate the function on every row. To keep the query SARGable, apply DATEADD() to the constant instead of the column, so the column can be searched directly with an Index Seek.
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

The comparison between the two actual execution plans shows exceptional reduction in all execution metrics:
<small>

| Metric            | Original Query | AI Refactored Query | Variation (%) |
|:------------------|---------------:|--------------------:|--------------:|
| Cost              | 0.496          | 0.006               | **-98.79% ✅**  |
| CPU Time [ms]     | 15             | 0                   | **-100.00% ✅** |
| Elapsed Time [ms] | 11             | 0                   | **-100.00% ✅** |
| Logical Reads     | 358            | 3                   | **-99.16% ✅**  |

</small>

  
## DATEPART()
The DATEPART() function extracts a part of a date (such as year or month), but applying it directly to an indexed column makes the query non-SARGable, preventing index seeks. To make the query SARGable, rewrite the condition using a date range that corresponds to the desired part, so SQL Server can efficiently use the index.
<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT UnitPrice, LineTotal
FROM Sales.SalesOrderDetail
WHERE DATEPART(YEAR, ModifiedDate) = 2025
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Sargable - Index Seek</h4>
      <pre><code>
SELECT UnitPrice, LineTotal
FROM Sales.SalesOrderDetail
WHERE ModifiedDate >= '2025-01-01' AND ModifiedDate < '2026-01-01'
      </code></pre>
    </td>
  </tr>
</table>

<div style="text-align: left;">
<img width="2339" height="454" alt="DatePART" src="https://github.com/user-attachments/assets/76be8db3-8a37-4e11-8642-831472e93146" />
</div>

The comparison between the two actual execution plans shows exceptional reduction in all execution metrics:
<small>

| Metric            | Original Query | AI Refactored Query | Variation (%) |
|:------------------|---------------:|--------------------:|--------------:|
| Cost              | 0.483          | 0.006               | **-98.76% ✅**  |
| CPU Time [ms]     | 16             | 0                   | **-100.00% ✅** |
| Elapsed Time [ms] | 12             | 0                   | **-100.00% ✅** |
| Logical Reads     | 358            | 3                   | **-99.16% ✅**  |

</small>


## YEAR()
Using the YEAR() function in the WHERE clause of a SQL query may seem simple and intuitive, but it's not sargable—meaning SQL Server cannot use an index efficiently, leading to slower performance, especially on large tables.
<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT UnitPrice, LineTotal
FROM Sales.SalesOrderDetail
WHERE YEAR(ModifiedDate) = 2025
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Sargable - Index Seek</h4>
      <pre><code>
SELECT UnitPrice, LineTotal
FROM Sales.SalesOrderDetail
WHERE ModifiedDate >= '2025-01-01' AND ModifiedDate <  '2026-01-01'
      </code></pre>
    </td>
  </tr>
</table>

<div style="text-align: left;">
<img width="2339" height="460" alt="YEAR" src="https://github.com/user-attachments/assets/15dc6fb6-83e7-4976-9702-406410b44db8" />
</div>

The comparison between the two actual execution plans shows exceptional reduction in all execution metrics:
<small>

| Metric            | Original Query | AI Refactored Query | Variation (%) |
|:------------------|---------------:|--------------------:|--------------:|
| Cost              | 0.483          | 0.006               | **-98.76% ✅**  |
| CPU Time [ms]     | 16             | 0                   | **-100.00% ✅** |
| Elapsed Time [ms] | 13             | 0                   | **-100.00% ✅** |
| Logical Reads     | 358            | 3                   | **-99.16% ✅**  |

</small>

