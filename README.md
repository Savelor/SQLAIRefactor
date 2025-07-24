This repository is the accompanying code for the "AI-based T-SQL Refactoring: an automatic intelligent code optimization with Azure OpenAI" article. Make sure that check that out.

# SQLAIRefactor

**SQLAIRefactor** is a Windows Forms application that leverages Azure OpenAI to analyze and optimize T-SQL queries. It connects to your SQL Server database, extracts schema metadata in JSON format, and uses prompt engineering and large language models to refactor queries and apply SQL Server best practices automatically.

<div align="center">
  <img src="https://github.com/user-attachments/assets/17562648-a598-42ba-b090-a55c0645b6ce" alt="Architecture" width="600"/>
</div>

This solution is an AI-powered application to automating SQL Server code analysis and refactoring. The system intelligently identifies inefficiencies and common T-SQL anti-patterns, applying best practices through a set of formalized coding rules, using prompt-driven instructions. It not only automatically rewrites problematic and inefficient code but also delivers contextual recommendations to improve quality, security, and maintainability.
<div align="center">
  <img src="https://github.com/user-attachments/assets/a29fac2d-d02e-4257-a2fe-d4cad9d7d4d7" alt="GUI1" width="700"/>
</div>
Designed to address real-world use cases, this soltion enables organizations to modernize and optimize their SQL workloads more efficiently, accelerating migrations and reducing manual effort in scenarios with large SQL codebases. This technique can represent an advancement in SQL Server application development and maintenance, reducing manual effort and improving code quality and performance.

<div align="center">
  <img src="https://github.com/user-attachments/assets/d89b924b-0726-4010-9cbe-d257d593b11d" alt="Example" width="600"/>
</div>



## 🚀 Features

- AI-powered SQL refactoring using GPT-4.1 or GPT-4o (Azure OpenAI)
- Retrieves and injects full table/column data types in JSON
- Identifies inefficiencies (e.g., implicit conversions, index scan vs. seek)
- Supports both Windows and SQL Authentication
- Renders results in an HTML-based view with syntax highlighting

---
## 📦 Theory

## 1. SELECT *
When writing SQL queries, using SELECT * should be avoided. This statement reads all columns, increasing I/O and memory usage due to potentially unnecessary data retrieval. Instead, only select the columns really needed in the code. This approach reduces data load, improves query performance, and keeps your code cleaner. In the example below, the query has been changed and the ‘*’ has been replaced with only the necessary columns: ProductID and LineTotal. 

## 2. OLD JOIN syntax
Old-style implicit joins combine join and filter conditions in the WHERE clause, making the query harder to read and maintain. The second version below uses explicit INNER JOIN syntax, which clearly separates join logic from filtering, enhancing clarity and structure. SQL Server process both with the same plan, but explicit JOINs are best practice because they make queries easier to understand, maintain, and extend.
   
## 3. ORDER BY / GROUP BY
When using ORDER BY or GROUP BY clauses, it is recommended to explicitly use column names instead of column position numbers. In ORDER BY, relying on numeric positions can lead to errors if the SELECT clause is later modified, changing the order of selected columns without updating the ORDER BY clause. This sorts the results set by unintended columns, potentially resulting in incorrect results and silent bug. Similar concept for GROUP BY. [Rules 10.3/4]

## 4. Numeric rounding functions: CEILING(), FLOOR() and ROUND(), SIGN() 
When used in a WHERE clause, these functions can prevent the SQL Server engine from utilizing indexes effectively, often resulting in an Index Scan instead of a more efficient Index Seek. From a performance standpoint, these predicates should be rewritten using equivalent arithmetic conditions that preserve index usage and allow the optimizer to choose an Index Seek. In the following example in WorldWideImportersDW database, the table City has a PK index on [City Key] column. Refactoring the WHERE condition shows a query cost, I/O, CPU and elapsed time decreases by 99%, changing the plan from an Index Scan to an Index Seek. [Rules 10.5/6]
#### see how to manage CEILING() (docs/Ceiling.md)
#### see how to manage FLOOR()
#### see how to manage ROUND()
#### see how to manage SIGN()

## 5.  Arithmetic functions
In this example in AdventureWorks2022 database, the ABS() function is used in the WHERE condition, preventing the use of the existing index on ReferenceOrderID column. This function has been replaced by an equivalent algebraic condition, so that the existing index can be used. In this way the execution plan changes from running an Index SCAN to a more efficient Index SEEK. [Rule 10.7]
#### ABS()
#### SQRT()
#### POWER()

## 6.  Arithmetic EXPRESSION 
Simple arithmetic expressions can be written differently to force the execution plan to change from using a table or index Scan to Index Seek. The following example is on AdventurWorks2022. Since the SalesOrderID is multiplied, the condition SalesOrderID * 3 < 1200 may prevent SQL Server from effectively using the existing PK index. That’s because SQL Server evaluates the multiplication result and not the column data, ‘hiding’ the indexed data. It compares each single multiplication result to 1200, leading to a Scan on the column. This case shows a variation of Cost: -99%, CPU and elapsed time: -99%, I/O: -99%. [Rules 10.8/9]
#### ><
#### equations like

## 4.5  Time management functions 
Some functions can be rewritten differently, avoiding applying the function to the column. In this scenario suppose that I have an index IX1 ON [Sales].[SalesOrderDetail] ([ModifiedDate]), even without Include columns. To take advantage of it we must rewrite the WHERE condition as shown below. The cost variation is -99%, elapsed time -58%. [Rules 10.11/14]
#### DATEDIFF Questa è una guida. [Vai alla guida dettagliata](docs/guida.md)
#### DATEADD
#### DATEPART
#### YEAR

## 8 Other functions
#### LIKE
#### ISNULL

## 5. Prevent implicit conversions
Implicit conversions occur when the engine automatically converts one data type to another without the user explicitly specifying it. This typically occurs when SQL Server compares two items having different data types, and needs to perform a type conversion before the comparison. When an implicit conversion occurs on an indexed column, SQL Server may not be able to use the index efficiently and may also produce poor cardinality estimates. For example, if an index is built on a column of type INT, but a query compares it to a SQL_VARIANT value, this triggers an implicit column conversion and SQL Server cannot use the index with Seek and may execute a full scan. Whether the conversion applies to the column or the value depends on data type precedence order, but our goal is to avoid relying on these rules and ensuring data type consistency in all scenarios. The best way to avoid these conversion-related issues is to ensure that data types involved in comparisons match. So, we can face two cases:

If we compare two table columns, we cannot change their data type, so we can use CONVERT function to explicitly force the data type to be the same in the comparison.
If we compare a variable or parameter, we can instruct the AI model to force a different variable or parameter declaration to match the column data type.


## 7. Remove Unused and Irrelevant code
Unused code refers to portions of code that are written but never executed during the lifecycle of an application. This can include declared variables that are never utilized, temporary tables that are created but never populated, or entire logic blocks that remain unreachable. Irrelevant code (unuseful), on the other hand, may be executed but has no impact in the current context. It may have served a purpose in an earlier version of the application or been introduced as a placeholder during development without being finalized or removed. In the example below, the original function contains unused parameters, superfluous local variables, and irrelevant logic. With the right prompts and guidance, an OpenAI model can detect and eliminate these elements, resulting in cleaner, more efficient and maintainable code. [Rules 10.15/16]

## 8. Generating a more secure code
SQL Injection is a security vulnerability that allows an attacker to modify the SQL queries an application makes to its database. By injecting malicious SQL code into input fields, an attacker can alter, retrieve, or even delete data, potentially compromising entire databases. It typically occurs when user input is not properly validated before being embedded in SQL statements. OpenAI models can assist in identifying risky coding patterns that lead to SQL injection vulnerabilities. These patterns often arise from insufficient input validation, lack of strict type enforcement, or the unsafe use of dynamic SQL execution methods such as EXEC with concatenated strings. In this stored procedure below, the @cityname parameter is directly concatenated into a SQL string and executed, making it vulnerable to injection.




## 📦 Prerequisites

- Windows OS
- [.NET Framework 4.8+](https://dotnet.microsoft.com/en-us/download/dotnet-framework)
- Azure OpenAI Resource (with `gpt-4.1`, `gpt-4o`, or `o3-mini` deployments)
- SQL Server with accessible schema

---

## 📄 License
This project is licensed under the MIT License.
