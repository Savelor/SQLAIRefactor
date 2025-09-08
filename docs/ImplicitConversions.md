## Avoiding Implicit conversions
In this example below, suppose to have an index on SalesOrderDetail.ModifiedDate column. Column ModifiedDate and variable @Salesday have different data types (datetime and sql_variant respectively). The comparison within the WHERE condition ( ModifiedDate >= @Salesday) triggers an implicit conversion of the column which prevents the use of index, leading to an index scan.  

```sql
--Create an index on the columns
CREATE NONCLUSTERED INDEX AI_index ON [Sales].[SalesOrderDetail] ([ModifiedDate]) INCLUDE ([SalesOrderID])

--Application code
DECLARE @Salesday sql_variant 
SET @Salesday = '20130731'

SELECT UnitPrice, LineTotal
FROM Sales.SalesOrderDetail
WHERE ModifiedDate >= @Salesday
```
<div style="text-align: left;">
  <img 
    src="https://github.com/user-attachments/assets/2c226c32-d5d1-46b6-857f-75e8ba0af952"
    alt="Convert_Implicit"
    style="width: 60%;" />
</div>
Whether the conversion applies to the column or the value depends on data type precedence order (https://learn.microsoft.com/en-us/sql/t-sql/data-types/data-type-precedence-transact-sql?view=sql-server-ver17), but our goal is to avoid relying on these rules and ensuring data type consistency in all scenarios. The best way to avoid these conversion-related issues is to ensure that data types involved in comparisons match. So, we can face two cases:

- If we compare two table columns, we cannot change their data type, so we can use CONVERT function to explicitly force the data type to be the same in the comparison.
- If we compare a variable or parameter, we can instruct the AI model to force a different variable or parameter declaration to match the column data type.
  
Given the code above, we can implement two different solutions: both isolate the column preventing functions or implicit conversions applied on it.

- **Solution1.** Declare the @Salesday variable with the same data type as ModifiedDate. 
- **Solution2.** Explicitly convert the @Salesday variable to the same data type as ModifiedDate. 

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

The table below shows the great improvement of the query execution with the plan without Implicit Conversion:
<small>

| Metric            | Original Query | Solution 1 | Variation (%) | Solution 2 | Variation (%) |
|:------------------|---------------:|-----------:|--------------:|-----------:|--------------:|
| Cost              | 0.99           | 0.12       | **-87.9%  ✅**     | 0.12       | **-87.9%  ✅**     |
| CPU Time [ms]     | 108            | 15         | **-86.11%  ✅**   | 16         | **-85.19%  ✅**   |
| Elapsed Time [ms] | 651            | 391        | **-39.94%  ✅**   | 411        | **-36.71%  ✅**   |
| Logical Reads     | 1313           | 224        | **-82.94%  ✅**   | 224        | **-82.94%  ✅**   |

</small>


## How to provide columns data types to the AI model 
To implement the refactoring, this scenario assumes that the AI model has complete knowledge of the data type of every column in each table. This allows the model to identify the correct data type for declaring the @Salesday variable or to apply the correct conversions to constant values. How can this information be ingested into the AI model via prompt? One possible solution is to use JSON format. JSON is an efficient, human-readable format that organizes data in a clear structure, making it easy for AI models to parse and interpret. Its flexibility, compactness, and widespread compatibility make it ideal for providing complex data in prompts. The following SQL query retrieves the columns data types in JSON format of all tables, preparing the information for inclusion in the AI prompt:
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
