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
#### [CEILING()](docs/Ceiling.md)
#### [FLOOR()]
#### [ROUND()]
#### [SIGN()]

## 5.  Arithmetic functions
In this example in AdventureWorks2022 database, the ABS() function is used in the WHERE condition, preventing the use of the existing index on ReferenceOrderID column. This function has been replaced by an equivalent algebraic condition, so that the existing index can be used. In this way the execution plan changes from running an Index SCAN to a more efficient Index SEEK. [Rule 10.7]
#### [ABS()]
#### [SQRT()]
#### [POWER()]

## 6.  Arithmetic EXPRESSION 
Simple arithmetic expressions can be written differently to force the execution plan to change from using a table or index Scan to Index Seek. The following example is on AdventurWorks2022. Since the SalesOrderID is multiplied, the condition SalesOrderID * 3 < 1200 may prevent SQL Server from effectively using the existing PK index. That’s because SQL Server evaluates the multiplication result and not the column data, ‘hiding’ the indexed data. It compares each single multiplication result to 1200, leading to a Scan on the column. This case shows a variation of Cost: -99%, CPU and elapsed time: -99%, I/O: -99%. [Rules 10.8/9]
#### ><
#### equations like

## 4.5  Time management functions 
Some functions can be rewritten differently, avoiding applying the function to the column. In this scenario suppose that I have an index IX1 ON [Sales].[SalesOrderDetail] ([ModifiedDate]), even without Include columns. To take advantage of it we must rewrite the WHERE condition as shown below. The cost variation is -99%, elapsed time -58%. [Rules 10.11/14]
#### DATEDIFF Questa è una guida. [Vai alla guida dettagliata](docs/Datediff.md)
#### DATEADD
#### DATEPART
#### YEAR

## 8 Other functions
#### LIKE
#### ISNULL

## 5. Prevent implicit conversions
Implicit conversions occur when the engine automatically converts one data type to another without the user explicitly specifying it. This typically occurs when SQL Server compares two items having different data types, and needs to perform a type conversion before the comparison. When an implicit conversion occurs on an indexed column, SQL Server may not be able to use the index efficiently and may also produce poor cardinality estimates. For example, if an index is built on a column of type INT, but a query compares it to a SQL_VARIANT value, this triggers an implicit column conversion and SQL Server cannot use the index with Seek and may execute a full scan. Whether the conversion applies to the column or the value depends on data type precedence order, but our goal is to avoid relying on these rules and ensuring data type consistency in all scenarios. The best way to avoid these conversion-related issues is to ensure that data types involved in comparisons match. So, we can face two cases:

- If we compare two table columns, we cannot change their data type, so we can use CONVERT function to explicitly force the data type to be the same in the comparison.
- If we compare a variable or parameter, we can instruct the AI model to force a different variable or parameter declaration to match the column data type.

In the case below in AdventureWorks2022, suppose to have an index on ModifiedDate column. Column ModifiedDate and variable @Salesday have different data types (datetime and sql_variant respectively). The comparison within the WHERE condition ( ModifiedDate >= @Salesday) leads to an implicit conversion of the column which prevents the use of index.
CREATE NONCLUSTERED INDEX AI_index ON [Sales].[SalesOrderDetail] ([ModifiedDate]) INCLUDE ([SalesOrderID])

```sql
DECLARE @Salesday sql_variant 
SET @Salesday = '20130731'

SELECT UnitPrice, LineTotal
FROM Sales.SalesOrderDetail
WHERE ModifiedDate >= @Salesday
```
<div style="text-align: left;">
  <img 
    src="https://github.com/user-attachments/assets/bf65f224-3d43-4ea7-b5ff-ae72850a0125" 
    alt="Convert_Implicit"
    style="width: 60%;" />
</div>

Given the code above, we can implement two different solutions: the first is to declare the @Salesday variable with the same data type as ModifiedDate. The second is to explicitly convert the @Salesday variable to the same data type as ModifiedDate. Both solutions isolate the column preventing functions or implicit conversions applied on it.

<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Solution 1</h4>
      <pre><code>
DECLARE @Salesday datetime 
SET @Salesday = '20130731'

SELECT SalesOrderID, ModifiedDate
FROM Sales.SalesOrderDetail
WHERE ModifiedDate >= @Salesday
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Solution 2</h4>
      <pre><code>
DECLARE @Salesday sql_variant 
SET @Salesday = '20130731'

SELECT SalesOrderID, ModifiedDate 
FROM Sales.SalesOrderDetail
WHERE ModifiedDate >= CONVERT(datetime,@Salesday)
      </code></pre>
    </td>
  </tr>
</table>

<div style="text-align: left;">
  <img 
    src="https://github.com/user-attachments/assets/14b75c76-6ac8-4def-b4d4-7f393a87e4b7"
    alt="Convert_Implicit"
    style="width: 60%;" />
</div>

This scenario assumes that the AI model has complete knowledge of the data type of every column in each table. This allows the model to identify the correct data type for declaring the @Salesday variable or to apply the correct conversions to constant values. How can this information be ingested into the AI model via prompt? One possible solution is to use JSON format. JSON is an efficient, human-readable format that organizes data in a clear structure, making it easy for AI models to parse and interpret. Its flexibility, compactness, and widespread compatibility make it ideal for providing complex data in prompts. The following SQL query retrieves the columns data types in JSON format of all tables, preparing the information for inclusion in the AI prompt:
```sql
--retrieves the columns data types in JSON format
SELECT 
    SCHEMA_NAME(t.schema_id) AS [Schema],
    t.name AS [Table],
    c.name AS [Column],
    CASE 
        WHEN ty.name IN ('char', 'varchar', 'nchar', 'nvarchar', 'binary', 'varbinary') THEN 
            ty.name + 
            CASE 
                WHEN c.max_length = -1 THEN '(MAX)'
                ELSE '(' + CAST(c.max_length / 
                    CASE 
                        WHEN ty.name IN ('nchar', 'nvarchar') THEN 2 
                        ELSE 1 
                    END AS VARCHAR) + ')'
            END
        WHEN ty.name IN ('decimal', 'numeric') THEN 
            ty.name + '(' + CAST(c.precision AS VARCHAR) + ',' + CAST(c.scale AS VARCHAR) + ')'
        ELSE ty.name
    END AS DataType
FROM 
    sys.tables t
    INNER JOIN sys.columns c ON t.object_id = c.object_id
    INNER JOIN sys.types ty ON c.user_type_id = ty.user_type_id
ORDER BY 
    t.name, c.column_id
FOR JSON PATH, ROOT('TablesColumns');
```
For example, the following JSON provides a structured representation of the test table schema:
<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Solution 1</h4>
      <pre><code>
CREATE TABLE [dbo].[test](
 [CultureID] [nchar](6) NOT NULL,
 [Name] [nvarchar](128) NOT NULL,
 [ModifiedDate] [datetime] NOT NULL
)
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <pre><code>
{
  "TablesColumns": [
    {
      "Schema": "dbo",
      "Table": "test",
      "Column": "CultureID",
      "DataType": "nchar(6)"
    },
    {
      "Schema": "dbo",
      "Table": "test",
      "Column": "Name",
      "DataType": "nvarchar(128)"
    },
    {
      "Schema": "dbo",
      "Table": "test",
      "Column": "ModifiedDate",
      "DataType": "datetime"
    }
  ]
}
      </code></pre>
    </td>
  </tr>
</table>

## 6. Replace deprecated Large Object Data Types
TEXT, NTEXT, and IMAGE data types are deprecated in SQL Server because they are legacy types with limited functionality and compatibility in modern T-SQL. They do not support common string or binary functions, cannot be used easily in expressions and are inefficient. To improve performance, maintainability, and compatibility with current and future SQL versions, Microsoft recommends replacing them with their modern counterparts: TEXT should be replaced with VARCHAR(MAX), NTEXT with NVARCHAR(MAX), and IMAGE with VARBINARY(MAX). These newer types support full string and binary operations, work more efficiently with indexes and memory, and are fully supported in all SQL Server features. [Rule 10.17]

## 7. Remove Unused and Irrelevant code
Unused code refers to portions of code that are written but never executed during the lifecycle of an application. This can include declared variables that are never utilized, temporary tables that are created but never populated, or entire logic blocks that remain unreachable. Irrelevant code (unuseful), on the other hand, may be executed but has no impact in the current context. It may have served a purpose in an earlier version of the application or been introduced as a placeholder during development without being finalized or removed. In the example below, the original function contains unused parameters, superfluous local variables, and irrelevant logic. With the right prompts and guidance, an OpenAI model can detect and eliminate these elements, resulting in cleaner, more efficient and maintainable code. [Rules 10.15/16]

<div style="text-align: left;">
  <img 
    src="https://github.com/user-attachments/assets/96219e8f-4d27-4cf4-9ae8-0ecef903dcbc"
    alt="Convert_Implicit"
    style="width: 70%;" />
</div>

## 8. Generating a more secure code
SQL Injection is a security vulnerability that allows an attacker to modify the SQL queries an application makes to its database. By injecting malicious SQL code into input fields, an attacker can alter, retrieve, or even delete data, potentially compromising entire databases. It typically occurs when user input is not properly validated before being embedded in SQL statements. OpenAI models can assist in identifying risky coding patterns that lead to SQL injection vulnerabilities. These patterns often arise from insufficient input validation, lack of strict type enforcement, or the unsafe use of dynamic SQL execution methods such as EXEC with concatenated strings. In this stored procedure below, the @cityname parameter is directly concatenated into a SQL string and executed, making it vulnerable to injection.

```sql
 CREATE PROCEDURE [dbo].[usp_testInj]
@cityname [varchar](256)
AS
BEGIN
DECLARE @query varchar(1024)
SET @query = 
'SELECT A.AddressID, A.AddressLine1, SP.Name
FROM Person.Address A INNER JOIN Person.StateProvince SP 
ON A.StateProvinceID = SP.StateProvinceID 
WHERE A.City = ''' + @cityname + ''
  
EXEC (@query)
END
```
The second line below shows how it is possible to drop a table TabX, just passing an executable string as a malicious parameter value.

EXEC dbo.usp_testInj 'Bothell'''  --OK

EXEC dbo.usp_testInj 'Bothell''; DROP TABLE dbo.TabX;'   --ATTACK!!

- **Solution 1**: A safe option is input validation. This involves checking that user inputs conform to expected formats before using them in SQL queries. By restricting input to valid characters or patterns, and excluding specific keywords, you can significantly reduce the risk of injection attacks, though this alone could not be sufficient.
- **Solution 2**: This solution uses the parameterized query executed by sp_executesql. The key protection comes from separating code (the SQL statement with parameter placeholders) from user input (the parameter value). This separation ensures the input is treated strictly as data, and not as executable code. [Rule 10.20]

<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Solution 1</h4>
      <pre><code>
--SOLUTION 1
CREATE PROCEDURE [dbo].[usp_testInj1]
@cityname [varchar](256)
AS
BEGIN
  DECLARE @query varchar(1024)
  -- Check for dangerous keywords in @cityname 
  IF CHARINDEX('DROP', @cityname) > 0 OR 
  CHARINDEX('DELETE', @cityname) > 0 OR
  CHARINDEX(';', @cityname) > 0 OR
  CHARINDEX('UPDATE', @cityname) > 0 OR
  CHARINDEX('TRUNCATE', @cityname) > 0 OR
  CHARINDEX('GRANT', @cityname) > 0
  BEGIN
    RAISERROR('Invalid input detected.', 16, 1)
  RETURN
END

SET @query =
'SELECT A.AddressID, A.AddressLine1, SP.Name
 FROM Person.Address A
 INNER JOIN Person.StateProvince SP
 ON A.StateProvinceID = SP.StateProvinceID
 WHERE A.City = ''' + @cityname + ''

EXEC (@query)
END
      </code></pre>
    </td>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Solution 2</h4>
      <pre><code>
--SOLUTION 2
CREATE PROCEDURE [dbo].[usp_testInj2]
@cityname [varchar](256)
AS
BEGIN
  DECLARE @query nvarchar(256)
  SET @query = 
  'SELECT A.AddressID, A.AddressLine1, SP.Name
   FROM Person.Address A 
   INNER JOIN Person.StateProvince SP 
   ON A.StateProvinceID = SP.StateProvinceID 
   WHERE A.City = @cityParam'

  EXEC sp_executesql @query,
  N'@CityParam VARCHAR(256)',
  @CityParam = @cityname
END
      </code></pre>
    </td>
  </tr>
</table>

## 9. Refactoring SQL with GPT-4o via Azure OpenAI and C#
To automate and improve SQL query refactoring using Azure OpenAI, for example, you can start deploying an AI model with Azure AI Foundry and integrating the Azure OpenAI .NET SDK into a C# application. The application interacts with a deployed GPT model (e.g., gpt-4o) through a structured sequence of chat messages. These messages include a system prompt that clearly defines the task, the SQL query to be optimized and the refactoring ‘rules’. The language model then analyzes the input, detects potential anti-patterns, and returns a refactored query by applying the rules provided. Prerequisites for this implementation include an active Azure OpenAI resource, a valid API key, a properly configured model deployment (e.g., gpt-4o), and the Azure.AI.OpenAI NuGet package.

Here is a simple example to start:

```csharp
// Import namespaces for using OpenAI chat functionality and Azure resources
using OpenAI.Chat;
using Azure;
using Azure.AI.OpenAI;

var endpoint = new Uri("https://resource.openai.azure.com/");
var apiKey = "d601113d574940538109ee*********";
var deploymentName = "gpt-4o";
ChatCompletionOptions requestOptions = null;

requestOptions = new ChatCompletionOptions()
     {
        MaxOutputTokenCount = 8192,
        Temperature = 0.4f,
        TopP = 1.0f
     };

var azureClient = new AzureOpenAIClient(endpoint, new AzureKeyCredential(apiKey));
var chatClient = azureClient.GetChatClient(deploymentName);

// Create chat messages
string RuleSet = <file content containing the prompt rules>
string SQLQueryToOptimize = <SQL query text to refactor>
string SchemaInfo = <JSON format of database schema information>
string Context = "You are a SQL Server developer who refactors code. Read and analyze all the provided T-SQL batch and rewrite it applying the following rules:"

List<ChatMessage> messages = new List<ChatMessage>()
     {
       new SystemChatMessage(Context),
       new UserChatMessage(SQLQueryToOptimize),
       new UserChatMessage(RuleSet),
       new UserChatMessage(SchemaInfo),
     };

// Send the chat messages to the model and get the response asynchronously
var response = await chatClient.CompleteChatAsync(messages, requestOptions);
string answer = response.Value.Content[0].Text;
answer = answer.Replace("\n", "\r\n");
```

## 10. Prompt ruleset
Below are some example prompt rules supplied to the model. These rules guide the AI in identifying specific cases described and refactoring the code according to the provided instructions. This content should be provided to the model through a prompt, encapsulated in the RuleSet variable as shown in the code example above.


## 📦 Prerequisites

- Windows OS
- [.NET Framework 4.8+](https://dotnet.microsoft.com/en-us/download/dotnet-framework)
- Azure OpenAI Resource (with `gpt-4.1`, `gpt-4o`, or `o3-mini` deployments)
- SQL Server with accessible schema

---

## 📄 License
This project is licensed under the MIT License.
