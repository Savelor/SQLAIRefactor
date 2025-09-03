## CEILING()
The CEILING() function is often applied to numeric columns to round values up, but using it directly on an indexed column makes the query non-SARGable. Instead, move the arithmetic to the constant side of the predicate using a proper interval condition so the column can be evaluated directly, enabling SQL Server to use an Index Seek.

<table style="width: 100%;">
  <tr>
    <td style="width: 60%; vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT [Valid From], [Valid To]
FROM Dimension.City
WHERE CEILING([City Key]) = 350
      </code></pre>
    </td>
    <td style="width: 40%; vertical-align: top; padding: 10px;">
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

The actual execution plan metrics show a great improvement replacing CEILING with a proper expression:
<small>

| Metric            | Original Query | AI Refactored Query | Variation (%) |
|:------------------|---------------:|--------------------:|--------------:|
| Cost              | 0.45           | 0.00                | **-100.00%** |
| CPU Time [ms]     | 16             | 0                   | **-100.00%** |
| Elapsed Time [ms] | 9              | 0                   | **-100.00%** |
| Logical Reads     | 438            | 3                   | **-99.32%**  |

</small>



## FLOOR()
The FLOOR() function rounds numeric values down, but applying it directly to an indexed column prevents SQL Server from using the index efficiently, resulting in a non-SARGable query. To make it SARGable, rewrite the condition by shifting the arithmetic to the constant side, so the column remains untouched and eligible for an Index Seek.

<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT [Valid From], [Valid To]
FROM Dimension.City
WHERE FLOOR([City Key]) = 714
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Sargable - Index Seek</h4>
      <pre><code>
SELECT [Valid From], [Valid To]
FROM Dimension.City
WHERE [City Key] >= 714 AND [City Key] < 714 + 1
      </code></pre>
    </td>
  </tr>
</table>

<div style="text-align: left;">
<img width="2082" height="276" alt="FLOOR" src="https://github.com/user-attachments/assets/efabf352-6aa4-4c0b-bbbe-c81051982314" />
</div>

The actual execution plan metrics show a great improvement in this use case:
<small>

| Metric            | Original Query | AI Refactored Query | Variation (%) |
|:------------------|---------------:|--------------------:|--------------:|
| Cost              | 0.45           | 0.00                | **-100.00%** |
| CPU Time [ms]     | 16             | 0                   | **-100.00%** |
| Elapsed Time [ms] | 7              | 0                   | **-100.00%** |
| Logical Reads     | 438            | 3                   | **-99.32%**  |

</small>


## ROUND()
The ROUND() function is often used to normalize numeric values, but applying it directly to an indexed column makes the query non-SARGable, forcing SQL Server to scan instead of seek. To keep the query SARGable, rewrite the predicate using an equivalent interval condition, leaving the indexed column untouched for efficient index usage.
<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT [Valid From], [Valid To]
FROM Dimension.City
WHERE ROUND([City Key], 0) = 4
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Sargable - Index Seek</h4>
      <pre><code>
SELECT [Valid From], [Valid To]
FROM Dimension.City
WHERE [City Key] >= 3.5 AND [City Key] < 4.5
      </code></pre>
    </td>
  </tr>
</table>

<div style="text-align: left;">
<img width="2447" height="484" alt="ROUND" src="https://github.com/user-attachments/assets/666cf420-96da-4512-b73e-8f9c28e05127" />
</div>


<small>

| Metric            | Original Query | AI Refactored Query | Variation (%) |
|:------------------|---------------:|--------------------:|--------------:|
| Cost              | 0.45           | 0.00                | **-100.00%** |
| CPU Time [ms]     | 16             | 0                   | **-100.00%** |
| Elapsed Time [ms] | 18             | 38                  | **+111.11%** |
| Logical Reads     | 438            | 3                   | **-99.32%**  |

</small>



## SIGN()
The SIGN() function returns the sign of a numeric expression, but when applied directly to an indexed column it makes the query non-SARGable, blocking efficient index seeks. A better approach is to rewrite the condition with explicit ranges (e.g., < 0, = 0, > 0), so SQL Server can leverage the index properly.
<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT [TransactionID]
FROM Production.TransactionHistory
WHERE SIGN([ActualCost]) = 1
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Sargable - Index Seek</h4>
      <pre><code>
SELECT [TransactionID]
FROM Production.TransactionHistory
WHERE [ActualCost] > 0
      </code></pre>
    </td>
  </tr>
</table>

<div style="text-align: left;">
<img width="2186" height="283" alt="SIGN" src="https://github.com/user-attachments/assets/e41b425a-02fe-4d54-a238-016af8976110" />
</div>


<small>

| Metric            | Original Query | AI Refactored Query | Variation (%) |
|:------------------|---------------:|--------------------:|--------------:|
| Cost              | 0.34           | 0.23                | **-32.35%**  |
| CPU Time [ms]     | 15             | 16                  | **+6.67%**   |
| Elapsed Time [ms] | 363            | 347                 | **-4.41%**   |
| Logical Reads     | 274            | 201                 | **-26.64%**  |

</small>

