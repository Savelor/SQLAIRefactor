## ABS()
The ABS() function returns the absolute value of a number, but applying it to an indexed column makes the query non-SARGable, since SQL Server must compute the function on every row. To make the query SARGable, rewrite the condition using equivalent range predicates (e.g., Column >= -value AND Column <= value), allowing the optimizer to use an Index Seek.
<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT ReferenceOrderLineID
FROM Production.TransactionHistory
WHERE ABS(ReferenceOrderID) > 50
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Sargable - Index Seek</h4>
      <pre><code>
SELECT ReferenceOrderLineID 
FROM Production.TransactionHistory 
WHERE ReferenceOrderID > 50 OR ReferenceOrderID < -50
      </code></pre>
    </td>
  </tr>
</table>

<div style="text-align: left;">
<img width="1561" height="268" alt="ABS" src="https://github.com/user-attachments/assets/088e19a5-2572-4481-8d9e-a14f9b7256c9" />
</div>


<small>

| Metric            | Original Query | AI Refactored Query | Variation (%) |
|:------------------|---------------:|--------------------:|--------------:|
| Cost              | 0.32           | 0.32                | **0.00%**    |
| CPU Time [ms]     | 31             | 25                  | **-19.35% ✅**  |
| Elapsed Time [ms] | 461            | 453                 | **-1.74%**   |
| Logical Reads     | 255            | 258                 | **+1.18%**   |

</small>



## SQRT()
The SQRT() function calculates the square root of a number, but when applied to an indexed column it makes the query non-SARGable, preventing the use of efficient index seeks. To keep it SARGable, rewrite the condition by squaring the constant value instead of applying SQRT() to the column, leaving the indexed column untouched.
<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT [MovementDate] 
FROM [dbo].[FactProductInventory] 
WHERE SQRT([UnitCost]) < 10
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Sargable - Index Seek</h4>
      <pre><code>
SELECT [MovementDate] 
FROM [dbo].[FactProductInventory] 
WHERE UnitCost < POWER(10,2)
      </code></pre>
    </td>
  </tr>
</table>

<div style="text-align: left;">
<img width="2108" height="277" alt="SQRT" src="https://github.com/user-attachments/assets/ae01d0f0-8a1e-4c73-b4ab-d2f1b47f6408" />
</div>

The actual plan comparison shows a great improvement in this use case on all execution metrics:
<small>

| Metric            | Original Query | AI Refactored Query | Variation (%) |
|:------------------|---------------:|--------------------:|--------------:|
| Cost              | 3.00           | 0.32                | **-89.33% ✅**  |
| CPU Time [ms]     | 156            | 62                  | **-60.26% ✅**  |
| Elapsed Time [ms] | 2059           | 1822                | **-11.52% ✅**  |
| Logical Reads     | 2647           | 1549                | **-41.48% ✅**  |

</small>



## POWER()
The POWER() function raises a number to a given exponent, but using it directly on an indexed column makes the query non-SARGable, forcing SQL Server to scan the index. To make the query SARGable, rewrite the predicate by applying the exponent to the constant side instead, keeping the indexed column untouched and enabling an Index Seek.
<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT [MovementDate] 
FROM [dbo].[FactProductInventory] 
WHERE POWER(UnitCost,2) < 400
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Sargable - Index Seek</h4>
      <pre><code>
SELECT [MovementDate] 
FROM [dbo].[FactProductInventory] 
WHERE UnitCost > -SQRT(400) AND UnitCost < SQRT(400)
      </code></pre>
    </td>
  </tr>
</table>

<div style="text-align: left;">
<img width="2439" height="474" alt="Power" src="https://github.com/user-attachments/assets/a9039508-be38-453d-a4e3-61cf62b5e907" />
</div>

The actual plan comparison shows a great reduction in cost and I/O:
<small>

| Metric            | Original Query | AI Refactored Query | Variation (%) |
|:------------------|---------------:|--------------------:|--------------:|
| Cost              | 3.04           | 1.76                | **-42.11% ✅**  |
| CPU Time [ms]     | 156            | 140                 | **-10.26% ✅**  |
| Elapsed Time [ms] | 1525           | 1540                | **+0.98%**   |
| Logical Reads     | 2650           | 1241                | **-53.17% ✅**  |

</small>

