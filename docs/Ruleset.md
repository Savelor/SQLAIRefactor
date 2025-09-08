## Prompt ruleset

1. `When you find a statement such as “SELECT * FROM Expression”, read all the batch, and identify which columns of that Expression are really used in the batch after that SELECT statement. 
Replace the “*” in the SELECT with only the columns you found as really used. If reading the code below the “SELECT *” statement it is not possible to understand which columns 
are really used, don’t make any assumption and replace the * with all column’s names of the table. You have the list of columns of all tables provided to you in JSON format.`

2. `Identify the SELECT statements that uses old syntax implicit joins, meaning that multiple tables are specified in the FROM clause separated by commas and the join columns in the WHERE clause. Rewrite these statements and use explicit join syntax with proper ‘JOIN’ keyword and specify the join columns with keyword ‘ON’.
Ensure that all table relationships and filter conditions remain logically equivalent.`

3. `When an ORDER BY clause is written using the column numbers (for example: ORDER BY 1, 3, 5) rewrite the ORDER BY clause replacing the numbers with column names used in the SELECT clause.`

4. `When you find WHERE expression containing functions on column such as: CEILING(column) = x, or FLOOR(column) = x, or ROUND(column) don’t apply any function on the column. Rewrite the WHERE expression leaving the column alone on left side of the comparison and rewrite an equivalent condition on x modifying the right side of the comparison. Apply the following rules:
a) “WHERE CEILING(UnitPrice) = 714” must be rewritten as “WHERE UnitPrice > 713 AND UnitPrice <= 714”
b) “WHERE FLOOR(UnitPrice) = 714” must be rewritten as “WHERE UnitPrice >= 714 AND UnitPrice < 715”
c) “WHERE ROUND(UnitPrice) = 714” must be rewritten as “WHERE UnitPrice BETWEEN 714 – 0.5 AND UnitPrice < 714 + 0.5”`

5. `If the provided code contains a WHERE condition with the SIGN(column) function applied to a column, rewrite the query avoiding to apply any function to the column, and make the condition SARGable. Apply the following rules:
a) “WHERE SIGN(SalesOrderID) = 1” must be rewritten as  “WHERE SalesOrderID > 0”
b) “WHERE SIGN(SalesOrderID) = -1” must be rewritten as “WHERE SalesOrderID < 0”
c) “WHERE SIGN(SalesOrderID) = 0” must be rewritten as “WHERE SalesOrderID = 0”`

6. `When the WHERE expression contains functions on column such as: ABS(column), SQRT(column) or POWER(column,2) having the column as parameter, rewrite the WHERE expression leaving the column alone on left side of the comparison and rewrite an equivalent condition modifying the right side of the comparison as shown in the following examples. Apply the following rules:
a) “WHERE ABS(level) < value”  must be rewritten as “WHERE level < value AND level > -value”
b) “WHERE SQRT(column) <  value” must be rewritten as: “WHERE column < POWER(value,2)”
c) “WHERE POWER(column,2) <  value” must be rewritten as: “WHERE column < SQRT(value)”
d) consider that SQRT(POWER(value,2)) = value`

7. `In a WHERE comparison clause, if there are calculations or transformations on a column, then leave the column alone on left side and move the equivalent calculation on the right. On the left side of the comparison there should appear only the column without any operator applied. This ensures better performance, as it allows SQL Server to optimize index usage effectively.
Examples:
a) avoid “WHERE price * 12 = 550“ replace with “WHERE price = 550/12”
b) avoid “WHERE price + 12 = 550“ replace with “WHERE price = 550-12”
c) avoid “WHERE price / 12 = 550“ replace with “WHERE price = 550*12”
d) avoid “WHERE price - 12 = 550“ replace with “WHERE price = 550+12”`

8. `If the WHERE condition is an equation-like expression, rewrite this T-SQL WHERE clause to make it SARGable by isolating the column on left side of the comparison. Apply the basic algebraic manipulation equation principles to isolate the column on left side. Verify that the rewritten condition has the same mathematical meaning.`

9. `When you find WHERE conditions containing LIKE operator such as: LIKE ‘%text’ or LIKE ‘%text%’, apply the following rules:
a) If the string to search has ‘%’ at the beginning of the string to search, use the RIGHT function instead. For example replace: “WHERE AddressLine LIKE ‘%way’” with: “WHERE RIGHT(AddressLine,3) = ‘way’”
b) If the string to search is between two wildcards‘%’, use the PATINDEX function instead. For example replace: “WHERE AddressLine LIKE '%way%'” with: “WHERE CHARINDEX(‘way’, AddressLine, 0) > 0”
c) If the string to search has wildcard '%' at the end of the string, don’t modify this condition. For example: “WHERE AddressLine LIKE 'way%'” don’t modify that.
d) If the WHERE condition contains both conditions: “column LIKE '%way'” AND “column LIKE 'way%'” then rewrite as “column = 'way'”`

10. `When you find WHERE expression containing function DATEADD(column) applied to a column, rewrite the WHERE expression leaving the column alone on left side of the comparison and rewrite an equivalent condition modifying the right side of the comparison. Apply the following rule:
“WHERE DATEADD(DAY, 30, ModifiedDate) >= ‘2024-01-01′” must be rewritten as: “WHERE ModifiedDate >= DATEADD(DAY, -30, '2024-01-01')”`

11. `When you find WHERE expression containing function DATEPART(column) applied tp a column together with 'YEAR' parameter, rewrite the WHERE expression leaving the column alone on left side of the comparison and rewrite an equivalent condition modifying the right side of the comparison. Apply the following rule:
“WHERE DATEPART(YEAR, ModifiedDate) = 2013” must be rewritten as: “WHERE ModifiedDate >= '2013-01-01' AND ModifiedDate < '2014-01-01'”.`

12. `When you find WHERE expression with comparisons containing DATEDIFF function applied to a column, rewrite the WHERE expression leaving the column alone on left side of the comparison and rewrite an equivalent condition modifying the right side of the comparison using DATEADD function. Apply the following rules:
a) “WHERE DATEDIFF(day, ModifiedDate, '2011-05-31') <= 5” must be rewritten as: “WHERE ModifiedDate >= DATEADD(day, -5, '2011-05-31')”
b) “WHERE DATEDIFF(month, '2011-06-30', ModifiedDate) > 5” must be rewritten as: “WHERE ModifiedDate > DATEADD(month, 5, '2011-06-30')”.
c) “WHERE DATEDIFF(year, '2011-06-30', ModifiedDate) > 5” must be rewritten as: “WHERE ModifiedDate > DATEADD(year, 5, '2011-06-30')”.` 

13. `If in the WHERE clause the YEAR function is applied to a column, rewrite the statement using range comparison according to the following example:
“YEAR(OrderDate) = 2022” must be rewritten as: “OrderDate >= '2022-01-01' AND OrderDate < '2023-01-01'”.`

14. `If the provided code contains a WHERE condition with the ISNULL() function applied to a column rewrite the query to avoid the function and make the condition SARGable. Apply the following rule:
"WHERE ISNULL(ColumnX, Value) > 23" If Value <> 23 then rewrite as "WHERE ColumnX > 23"`

15. `Analyze the provided SQL batch code carefully to identify UNUSED elements such as variables or items that are declared in DECLARE statement and after such declaration they don’t appear in the code anymore. Before you decide something is unused, check that if it’s really not used after DECLARE declaration line. Return the query rewritten by removing all unused items that you identified from the code. Unused items can be:
a) Remove variables or table variables never used after the creation. 
b) Remove temporary tables created in the code and never used after the creation. `

16. `Always analyze all input parameters of a stored procedure or function definition. If an input parameter is declared but never used within the body (i.e., it does not appear in any logic, condition, assignment, or query), it is considered redundant and must be removed. Refactor this exact procedure without adding or removing logic.`

17. `In a multi line function, check the entire flow of the SQL code from start to finish. Determine which elements do not contribute to the final RETURN statement (redundant or irrelevant items). Remove redundant or irrelevant items them from the body of the function. So apply the following rules:
a) Remove unused or irrelevant function parameters from multi line function definition.
b) Remove variables or table variables or temporary tables not related to the result.
c) Remove irrelevant pieces of code not contributing to the Returned value. `

18. `If the provided code contains the declaration of a temporary table or table variable having columns of data types: TEXT, NTEXT, or IMAGE, then replace those columns with the modern equivalent data types:
a) Replace TEXT with VARCHAR(MAX)
b) Replace NTEXT with NVARCHAR(MAX)
c) Replace IMAGE with VARBINARY(MAX)
Ensure the new version of batch uses either a temporary table or a table variable with the updated data types.`

19. `Identify all WHERE conditions containing comparisons between column and variable or parameter, using operators such as: =, >, >=, <, or <=. Determine the column’s data type based on the provided database schema information provided to you in JSON format and determine the variable’s or 
parameter’s data type by analyzing the code. If the column and variable have different data types, rewrite the WHERE condition leaving the column alone on left side of the comparison without any conversion. 
On the right side apply the CONVERT() function to the variable to match the column’s data type or declare the variable with the same data type as the column. 
At the end, column and variable must have exactly the same data type.`

20. `Identify all WHERE conditions containing comparisons between column and constants or literals, using operators such as: =, >, >=, <, or <=. Determine the column’s data type based on the provided JSON data and determine the literal or constant data type. 
If the column and constant have different data types, rewrite the WHERE condition leaving the column alone on left side of the comparison without any conversion. 
On the right side apply the CONVERT() function to the constant or literal to match the column’s data type. At the end, column and constant must have exactly the same data type.`

21. `Analyze the provided code and identify dynamic SQL executions such as:  "EXEC @query" or "EXEC sp_executesql @query" without parameters. Identify only the cases with the command string @query is unvalidated AND it is built by concatenating multiple strings or variables. In this cases rewrite the code using one of the following two alternative options:
a) keep EXEC @query, and add to the code additional check validations on sql string. Add the verification that the command string @query doesn't contain suspicious keywords "DROP", "DELETE", ";", "UPDATE". In addition report a warning comment in the modified code.
b) Replace EXEC with EXEC sp_executesql @query, passing parameters instead concatenating parameters in the @query command string.
c) If the argument of EXEC function is a fixed query not built concatenating any variable, just remove exec and execute the argument statement.`

22. `Identify all WHERE conditions containing comparisons with COALESCE(column1, column2) function applied to 2 table columns.
Rewrite the WHERE condition leaving the column alone on left side of the comparison. Apply according to the following example:
""WHERE COALESCE(LastName, FirstName) = 'James'""  must be rewritten as "WHERE LastName = 'James' OR (LastName IS NULL AND FirstName = 'James')"`

23. `Identify all WHERE conditions containing comparisons with COALESCE(column1, column2) function applied to 2 table columns.
Rewrite the WHERE condition leaving the column alone on left side of the comparison. Apply according to the following example:
""WHERE COALESCE(LastName, FirstName) = 'James'""  must be rewritten as "WHERE LastName = 'James' OR (LastName IS NULL AND FirstName = 'James')"`

24. `Analyze SQL code and identify dynamic SQL execution patterns such as EXEC @query_stmt or EXEC sp_executesql @query_stmt. Focus on cases where the query string is built using string concatenation and is not properly validated (SQL injection)
Identify only the cases with the command string @query_stmt is unvalidated AND it is built by concatenating multiple strings or variables without checking the content. In this cases rewrite the code using one of the following alternative options:
a) keep "EXEC @query", and add to to the code additional check validations on sql string. Add the verification that the command string @query doesn’t contain suspicious keywords “TRUNCATE”, “DROP”, “DELETE”, “;”, “UPDATE”. In addition report a warning comment in the modified code.
b) Replace ‘EXEC @query’ with ‘EXEC sp_executesql @query’, passing parameters instead of concatenating parameters inside the @query string.
c) If dynamic SQL includes user-supplied identifiers (e.g., table names, schemas), apply QUOTENAME() to each part individually before including them in the SQL string. This prevents injection by ensuring only valid SQL identifiers are accepted.
d) If the argument of EXEC function is a fixed query not built concatenating any variable, just remove exec and execute the argument statement.`

24. `When you find a cursor then you can often rewrite the logic as a single SELECT or a CTE or a WHILE loop if ALL the following a) and b) conditions are true:
a) No procedural or stateful logic is needed in the cursor, and the cursor is just iterating over rows to perform: Filtering or Aggregation or Ranking or Calculations based on other rows, 
b) No procedural or stateful logic is needed: no Calling stored procedures per row, no Modifying data conditionally depending on prior rows, No Maintaining complex row-dependent state.`

25. `When you find a cursor and it cannot be replaced with a simple query, verify that at the end the cursor is properly closed with both CLOSE and DEALLOCATE statements. If these instructions are missing, rewrite the cursor including them.`

26. `When a variable or table variable is written with INSERT or UPDATE, verify that after that write statement the variable is really used in the code. If not drop the INSERT or UPDATE statement.`

27. `When within a stored procedure a temporary table is created or loaded with data and then it is not used later in the code, rewrite the stored procedure without that temporary table.`

28. `When the WHERE condition contains a CONVERT on a column compared to a value (e.g., CONVERT(INT, ProductId) = 21222000), detect the column’s data type and rewrite the condition by placing the column alone on the left side and converting the value on the right side to the column’s data type.`

29. `When the WHERE condition contains a CAST on a column compared to a value (e.g., CAST(ProductId AS INT) = 21222000), detect the column’s data type and rewrite the condition by placing the column alone on the left side and casting the value on the right side to the column’s data type.`

30. `If a query contains an OUTER APPLY with a subquery that returns zero or more rows per outer row, does not contain TOP, ORDER BY, aggregate functions, window functions, or nested subqueries, and whose WHERE clause consists solely of equality conditions correlating the outer and inner tables, 
then rewrite the OUTER APPLY as a LEFT JOIN using the same correlation conditions in the ON clause, and update all references from the OUTER APPLY alias to the joined table alias.`

31. `If you find case like "IF (SELECT COUNT(*) > 0)" with a simple subquery verifying if rows returned > 0 then rewrite the case using EXISTS predicate with SELECT 1 in the subquery.`

32. `If a local temporary table (i.e., a table whose name starts with #) is declared inside a stored procedure and is used exclusively for either Write operations 
(such as INSERT, SELECT INTO, UPDATE, or DELETE) or read operations (such as SELECT, JOIN, or usage in expressions), but NOT both, then the table has no meaningful effect on 
the procedure's logic. In such cases, the table must be considered unused and should be entirely removed from the procedure body, including its declaration and all associated references. 
This rule does not apply if the temporary table is passed to dynamic SQL, referenced in nested stored procedures, or returned explicitly in the procedure's output. 
A valid temporary table must have at least one write and one read operation within the procedure body to be considered purposeful.`

33. `If in a SQL batch a table variable Es: DECLARE @t TABLE (col1 INT) is declared and is used exclusively for either Write operations (such as INSERT, SELECT INTO, UPDATE, or DELETE) or read operations (such as SELECT, JOIN, or usage in expressions), but NOT both, then the table variable has no meaningful effect on the batch logic. In such cases, the table must be considered unused and should be entirely removed from the batch, including its declaration and all associated references. A valid table variable must have at least one write and one read operation within the batch body to be considered purposeful.`

