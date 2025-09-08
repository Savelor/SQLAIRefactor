# SQLAIRefactor
SQLAIRefactor is a solution based on OpenAI which analyzes all SQL Server application code and identifies inefficiencies and anti-patterns, reporting evidence to the developer and proposing solutions and alternatives aligned with current best practices. This repository is the accompanying code for the **"AI-based T-SQL Refactoring: an automatic intelligent code optimization with Azure OpenAI"** article. Make sure that check that out at https://devblogs.microsoft.com/azure-sql/?p=4778&preview=true.

# Refactoring use cases
The following section showcases a curated collection of real-world SQL optimization use cases where AI can make a meaningful impact. Each scenario highlights common challenges in T-SQL development—from anti-patterns and performance bottlenecks to security flaws and inefficient code structures. In every case, the AI model can be guided using structured prompts to identify and refactor problematic code, improving performances, the execution plan, clarity and best practice alignment. This catalog can serve as a practical reference for developers to identify significant use cases where code can be refactored with great benefits.

## ⚙️1. Consistent Syntax
-  [SELECT *](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/ConsistentSyntax.md#1-select-)  
-  [Old join style](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/ConsistentSyntax.md#2-old-join-syntax)  
-  [ORDER BY](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/ConsistentSyntax.md#3-order-by--group-by)  

## ⚙️2. Numeric rounding functions: CEILING(), FLOOR() and ROUND(), SIGN() 
When used in a WHERE clause, these functions applied to an indexed column can prevent the SQL Server engine from utilizing indexes effectively, often resulting in an Index Scan instead of a more efficient Index Seek. From a performance standpoint, these predicates should be rewritten using equivalent arithmetic conditions that preserve index usage and allow the optimizer to choose an Index Seek. The following examples are in **WorldWideImportersDW** database. Refactoring the WHERE condition shows a significant query cost decrease and performance improvement. 
-  [CEILING()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/NumericRoundFunctions.md#ceiling)  
-  [FLOOR()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/NumericRoundFunctions.md#floor)  
-  [ROUND()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/NumericRoundFunctions.md#round)  
-  [SIGN()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/NumericRoundFunctions.md#sign)  

## ⚙️ 3.  Arithmetic functions
In this list in **AdventureWorks2022** database, these arithmetic functions used in the WHERE predicate have been replaced by equivalent algebraic condition, so that the existing index can be used. In this way the execution plan changes from running an Index SCAN to a more efficient Index SEEK. Tests show good improvements in cost and I/O.
-  [ABS()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/ArithmeticFunctions.md#abs)  
-  [SQRT()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/ArithmeticFunctions.md#sqrt)  
-  [POWER()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/ArithmeticFunctions.md#power)  

## ⚙️4. Time management functions 
The following time functions can be replaced in WHERE predicate, avoiding applying the function to the column. In this scenario in **AdventureWorks2022** database suppose that there is an index IX1 ON [Sales].[SalesOrderDetail] ([ModifiedDate]), even without Include columns. The following tests show exceptional improvement in all execution metrics of the AI generated code: 
-  [DATEDIFF()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/TimeFunctions.md#datediff) 
-  [DATEADD()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/TimeFunctions.md#dateadd)
-  [DATEPART()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/TimeFunctions.md#datepart)
-  [YEAR()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/TimeFunctions.md#year)

## ⚙️5. Arithmetic Operators 
Simple arithmetic expressions can be written differently to force the execution plan to change from using a table or index Scan to Index Seek. The following examples run on **AdventurWorks2022** database. 
-  [*, /, +, -](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/ArithmeticOperators.md#arithmetic-operators)
-  [Simple equations](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/ArithmeticOperators.md#simple-equations)


## ⚙️6. Other functions
- [LIKE](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/OtherFunctions.md#like)
- [ISNULL()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/OtherFunctions.md#isnull)
- [COALESCE()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/OtherFunctions.md#coalesce)
- [CONVERT()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/OtherFunctions.md#convert)
- [CAST()](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/OtherFunctions.md#cast)
- [OUTER APPLY](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/OtherFunctions.md#outer-apply)

## 🧩7. Prevent implicit conversions
Implicit conversions occur when the engine automatically converts one data type to another without the user explicitly specifying it. This typically occurs when SQL Server compares two items having different data types, and needs to perform a type conversion before the comparison. When an implicit conversion occurs on an indexed column, SQL Server may not be able to use the index efficiently and may also produce poor cardinality estimates. 

Analyze the details about how to avoid implicit conversione in you code here: [**Avoiding implicit conversions**](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/ImplicitConversions.md#avoiding-implicit-conversions)

## 🛡️8. Generating a more secure code (avoiding SQL injection)
SQL Injection is a security vulnerability that allows an attacker to modify the SQL queries an application makes to its database. It has been one of the OWASP Top 10 vulnerabilities for over a decade. By injecting malicious SQL code into input fields, an attacker can alter, retrieve, or even delete data, potentially compromising entire databases. It typically occurs when user input is not properly validated before being embedded in SQL statements. OpenAI models can assist in identifying risky coding patterns that lead to SQL injection vulnerabilities. These patterns often arise from insufficient input validation, lack of strict type enforcement, or the unsafe use of dynamic SQL execution methods such as EXEC with concatenated strings. In this stored procedure below, the @cityname parameter is directly concatenated into a SQL string and executed, making it vulnerable to injection.
Look at the details of the problem and examine possible solutions which can be implemented automatically in [**Avoiding SQL injection**](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/SQLinjection.md#avoiding-sql-injection)

## 🐌9. Refactoring cursors
The inefficiency of cursors is a classic pain point in SQL Server performance. Cursors allow you to iterate through rows one by one, similar to a loop in procedural languages. While this might feel intuitive for developers coming from imperative programming backgrounds, cursors are usually inefficient for several reasons: 

- Row-by-row processing: SQL Server is optimized for set-based operations. Cursors break this model.
- Resource intensive: They require memory and locks, and often spill to tempdb.
- Slow performance: For large result sets, performance degrades dramatically compared to set-based
  
See the details about how to refactor cursors here: [**Avoiding cursors**](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/Cursors.md#avoiding-cursors)

## ✂️10. Remove Unused and Irrelevant code
Unused code refers to portions of code that are written but never executed during the lifecycle of an application. This can include declared variables that are never utilized, temporary tables that are created but never populated, or entire logic blocks that remain unreachable. Irrelevant code (unuseful), on the other hand, may be executed but has no impact in the current context. It may have served a purpose in an earlier version of the application or been introduced as a placeholder during development without being finalized or removed. In the example below, the original function contains unused parameters, superfluous local variables, and irrelevant logic. With the right prompts and guidance, an OpenAI model can detect and eliminate these elements, resulting in cleaner, more efficient and maintainable code. 

<div style="text-align: left;">
  <img 
    src="https://github.com/user-attachments/assets/96219e8f-4d27-4cf4-9ae8-0ecef903dcbc"
    alt="Convert_Implicit"
    style="width: 70%;" />
</div>

## ♻️11. Replace deprecated Large Object Data Types
TEXT, NTEXT, and IMAGE data types are deprecated in SQL Server because they are legacy types with limited functionality and compatibility in modern T-SQL. They do not support common string or binary functions, cannot be used easily in expressions and are inefficient. To improve performance, maintainability, and compatibility with current and future SQL versions, Microsoft recommends replacing them with their modern counterparts: TEXT should be replaced with VARCHAR(MAX), NTEXT with NVARCHAR(MAX), and IMAGE with VARBINARY(MAX). These newer types support full string and binary operations, work more efficiently with indexes and memory, and are fully supported in all SQL Server features.

## 12. Refactoring SQL with GPT-4o via Azure OpenAI and C#
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

## 12. Prompt ruleset
When refactoring T-SQL code, an OpenAI model can be guided not only by its training but also by a custom set of refactoring rules. These rules describe which patterns are considered bad practices and how they should be transformed into more optimized, secure, or maintainable alternatives. The process works as follows:

- The model is given the original T-SQL code.
- It is also provided with the refactoring rules.
- Using both its training and the rules, the model identifies target cases (such as unsafe dynamic SQL, inefficient cursor usage, or string concatenation issues).
- The model then rewrites the code according to the specified rules, producing an improved version.

These rules effectively define the scope of refactoring use cases we want to address. See the details about how to refactor cursors here: [**Ruleset definition file**](https://github.com/Savelor/SQLAIRefactor/blob/master/docs/Cursors.md#avoiding-cursors)
<div style="text-align: center;">
  <img 
    src="https://github.com/user-attachments/assets/e1bafd47-1832-410b-b15b-0f38fde37049"
    alt="Convert_Implicit"
    style="width: 60%;" />
</div>

## 13. SQLAIRefactor as a Windows application
SQLAIRefactor is a Windows Forms application that leverages Azure OpenAI to analyze and optimize T-SQL queries. It connects to your SQL Server database, extracts schema metadata in JSON format, and uses prompt engineering and large language models to refactor queries and apply SQL Server best practices automatically.
This solution is an AI-powered application to automating SQL Server code analysis and refactoring. The system intelligently identifies inefficiencies and common T-SQL anti-patterns, applying best practices through a set of formalized coding rules, using prompt-driven instructions. It not only automatically rewrites problematic and inefficient code but also delivers contextual recommendations to improve quality, security, and maintainability.
<div align="center">
  <img src="https://github.com/user-attachments/assets/a29fac2d-d02e-4257-a2fe-d4cad9d7d4d7" alt="GUI1" width="650"/>
</div>

### How to use the tool

- **1. Paste Your Code**  
  Insert the original T-SQL code you want to optimize into the left panel of the interface.

- **2. (Optional) Connect to a Database**  
  For more accurate optimization, especially when metadata is required, connect to the relevant database. This allows the tool to retrieve metadata and pass it to OpenAI for better results.

- **3. Generate Optimization**  
  Click the button to submit your code. The tool sends the input from the left panel as a prompt to the AI.

- **4. Review Results**  
  The AI returns the optimized T-SQL code, along with the applied refactoring rules and additional insights or considerations.

<div align="center">
<img width="650" height="1801" alt="howtouse" src="https://github.com/user-attachments/assets/b27dd7a9-4466-4dcc-ba84-e566b16f285c" />
</div>

## 🚀 Features

- AI-powered SQL refactoring using GPT-4.1 or GPT-4o (Azure OpenAI)
- Retrieves and injects full table/column data types in JSON
- Identifies inefficiencies (e.g., implicit conversions, index scan vs. seek)
- Supports both Windows and SQL Authentication
- Renders results in an HTML-based view with syntax highlighting

---
## 📦 Theory

## 📦 Prerequisites

- Windows OS
- [.NET Framework 4.8+](https://dotnet.microsoft.com/en-us/download/dotnet-framework)
- Azure OpenAI Resource (with `gpt-4.1`, `gpt-4o`, or `o3-mini` deployments)
- SQL Server with accessible schema

---

## 📄 License
This project is licensed under the MIT License.
