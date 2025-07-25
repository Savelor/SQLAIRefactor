## ABS()


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


<div style="background: white; font-family: Courier; padding: 10px; margin: 0;">
Cost=0.32, CPU time=15 ms,  elapsed time=454 ms. logicalReads=262 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cost=0.32,  CPU time=16 ms,  elapsed time=452 ms logicalReads=262
</div>


## SQRT()

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


<div style="background: white; font-family: Courier; padding: 10px; margin: 0;">
Cost=3.00, CPU time=156 ms,  elapsed time=2059 ms. logicalReads=2647 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cost=0.32,  CPU time=62 ms,  elapsed time=1822 ms logicalReads=1549
</div>


## POWER()

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
<img width="2439" height="474" alt="Power" src="https://github.com/user-attachments/assets/55b3bef1-0f38-453b-ab65-994221537fef" />
</div>

<div style="background: white; font-family: Courier; padding: 10px; margin: 0;">
Cost=3.04, CPU time=156 ms,  elapsed time=1525 ms. logicalReads=2650 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cost=1.76,  CPU time=140 ms,  elapsed time=1540 ms logicalReads=1241
</div>
