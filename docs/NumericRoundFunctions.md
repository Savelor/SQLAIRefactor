## CEILING()
The CEILING() function is often applied to numeric columns to round values up, but using it directly on an indexed column makes the query non-SARGable. Instead, move the arithmetic to the constant side of the predicate so the column can be evaluated directly, enabling SQL Server to use an Index Seek.

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


<div style="background: white; padding: 10px; margin: 0;">
Cost=0.45, CPU time=16 ms,  elapsed time=9 ms. LogicalReads=438 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cost=0.00,  CPU time=0 ms,  elapsed time=0 ms LogicalReads=3
</div>


## FLOOR()
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


<div style="background: white; font-family: Courier; padding: 10px; margin: 0;">
Cost=0.45, CPU time=16 ms,  elapsed time=7 ms. LogicalReads=438 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cost=0.00,  CPU time=0 ms,  elapsed time=0 ms LogicalReads=3
</div>



## ROUND()
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


<div style="background: white; font-family: Courier; padding: 10px; margin: 0;">
Cost=0.45, CPU time=16 ms,  elapsed time=12 ms. LogicalReads=438 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cost=0.00, CPU time=0 ms,  elapsed time=41 ms LogicalReads=3
</div>


## SIGN()
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


<div style="background: white; font-family: Courier; padding: 10px; margin: 0;">
Cost=0.34, CPU time=15 ms, elapsed time=363 ms. LogicalReads=274 &nbsp;&nbsp;&nbsp;&nbsp;Cost=0.23, CPU time=16 ms, elapsed time=347 ms LogicalReads=201
</div>
