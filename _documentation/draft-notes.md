
-----

### Desktop

Added heroicons to ServiceStack.Desktop

/lib/svg/heroicons/outline/academic-cap.svg
/lib/svg/heroicons/solid/academic-cap.svg

### GitHubGateway

- Add `GetRateLimits()` and `GetRateLimitsAsync()` APIs

## ProcessUtils

Used in gist.cafe



## AutoQuery

### Extend support of AutoPopulate and AutoMap attributes to Query APIs

Add support for `[AutoPopulate]` and `[AutoMap]` attributes to AutoQuery DTOs.
[AutoPopulate](https://docs.servicestack.net/autoquery-crud#autopopulate) lets you pre-populate a Request DTO Property with a constant value,
a static expression or a #Script Expression, in this case populates it with the `userAuthName` Script method which returns the Authenticated
User Name, which in this case populates the `CreatedBy` exact match filter to return all Bookings created by the Authenticated User:

```csharp
[ValidateIsAuthenticated]
[AutoPopulate(nameof(CreatedBy), Eval = "userAuthName")]
[AutoApply(Behavior.AuditQuery)]
public class QueryUserBookings : QueryDb<Booking>
{
    public string CreatedBy { get; set; }
}
```

Whilst [AutoMap](https://docs.servicestack.net/autoquery-crud#automap-and-autodefault-attributes) lets you map the value of a Request DTO
to a DataModel property with a different name which has the same behavior as above to populate the `CreatedBy` exact match filter:

```csharp
[ValidateIsAuthenticated]
[AutoPopulate(nameof(UserName), Eval = "userAuthName")]
[AutoApply(Behavior.AuditQuery)]
public class QueryUserBookings : QueryDb<Booking>
{
    [AutoMap(nameof(AuditBase.CreatedBy))]
    public string UserName { get; set; }
}
```

## Services

New `IServiceBeforeFilterAsync` and `IServiceAfterFilterAsync` interfaces can be used to execute custom async logic
before and after each API in your Service or base class, e.g:

```csharp
public class MyServices : Service, IServiceBeforeFilterAsync, IServiceAfterFilterAsync
{
    public Task OnBeforeExecuteAsync(object requestDto) => ...;

    public Task<object> OnAfterExecuteAsync(object response) => ...;
}
```


### Raven DB Auth Repo

[Zvjezdan Tomicevic](https://github.com/zeko77) from the ServiceStack community rewrote the `RavenDbUserAuthRepository` to use
only async APIs and upgraded to use the latest **RavenDB.Client** NuGet package dependency.


### Cross platform dotnet scripts

https://docs.servicestack.net/dotnet-scripts

## JWT

- `RequiresAudience` - Tokens must contain aud which is validated
- `ResolveUnixTime` - override resolving Unix Timestamps embedded in JWT Tokens
- `PreValidateJwtPayloadFilter` - Inspect or modify JWT Payload before validation

Make Custom JWT Auth Provider implementations easier

- `HasExpired`
- `HasBeenInvalidated`
- `HasInvalidAudience`
- `HasInvalidNotBefore`

Added `HasInvalidNotBefore` validation


### Java & Kotlin ServiceStack Reference

Our [Android](https://mvnrepository.com/artifact/net.servicestack/android) and [JVM](https://mvnrepository.com/artifact/net.servicestack/client) ServiceStack Client Libraries also received updates.


### Testing

The `AppSelfHostBase` base class commonly used for multi-platform .NET Core / .NET Framework integration tests now allows
specifying individual Service classes to allow for more focused tests:

 ```csharp
class AppHost : AppSelfHostBase
{
    public AppHost()
        : base(nameof(MyTests), typeof(TestServices1), typeof(TestServices2)) {}

    public override void Configure(Container container)
    {
        //...
    }
}
```

### AutoQuery

- Handle matching tables using **snake_case** in AutoGen

Add

- `AddDataContractAttributes` to include `[DataContract]` and `[DataMember]` attributes required by gRPC
- `AddIndexesToDataMembers`  to include `Order` DataMember property required by gRPC

### ServiceStack Auth

- `MicrosoftGraphAuthProvider` now populating Application roles by @DeonHeyns
  Firstly you need to assign the App Roles via Azure AD.  
  https://docs.microsoft.com/en-us/azure/active-directory/develop/howto-add-app-roles-in-azure-ad-apps#assign-users-and-groups-to-roles

- `LoadUserAuthInfoFilterAsync` added to `AuthProvider`
- `GetAuthorization()`, `GetBearerToken()` and `GetJwtToken()` can be overridden in AppHost to customize how each are populated from HTTP Request
- JWT ids (`jti` ) are now populated with unique Id, by default uses auto incrementing id
    - `ResolveJwtId` to customize Unique JWT Id Generation
    - `ResolveRefreshJwtId` to customize Unique JWT Refresh Token Id Generation
    - Can use `InvalidateJwtIds` to invalidate specific JWT by Id
- `JwtAuthProvider.Dump(jwt)` and `PrintDump(jwt)` APIs can be used to dump a human-friendly view of the contents of a JWT Token

### ServiceStack

- Can override `ResolveLocalizedStringFormat` in AppHost to localize all built-in end-user visible Error Messages.
- Added `IsDevelopmentEnvironment()`, `IsStagingEnvironment()` and `IsProductionEnvironment()` AppHost convenience ext methods


```csharp
public interface ITypedFilterAsync
{
    Task InvokeAsync(IRequest req, IResponse res, object dto);
}

public interface ITypedFilterAsync<in T>
{
    Task InvokeAsync(IRequest req, IResponse res, T dto);
}    
```

```csharp
public interface IAppHost
{
    // Add an Async Request Filter for a specific Request DTO Type
    void RegisterTypedRequestFilterAsync<T>(Func<IRequest, IResponse, T, Task> filterFn);    

    // Add ITypedFilterAsync as an Async Typed Request Filter for a specific Request DTO Type
    void RegisterTypedRequestFilterAsync<T>(Func<Container, ITypedFilterAsync<T>> filter);}

    // Add an Async Request Filter for a specific Response DTO Type
    void RegisterTypedResponseFilterAsync<T>(Func<IRequest, IResponse, T, Task> filterFn);

    // Add ITypedFilterAsync as an Async Typed Request Filter for a specific Request DTO Type
    void RegisterTypedResponseFilterAsync<T>(Func<Container, ITypedFilterAsync<T>> filter);        
}
```

### Metadata

- Tag names are sorted alphabetically
- Tag filters can be deep linked + supports back/forward page navigation


### Fluent Validation

- FluentValidation upgraded to v9.5.1

### Server Sent Events

- Add `StopAsync()` API
- `Stop()` and `StopAsync()` APIs explicitly disposes all active subscriptions which is called when AppHost disposes
- `AppHost.Dispose()` is explicitly called during `IApplicationLifetime.ApplicationStopping` event

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
