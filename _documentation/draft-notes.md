
-----



## ServiceStack.Redis

- `GetClientAsync()` and `GetReadOnlyClientAsync()` in `PooledRedisClientManager` was rewritten to use async locks by Pete Ness

## ServiceStack.OrmLite

Added new CIL SQLite provider:

> dotnet add package ServiceStack.OrmLite.Sqlite.Cil

- Added `EnableForeignKeysCheck()` and `DisableForeignKeysCheck()` APIs (inc async APIs)
- Added `InTransaction()` and `GetTransaction()` IDbConnection APIs

Added precision support for MySQL DATETIME:

 ```csharp
MySqlDialect.Provider.GetConverter(typeof(DateTime)).Precision = 3;
 ```

[@pedoc](https://github.com/pedoc) Added C# DateTime `ToString()` Format Support for MySQL Server `DATE_FORMAT(column,dateFormat)`, e.g:

```csharp
var yesterday = DateTime.Now.AddDays(-1).ToString("yyyy-MM-dd");
var today = DateTime.Now.ToString("yyyy-MM-dd");
db.Save(new Table { Id = 1, Date = DateTime.Now });
var q = db.From<Table>().Where(i => i.Date.ToString("%Y-%m-%d") == yesterday));
var results = db.Select<string>(q.Select(i => i.Date.ToString("%Y-%m-%d"));
```


### Cross platform dotnet scripts

https://docs.servicestack.net/dotnet-scripts

### gist.cafe