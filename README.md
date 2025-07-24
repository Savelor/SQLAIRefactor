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
**2. OLD JOIN syntax**
   
**3. ORDER BY / GROUP BY**
When using ORDER BY or GROUP BY clauses, it is recommended to explicitly use column names instead of column position numbers. In ORDER BY, relying on numeric positions can lead to errors if the SELECT clause is later modified, changing the order of selected columns without updating the ORDER BY clause. This sorts the results set by unintended columns, potentially resulting in incorrect results and silent bug. Similar concept for GROUP BY. [Rules 10.3/4]

**4. Numeric rounding functions: CEILING(), FLOOR() and ROUND(), SIGN()**
When used in a WHERE clause, these functions can prevent the SQL Server engine from utilizing indexes effectively, often resulting in an Index Scan instead of a more efficient Index Seek. From a performance standpoint, these predicates should be rewritten using equivalent arithmetic conditions that preserve index usage and allow the optimizer to choose an Index Seek. In the following example in WorldWideImportersDW database, the table City has a PK index on [City Key] column. Refactoring the WHERE condition shows a query cost, I/O, CPU and elapsed time decreases by 99%, changing the plan from an Index Scan to an Index Seek. [Rules 10.5/6]
## CEILING()
## FLOOR()
## ROUND()
## SIGN()



## 📦 Prerequisites

- Windows OS
- [.NET Framework 4.8+](https://dotnet.microsoft.com/en-us/download/dotnet-framework)
- Azure OpenAI Resource (with `gpt-4.1`, `gpt-4o`, or `o3-mini` deployments)
- SQL Server with accessible schema

---

## 📄 License
This project is licensed under the MIT License.
