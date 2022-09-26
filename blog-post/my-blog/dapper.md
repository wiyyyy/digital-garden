---
tags: orm, dapper, ado.net, benchmark
---

All types of applications in business world is required to use database in order to store all the information related to business. Therefore every application is needed to query database to retrieve or place data on database. 

Every data inserted to or queried from database is consist of some model wrapping around it. In ADO.NET library these model mapping is done manually by developers. For complex models, complex queries and model that used multiple times in different queries need to be mapped properly. These can be an issue for consistency of code base especially developer joined recently on company. 

Dapper is created for limiting these issue and provide better mapping options developers. With using Dapper, developers can map their queries to their model without mapping all the properties in class manually.

Dapper is actually a wrapper around ADO.NET. So under the hood every query is executed by ADO.NET. With this advantage query execution is really fast. I will try to show you speed of this on the benchmark part of this post.

So lets talk about Dapper style with comparisons to ADO.NET style.


## Dapper vs ADO.NET Code Style

### ADO.NET Style

````csharp
public List<MyModel> GetData(string id)
{
    List<MyModel> models = new List<MyModel>();
    using(var conn = new SqlConnection(_connectionString))
    {
        conn.Open();
        using(var cmd = conn.CreateCommand())
        {
            cmd.CommandText = "Select * from table where Id=@Id"; 
            cmd.Parameters.AddWithValue("@Id", id);
            using(var reader = cmd.ExecuteReader())
            {
                while(reader.Read())
                {
                    MyModel model = new MyModel();
                    model.Prop1 = reader["column_1"].ToString();
                    model.Prop2 = reader["column_2"].ToString();
                    model.Prop3 = reader["column_3"].ToString();
                    models.Add(model);
                }
            }
        }
    }
    return models;
}
````

This is the code you would write when interacting with database via ADO.NET. Let's write this in Dapper. 

### Dapper Style

````csharp
public List<MyModel> GetData(string id)
{
    using(var conn = new SqlConnection(_connectionString))
    {
        return conn.Query<MyModel>("Select * from table where Id=@id",new { id }).ToList();
    }
}
````

The code blocks above are doing the same operation on both database and application. As you can see it is very easy and more readable in Dapper. In ADO.NET you have to map every property manually to meet the requirements of your custom class but with Dapper you don't have to map or track every property on your class. We can pass our parameters in anonymous class. This enables writing more readable and more maintainable code both teams and developers.

### Dapper Insert

````csharp
public void InsertData(MyModel myModel)
{
    using(var conn = new SqlConnection(_connectionString))
    {
        return conn.ExecuteNonQuery(@"Insert Into table (column_1,column_2,column_3) 
                                    Values(@column_1, @column_2, @column_3)", 
                    new { myModel.column_1, myModel.column_2, myModel.column_3 });
    }
}
````

### Dapper ExecuteScalar

Dapper provide generic methods for define return type of query. In example below, we are able to convert ``column_1`` data to ``int`` so that we don't have to convert manually this value with ADO.NET. With this option we can write ``IsExists`` methods more readable.

````csharp
public int IsExists(int id)
{
    using(var conn = new SqlConnection(_connectionString))
    {
        return conn.ExecuteScalar<int>(@"Select column_1 from table where id=@id", new { id }) > 0;
    }
}
````

## Performance

So we reviewed the coding style of both ADO.NET and Dapper. Let's continue on performance part. 

Benchmark below is a simple ``select *`` query. It is getting data from database on my development server. So there is a little network overhead. But still, result can give you opinion on it. 

| Method | count |      Mean |     Error |    StdDev |    Median |   Gen 0 | Gen 1 | Gen 2 | Allocated |
|------- |------ |----------:|----------:|----------:|----------:|--------:|------:|------:|----------:|
| AdoNet |     1 |  8.656 ms | 0.1730 ms | 0.4275 ms |  8.577 ms |       - |     - |     - |   3.95 KB |
| Dapper |     1 |  8.372 ms | 0.4410 ms | 1.2653 ms |  7.857 ms |       - |     - |     - |   4.79 KB |
| AdoNet |    10 |  8.697 ms | 0.1720 ms | 0.2047 ms |  8.700 ms |       - |     - |     - |   5.87 KB |
| Dapper |    10 |  8.589 ms | 0.3491 ms | 0.9732 ms |  8.245 ms |       - |     - |     - |   7.04 KB |
| AdoNet |   100 | 10.256 ms | 0.2044 ms | 0.3471 ms | 10.225 ms |       - |     - |     - |  24.57 KB |
| Dapper |   100 |  9.828 ms | 0.4215 ms | 1.1751 ms |  9.412 ms |       - |     - |     - |   28.5 KB |
| AdoNet |  1000 | 23.662 ms | 0.4700 ms | 1.0217 ms | 23.494 ms | 31.2500 |     - |     - | 207.41 KB |
| Dapper |  1000 | 23.677 ms | 1.1306 ms | 3.2258 ms | 22.480 ms |       - |     - |     - | 239.45 KB |

As you can see from benchmark result Dapper is as fast as ADO.NET and provide more clean approach on your data access operations. 

## Conclusion

We look into ADO.NET and Dapper library usage and their performance. There is many more ORM tools and libraries in .NET world. You can find the ORM cookbook site that covers all the ORM tools in .NET world and can investigate all of them. 

Huge thanks to Dapper library creators and contributors. They are very good at .NET and related technologies. In case you want to follow them you can use the links below.


## Reference

https://learn.microsoft.com/tr-tr/dotnet/framework/data/adonet/ado-net-code-examples#sqlclient

https://dapperlib.github.io/Dapper/

https://tortugaresearch.github.io/DotNet-ORM-Cookbook/

[@Nick_craver](https://twitter.com/Nick_Craver)

[@marcgravell](https://twitter.com/marcgravell)

