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
