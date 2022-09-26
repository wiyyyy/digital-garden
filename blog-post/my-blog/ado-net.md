---
tags: ado.net, orm, info-capsule
---
ADO.NET is developed by Microsoft in order to interact with different database engines. 

Basic flow for creating and executing queries with every engine is defined below
- Create connection
- Open connection
- Create query
- Execute query
- Parse results

## Creating connection
You can create ado.net connections like below. It is important to use ``using`` block. After ``using`` blocks connection is automatically closed and returned to the pool by Ado.Net. ``SqlConnection`` class is implementing ``IDisposable`` interface in order to provide that feature.
Another note is that you have to call ``conn.Open()`` method. If not connection is created in ``Closed`` state
````csharp
using (var conn = new SqlConnection(connectionString))
{
    conn.Open();   
}
````

## Create query
You can create SQL command with the code below. Again it is important to use ``using`` block in order to prevent memory leaks in application. If your application is data heavy and executing many queries with short time this is important.
````csharp
using (var cmd = conn.CreateCommand())
{
    cmd.CommandText = "SELECT * FROM Table";
}
````
> ## ⚠ How to prevent SQL injection attacks
> SQL injection is critical security vulnerability. Hackers can get database dump with this vulnerability. If your application is containing sensitive data this can cause a lot of trouble. 
> 
> General recommendation in every language to prevent SQL injection attacks is to parameterize SQL query so that malicious SQL command can not be injected to query.
> 
> In Ado.Net you can do this with ``Parameters`` property of ``SqlCommand`` class
> ````csharp
> using (var cmd = conn.CreateCommand())
> {
>    cmd.CommandText = "SELECT * FROM Table WHERE Id=@Id";
>    cmd.Parameters.AddWithValue("@Id",id);
> }
> ````

## Execute Query 
After creating SQL command we need to execute it in order to get the result from database. There are many ways to execute queries in Ado.Net and it depends on SQL query itself. 

Some execute commands
- ExecuteNonQuery
  Used mainly for executing update, insert and delete queries. Return number of row effected.
  ````csharp
  ...
  cmd.ExecuteNonQuery();
  ...
  ````
- ExecuteScalar
  Returns first row and first column of result. 
  ````csharp
  ...
  cmd.ExecuteScalar();
  ...
  ````
- ExecuteReader
  For parsing result. Returns sequence of result. Iteration needed.
  ````csharp
  using (var reader = cmd.ExecuteReader())
  {
      while (reader.Read())
      {
          // Parse result
      }
  }
  ````
## Parsing Results
Parsing the result is required in Ado.Net. You can parse data for your class with the Reader manually. Which column belongs to which property of class. 
````csharp
public MyClass 
{
    public string Property1 { get; set; }
    public int Property2 { get; set; }
}
--
MyClass myClass = new MyClass();
using (var reader = cmd.ExecuteReader())
{
    myClass.Property1 = reader["ColumnName"].ToString();
    myClass.Property2 = (int)reader["AnotherColumnName"];
}
  ````

## Full Code
````csharp
MyClass myclass = new MyClass();
using (var conn = new SqlConnection(connectionString))
{
    conn.Open();
    using (var cmd = conn.CreateCommand())
    {
        cmd.CommandText = "Select * from Table where Id=@Id";
        cmd.Parameters.AddWithValue("@Id",id);
        using (var reader = cmd.ExecuteReader())
        {
            while(reader.Read())
            {
                myclass.Property1 = reader["ColumnName"].ToString();
                myclass.Property2 = (int)reader["AnotherColumnName"];
            }
        }
    }
}
````