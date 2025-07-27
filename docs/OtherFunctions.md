## LIKE
WHERE predicates that include a LIKE clause typically result in execution plans that are difficult to optimize. However, rewriting the LIKE condition using functions such as RIGHT and PATINDEX can lead to significantly more efficient execution plans. Tests show that this approach can reduce I/O operations, especially as table size increases.

1. **Wildcard on right:**, the plan is good enough, no need to rewrite it.

<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Sargable - Index Seek</h4>
      <pre><code>
SELECT AddressId, PostalCode, City
FROM Person.Address
WHERE AddressLine1 Like 'way%'
      </code></pre>
    </td>
  </tr>
</table>

<img src="https://github.com/user-attachments/assets/64dacae3-ce39-42b4-8f8c-4d464932b5f9" alt="LIKE1" width="239" height="121" />

<div style="background: white; padding: 10px; margin: 0; margin-bottom: 20px">
Cost=0.00, CPU time=0 ms,  elapsed time=0 ms, LogicalReads=3 
</div>
<p>&nbsp;</p>

2. **Wildcard on left:** it is possible to refactor using RIGHT function. Tests show the same execution plan in both versions, but the refactored code always shows better CPU performance and elapsed time

<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT AddressId, PostalCode, City
FROM Person.Address
WHERE AddressLine1 Like '%way'
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable, but better performance</h4>
      <pre><code>
SELECT AddressId, PostalCode, City
FROM Person.Address
WHERE RIGHT(AddressLine1,3) = 'way'
      </code></pre>
    </td>
  </tr>
</table>


<div style="text-align: left;">
<img width="1278"  alt="LIKE2" src="https://github.com/user-attachments/assets/217506dc-baa3-4c01-b6b3-679aab64b290" />
</div>

<div style="background: white; padding: 10px; margin: 0;">
Cost=0.18, CPU time=32 ms,  elapsed time=75 ms, LogicalReads=216 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cost=0.18,  CPU time=0 ms, elapsed time=56 ms LogicalReads=216
</div>

<p>&nbsp;</p>


3. **Wildcard on both sides:**
   it is possible to refactor using PATHINDEX function. Tests show the same execution plan in both versions, but the refactored code always shows better CPU performance and elapsed time. Here some examples on AdventureWorks2022
   
<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT AddressId, PostalCode, City
FROM Person.Address
WHERE AddressLine1 Like '%way%'
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable, but better performance</h4>
      <pre><code>
SELECT AddressId, PostalCode, City
FROM Person.Address
WHERE CHARINDEX('way', AddressLine1) > 0
--Alternatively: WHERE PATINDEX('%way%', AddressLine1) > 0
      </code></pre>
    </td>
  </tr>
</table>

<img src="https://github.com/user-attachments/assets/e0d6afcf-7c74-4f3a-ae0d-bfb595eae779" alt="LIKE3" width="1049" height="191" />

<div style="background: white; padding: 10px; margin: 0;">
Cost=0.18, CPU time=16 ms,  elapsed time=110 ms, LogicalReads=216 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cost=0.18, CPU time=31 ms, elapsed time=91 ms LogicalReads=216
</div>

## ISNULL
The first example is almost an error:
Supponiamo che qui essita un indice sulla colonna:
CREATE NONCLUSTERED INDEX IX1_CarrierTrackingNumber ON [Sales].[SalesOrderDetail] ([CarrierTrackingNumber])
**Primo esempio** 

<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable - Index Scan</h4>
      <pre><code>
SELECT * from [Sales].[SalesOrderDetail]
where ISNULL(CarrierTrackingNumber, '') = '4911-403C-98'
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 NOT Sargable, but better performance</h4>
      <pre><code>
SELECT * from [Sales].[SalesOrderDetail]
where CarrierTrackingNumber = '4911-403C-98'
      </code></pre>
    </td>
  </tr>
</table>

<img width="2413" height="465" alt="ISNULL1" src="https://github.com/user-attachments/assets/8519fc9d-70c7-4e29-9d24-03d2a8fa856e" />

<div style="background: white; padding: 10px; margin: 0;">
Cost=0.60, CPU time=15 ms,  elapsed time=74 ms, LogicalReads=492 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Cost=0.05, CPU time=0 ms, elapsed time=70 ms LogicalReads=39
</div>


## COALESCE

