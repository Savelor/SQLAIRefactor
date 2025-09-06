## Avoiding SQL injection
SQL Injection (SQLi) is a type of security vulnerability that happens when an application allows untrusted input (like form fields, URL parameters, or API inputs) to be directly included in SQL queries without proper handling. Typically, SQL injection cases are characterized by the following scenarios:
- User input (e.g., @cityname) is directly concatenated into the query string.
- Attackers can inject arbitrary SQL code (for example: '; DROP TABLE dbo.TabX;--).
- Root cause: the application mixes code and data within a single SQL string.

The following example illustrates code that may be vulnerable to a SQL injection attack:
 
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

- **Solution 1**: A safe option is to introduce input validation. This involves checking that user inputs conform to expected formats before using them in SQL queries. By restricting input to valid characters or patterns, and excluding specific keywords, you can significantly reduce the risk of injection attacks, though this alone could not be sufficient.
- **Solution 2**: This solution uses the parameterized query executed by sp_executesql. The key protection comes from separating code (the SQL statement with parameter placeholders) from user input (the parameter value). This separation ensures the input is treated strictly as data, and not as executable code.
- **Solution 3**: Use Quotename function. QUOTENAME safely wraps input in single quotes and escapes any embedded ones. This approach is not as safe as parameterization, and can be used only if parameterization isn’t possible.
- **Solution 4**: When the query structure is fixed a static SQL statement can be a good option, since there's no string concatenation.

<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Solution 1</h4>
      <pre><code>
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
ciao ciao 

<table>
  <tr>
    <td style="vertical-align: top; padding: 10px;">
      <h4>🔹 Solution 3</h4>
      <pre><code>
CREATE PROCEDURE [dbo].[usp_testInj_Quotename]
@cityname VARCHAR(256)
AS
BEGIN
    DECLARE @query NVARCHAR(MAX)

    SET @query = 
    'SELECT A.AddressID, A.AddressLine1, SP.Name 
     FROM Person.Address A
     INNER JOIN Person.StateProvince SP 
       ON A.StateProvinceID = SP.StateProvinceID
     WHERE A.City = ' + QUOTENAME(@cityname, '''');

    EXEC (@query);
END
      </code></pre>
    </td>
  </tr>
</table>

