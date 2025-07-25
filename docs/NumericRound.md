When used in a WHERE clause, these functions can prevent the SQL Server engine from utilizing indexes effectively, often resulting in an Index Scan instead of a more efficient Index Seek. From a performance standpoint, these predicates should be rewritten using equivalent arithmetic conditions that preserve index usage and allow the optimizer to choose an Index Seek. In the following example in **WorldWideImportersDW** database, the table City has a PK index on [City Key] column. Refactoring the WHERE condition shows a query cost, I/O, CPU and elapsed time decreases by 99%, changing the plan from an Index Scan to an Index Seek. [Rules 10.5/6]

## CEILING()




## FLOOR()



## ROUND()



## SIGN()
