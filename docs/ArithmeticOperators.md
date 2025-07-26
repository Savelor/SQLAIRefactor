
Simple arithmetic expressions can be written differently to force the execution plan to change from using a table or index Scan to Index Seek. The following example is on AdventurWorks2022. Since the SalesOrderID is multiplied, the condition SalesOrderID * 3 < 1200 may prevent SQL Server from effectively using the existing PK index. That’s because SQL Server evaluates the multiplication result and not the column data, ‘hiding’ the indexed data. It compares each single multiplication result to 1200, leading to a Scan on the column.

