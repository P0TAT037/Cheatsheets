# Dapper

Dapper is a simple object mapper (micro-orm) for .NET that allows you to execute raw SQL queries, map the results to objects, and execute stored procedures, among other things. Dapper was developed by the Stack Overflow team and it supports variety of DB providers such as SQL Server, PostgreSQL, MySQL, Sqlite etc.

## How to use

Dapper is so simple, to use dapper you just need to have the database provider for your database. For example if you are using SqlServer you need to install the `System.Data.SqlClient` package.

you connect to the database by creating a new `SqlConnection`
then you define your sql query and pass it to the one of the Dapper query extension methods of the connection object

```csharp
using var connection = new SqlConnection(connectionString);

var sql = "SELECT * FROM Users";

// this will return a `IEnumerable<User>` of User data mapped to User objects
var users = connection.Query<User>(sql);
```

## Dapper's Most Common Query Methods

### Querying Scalar Values

- `ExecuteScalar` - Executes a query and returns the first column of the first row in the result set returned by the query as a `dynamic` type. Additional columns or rows are ignored.

- `ExecuteScalar<T>` - Executes a query and returns the first column of the first row in the result set returned by the query. Additional columns or rows are ignored.

- `ExecuteScalarAsync` - Asynchronously executes a query and returns the first column of the first row in the result set returned by the query as a `dynamic` type. Additional columns or rows are ignored.

- `ExecuteScalarAsync<T>` - Asynchronously executes a query and returns the first column of the first row in the result set returned by the query. Additional columns or rows are ignored.

### Querying Single Row

- `QuerySingle` - Returns the first row of a query result as a `dynamic` type, throwing an `InvalidOperationException` exception if no rows or more than one are found.

- `QuerySingle<T>` - Returns the first row of a query result, throwing an `InvalidOperationException` exception if no rows or more than one are found.

- `QuerySingleOrDefault` - Returns the first row of a query result as a `dynamic` type, throwing an `InvalidOperationException` exception if more than one row is found or null if no elements are found.

- `QuerySingleOrDefault<T>` - Returns the first row of a query result, throwing an `InvalidOperationException` exception if more than one row is found or null if no elements are found.

- `QueryFirst` - Returns the first row of a query result as a `dynamic` type, throwing an `InvalidOperationException` exception if no rows are found.

- `QueryFirst<T>` - Returns the first row of a query result, throwing an `InvalidOperationException` exception if no rows are found.

- `QueryFirstOrDefault` - Returns the first row of a query result as a `dynamic` type or null if no elements are found.

- `QueryFirstOrDefault<T>` - Returns the first row of a query result or null if no elements are found.

### Querying Multiple Rows

- `Query` - Returns an `IEnumerable` of `dynamic` types.

- `Query<T>` - Returns an `IEnumerable` of the specified type.

- `QueryAsync` - Asynchronously returns an `IEnumerable` of `dynamic` types.

- `QueryAsync<T>` - Asynchronously returns an `IEnumerable` of the specified type.

### Non-Query Commands

These are the commands that are not expected to return data like insert, update, and delete. for those you can use one of these methods:

- `Execute` - Executes a query, returning an `int` that represents the number of rows affected.

- `ExecuteAsync` - Asynchronously executes a query, returning an `int` that represents the number of rows affected.

## Stored Procedures

To execute a stored procedure you can use the `Query` or `Execute` and pass a sql statement that calls the stored procedure.

```csharp
var sql = "EXEC GetSalesByYear @BeginningDate, @EndingDate";
var values = new { BeginningDate = "2017-01-01", EndingDate = "2017-12-31" };
var results = connection.Query(sql, values).ToList();
results.ForEach(r => Console.WriteLine($"{r.OrderID} {r.Subtotal}"));
```

Or you can change the `CommandType` parameter to `CommandType.StoredProcedure` and pass the name of the stored procedure as the first parameter.

```csharp
var storedProcedureName = "GetSalesByYear";
var values = new { BeginningDate = "2017-01-01", EndingDate = "2017-12-31" };
var results = connection.Query(storedProcedureName, values, commandType: CommandType.StoredProcedure).ToList();
results.ForEach(r => Console.WriteLine($"{r.OrderID} {r.Subtotal}"));
```

## Using Parameters

It's important to use parameters in your queries and to avoid using string interpolation to prevent SQL injection attacks. Dapper supports some ways in which you can pass parameters safely to a query. Following are the most common methods.

### Anonymous Parameters

```csharp
var parameters = new { UserName = username, Password = password };
var sql = "SELECT * from users where username = @UserName and password = @Password";
var result = connection.Query(sql, parameters);
```

### Dynamic Parameters

```csharp
var dictionary = new Dictionary<string, object>
{
    { "@ProductId", 1 }
};
var parameters = new DynamicParameters(dictionary);
var sql = "SELECT * FROM products WHERE ProductId = @ProductId";
using (var connection = new SqlConnection(connectionString))
{
    var product = connection.QuerySingle<Product>(sql, parameters);
}
```

Or you can pass it as a `DynamicParameters` object directly to the `Query` method.

```csharp
var parameters = new DynamicParameters({ ProductId = 1 });
var sql = "SELECT * FROM products WHERE ProductId = @ProductId";
using (var connection = new SqlConnection(connectionString))
{
    var product = connection.QuerySingle<Product>(sql, parameters);
}
```

### String Parameters

```csharp
string sql = "SELECT * FROM Customers WHERE Email = @Email";
using (var connection = new SqlConnection(connectionString))
{
connection.Open();
var dbParams = new DbString()
{
    Value = Email, 
    IsAnsi=true, 
    IsFixedLength=true 
};
var customer = connection.Query<Customer>(sql, new { Email = dbParams })
    .FirstOrDefault();
}
```

_ _ _

### *That's it, now you know.*
