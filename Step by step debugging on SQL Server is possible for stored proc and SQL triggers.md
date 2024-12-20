By using SQL Server Management Studio 17.9.1 (the latest version having the debug option) or Visual Studio (demo: https://www.youtube.com/watch?v=wng_eetygXM )

I can see the variable content (except table variables)
I can see the callstack.

For table variables, I cannot directly see them. But, I can add variable to view the content.

e.g. in a SQL trigger, I can see the content of inserted:

xml format:

`DECLARE @v1 XML = (SELECT * FROM inserted FOR XML AUTO)`

json format:

`DECLARE @v2 XML = (SELECT * FROM inserted FOR JSON AUTO)`
