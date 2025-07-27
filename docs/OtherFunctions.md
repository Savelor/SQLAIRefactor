## LIKE
WHERE predicates that include a LIKE clause typically result in execution plans that are difficult to optimize. However, rewriting the LIKE condition using functions such as RIGHT and PATINDEX can lead to significantly more efficient execution plans. Tests show that this approach can reduce I/O operations, especially as table size increases.

1. Wildcard on right, the plan is good enough:

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

<div style="text-align: left;">
<img width="432" height="202" alt="LIKE1" src="https://github.com/user-attachments/assets/64dacae3-ce39-42b4-8f8c-4d464932b5f9" />
</div>



2. Wildcard on left: it is possible to refactor using RIGHT function. Tests show the same execution plan in both versions, but the refactored code always shows better CPU performance and elapsed time

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
<img width="1865" height="278" alt="LIKE2" src="https://github.com/user-attachments/assets/6518eb12-dc7c-4a86-b2f4-abd26647b7e1" />
</div>

3. Wildcard on both sides:
   
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
   

