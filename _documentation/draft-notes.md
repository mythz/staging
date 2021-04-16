
-----

### Postman2 Support

`postman2` {to:'$HOST'} feature,postman Configure support for Postman2
https://forums.servicestack.net/t/postman-v2-format-support/9424/5

### Desktop

Added heroicons to ServiceStack.Desktop

/lib/svg/heroicons/outline/academic-cap.svg
/lib/svg/heroicons/solid/academic-cap.svg

### GitHubGateway

- Add `GetRateLimits()` and `GetRateLimitsAsync()` APIs

- Add `DumpTable()` and `PrintDumpTable()` APIs to generate Markdown ascii tables from a list of POCOs
  //TODO Example

### Add ServiceStack Reference

Use:

```csharp
TypeScriptGenerator.InsertTsNoCheck = true;
```

To generate `// @ts-nocheck` at the top of your TypeScript dtos.ts.

```csharp
TypeScriptGenerator.UseNullableProperties = true;
```

To generate nullable properties instead of optional properties, e.g:

```ts
export class Data
{
    public value: number|null;
    public optionalValue: number|null;
    public text: string|null;
}
```

### Inspect Utils

//INSPECT_VARS=.gistrun/vars.json

## ProcessUtils

Used in gist.cafe

### Non Nullable Reference Types

If you've enabled [nullable reference types](https://devblogs.microsoft.com/dotnet/try-out-nullable-reference-types/) in your code-base
ServiceStack will report it as a **Required** property in its metadata pages, however they should be used in conjunction with a Validation
rule to assert that API Requests match the state of your code-base, using either a `.NotNull()` Fluent Validation rule or
`[ValidateNotNull]` declarative validation attribute, e.g:

```csharp
public class MyRequest
{
    [ValidateNotNull]
    public string A { get; set; }
    public string? B { get; set; }
}
```

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

### Swift Add ServiceStack Reference

The Swift Add ServiceStack Reference support has been rewritten to conform to Apple's standardized
[Swift's Codable support](https://developer.apple.com/documentation/foundation/archives_and_serialization/encoding_and_decoding_custom_types)
whose built-in compiler support results in much cleaner DTOs and more reusable DTOs that can be reused as data models
throughout your App where they can be saved as loaded with Swift's Serializers and interoperate
with other libraries like [gist.cafe's Swift Inspect](https://gist.cafe/#project/swift)
which is now integrated into our
[ServiceStack.Swift](https://github.com/ServiceStack/ServiceStack.Swift)

### Codable DTOs

ServiceStack now generates clean "implementation-free" Codable DTOs for all Types where they're supported by Swift, e.g:

```swift
// @Route("/posts/{Id}", "GET")
public class GetPost : IReturn, IGet, Codable
{
    public typealias Return = GetPostResponse

    public var id:Int?
    public var include:String?

    required public init(){}
}
```

But will also auto generate the Codable DTO implementations for Types that the Swift compiler's built-in Codable generation doesn't support,
namely inherited Types & types with self references. This limitation means that all AutoQuery DTOs, since they inherit from `QueryDb<T>`,
will have their Codable implementation generated with their DTO, e.g:

```swift
// @Route("/customers", "GET")
// @Route("/customers/{CustomerId}", "GET")
public class QueryCustomers : QueryDb<Customers>, IReturn, IGet
{
    public typealias Return = QueryResponse<Customers>

    public var customerId:Int?

    required public init(){ super.init() }

    private enum CodingKeys : String, CodingKey {
        case customerId
    }

    required public init(from decoder: Decoder) throws {
        try super.init(from: decoder)
        let container = try decoder.container(keyedBy: CodingKeys.self)
        customerId = try container.decodeIfPresent(Int.self, forKey: .customerId)
    }

    public override func encode(to encoder: Encoder) throws {
        try super.encode(to: encoder)
        var container = encoder.container(keyedBy: CodingKeys.self)
        if customerId != nil { try container.encode(customerId, forKey: .customerId) }
    }
}
```

Should you want to, you can force Types to generate their Codable implementation with the new `[Synthesize]` attribute, e.g:

```csharp
[Synthesize]
public class MyRequest { ... }
```

### Swift .NET DTO Type Converters

To help with .NET Interop this release includes utility methods to convert to/from native Swift Types and .NET DTO Dart Types:

- `func fromDateTime(_ jsonDate:String) -> Date?` - From .NET DateTime (WCF JSON or ISO Date) to Swift Date
- `func toDateTime(_ dateTime:Date) -> String` - From Swift Date to .NET DateTime (WCF JSON Date)
- `func fromTimeSpan(timeSpan:String) -> TimeInterval?` - From .NET TimeSpan (XSD Duration) to Swift TimeInterval
- `func toTimeSpan(timeInterval:TimeInterval) -> String` - From Swift TimeInterval to .NET TimeSpan (XSD Duration)
- `func fromByteArray(_ base64String:String) -> [UInt8]` - From .NET byte[] (Base64 String) to Swift [UInt8]
- `func toByteArray(_ bytes:[UInt8]) -> String` - From Swift [UInt8] to .NET byte[] (Base64 String)
- `func fromGuid(_ guid:String) -> String` - From .NET Guid to Guid string
- `func fromByteArray(_ base64String:String) -> [UInt8]` - From Guid string to .NET Guid

### Swift Value Type Converters

There's also extensible support for custom Type serialization when you want to serialize a custom .NET Type directly
into a native Swift Type. These extensible APIs are used for converting a .NET `TimeSpan` into a Swift `TimeInterval`.

First a Type Alias is registered with Swift code-gen to specify which Type `TimeSpan` should be generated as
in the generated Swift DTOs:

```csharp
SwiftGenerator.TypeAliases["TimeSpan"] = "TimeInterval";
```

Then the Swift client Apps would need to register a Convert that implements `StringConvertible` specifying which Type
it's converting and how to convert to & from String values.

```swift
Converters.register(TimeIntervalConverter())
```

The built-in `TimeIntervalConverter` uses the TimeSpan DTO Type Converters above to do this:

```swift
public class TimeIntervalConverter : StringConvertible {
    public var forType = Reflect<TimeInterval>.typeName

    public func fromString<T>(_ type: T.Type, _ string: String) -> T? {
        return fromTimeSpan(string) as? T
    }
    public func toString<T>(instance: T) -> String? {
        return toTimeSpan(instance as! TimeInterval)
    }
}
```

An alternative solution is to have the Swift Type implement the [LosslessStringConvertible](https://developer.apple.com/documentation/swift/losslessstringconvertible) protocol where you can then just register the Type itself, e.g:

```swift
Converters.register(TimeInterval.self)
```

### Upgrade to ServiceStack.Swift v5

To use the new Codable support your Apps will need to upgrade to use the latest **v5.x** ServiceStack package:

```swift
dependencies: [
    .package(name: "ServiceStack", 
        url: "https://github.com/ServiceStack/ServiceStack.Swift.git", 
        Version(5,0,0)..<Version(6,0,0)),
],
```

### Revert to legacy Swift DTOs

If you're not ready to upgrade to use the latest Swift 5 and DTOs with Codable support you can revert to use previous Swift DTO implementation
by using `swift4` when downloading and updating your Server DTOs:

    $ x swift4 https://techstacks.io

You'll also need to use **v1.x** of the **ServiceStack.Swift** library:

```swift
dependencies: [
    .package(name: "ServiceStack", url: "https://github.com/ServiceStack/ServiceStack.Swift.git", 
        Version(1,0,0)..<Version(2,0,0)),
],
```

We'll maintain the old Swift version until the next major ServiceStack release to enable staged migrations to the new Swift Client Library and Codable DTOs.

### Dart Add ServiceStack Reference

The Dart library and generated DTOs also received a lot of attention to add support for minified production Flutter Web builds now that
[Flutter Apps support Web Apps](https://flutter.dev/web) which requires every DTO Type now generated to include its non minified Type name.

The Dart [servicestack](https://pub.dev/packages/servicestack) client library now also includes AutoQuery & Auto CRUD base classes and interfaces
which reduces the amount of generated code each API would use and enables code sharing & reuse when importing multiple ServiceStack references
from different remote hosts as they how share the same library base classes and interfaces.

### Dart .NET Type Converters

To help with .NET Interop this release includes utility methods to convert to/from native Dart Types and .NET DTO Dart Types:

- `DateTime fromDateTime(String jsonDate)` - From .NET DateTime (WCF JSON or ISO Date) to Dart DateTime
- `String toDateTime(DateTime dateTime)` - From Dart DateTime to .NET DateTime (WCF JSON Date)
- `Duration fromTimeSpan(String str)` - From .NET TimeSpan (XSD Duration) to Dart Duration
- `String toTimeSpan(Duration duration)` - From Dart Duration to .NET TimeSpan (XSD Duration)
- `Uint8List fromByteArray(String base64String)` - From .NET byte[] (Base64 String) to Dart Uint8List
- `String toByteArray(Uint8List bytes)` - From Dart Uint8List to .NET byte[] (Base64 String)
- `String fromGuid(String guid)` - From .NET Guid to Guid string
- `String toGuid(String guid)` - From Guid string to .NET Guid

### Debug Logging

To help with diagnosing production runtime issues you can enable debug logging can be enabled with:

 ```dart
void main() {
  Log.levels.add(LogLevel.Debug);
  runApp(MyApp());
}
```

A `ConsoleLogger` with **Warn** and **Error** log levels are enabled by default, this can be disabled by clearing the
log levels or replacing the Logger implementation:

 ```dart
Log.levels.clear(); //or
Log.logger = new NullLogger();
```

### Java & Kotlin ServiceStack Reference

Our [Android](https://mvnrepository.com/artifact/net.servicestack/android) and [JVM](https://mvnrepository.com/artifact/net.servicestack/client) ServiceStack Client Libraries also received updates.

### Utils

Static helpers on `Utils.*` to help translating between built-in JVM and .NET DTO Types:

- `Date fromDateTime(String jsonDate)` - From .NET DateTime (WCF JSON or ISO Date) to JVM Date
- `String toDateTime(Date date)` - From JVM Date to .NET DateTime (WCF JSON Date)
- `TimeSpan fromTimeSpan(String xsdDuration)` - From .NET TimeSpan (XSD Duration) to Java TimeSpan
- `String toTimeSpan(TimeSpan timeSpan)` - From Java TimeSpan to .NET TimeSpan (XSD Duration)
- `UUID fromGuid(String guid)` - From .NET Guid to JVM UUID
- `String toGuid(UUID uuid)` - From JVM UUID to .NET Guid
- `byte[] fromByteArray(String base64)` - From .NET byte[] (Base64 String) to JVM signed byte[]
- `String toByteArray(byte[] bytes)` - From JVM signed byte[] to .NET byte[] (Base64 String)

### TypeScript Service Client

Replaced abandoned `fetch-everywhere` with `cross-fetch` to reference latest `node-fetch` without vulnerability warnings.

- Removed all global constants so UMD `/js/servicestack-client.js` works when included as a defer script
- `bindHandlers()` invokes callback bound with the target element containing the declarative event handler

```ts
// ES2017 flatMap polyfill 
function flatMap(f: Function, xs: any[]): any;
// return distinct string collection
function uniq(xs: string[]): string[];
// encode HTML
function enc(o: any): string;
// convert JS object into HTML attributes string
function htmlAttrs(o: any): string;
// return index of first needle found
function indexOfAny(str: string, needles: string[]): number;
// return left part of needle
function leftPart(strVal: string, needle: string): string;
// return right part of needle
function rightPart(strVal: string, needle: string): string;
// return last left part of needle
function lastLeftPart(strVal: string, needle: string): string;
// return last right part of needle
function lastRightPart(strVal: string, needle: string): string;
// return new object containing only specified keys
function onlyProps(obj: any, keys: string[]): any;
// return unique keys in object array
function uniqueKeys(rows: any[]): string[];

// .NET DTO Conversion Utils
function fromDateTime(dateTime: string): Date;
function toDateTime(date: Date): string;
function fromTimeSpan(xsdDuration: string): string;
function toTimeSpan(xsdDuration: string): string;
function fromGuid(xsdDuration: string): string;
function toGuid(xsdDuration: string): string;
function fromByteArray(base64: string): Uint8Array;
function toByteArray(bytes: Uint8Array): string;
function toBase64String(source: string): string;
```

Included JSV Serializer which can be used to send deep object graphs in QueryStrings or FormData complex type properties:

```ts
class JSV {
    static ESCAPE_CHARS: string[];
    static encodeString(str: string): string;
    static encodeArray(array: any[]): string;
    static encodeObject(obj: any): string;
    static stringify(obj: any): any;
}
```

Inspect utils to dump object graphs in human-friendly view:

```ts
class Inspect {
    // Serialize object so it can be inspected after its run, e.g. in gist.cafe
    static vars(obj: any): void;
    // Dump object into human-readable format
    static dump(obj: any): string;
    static printDump(obj: any): void;
    // Dump tabular results into human-readable ascii markdown table
    static dumpTable(rows: any[]): string;
    static printDumpTable(rows: any[]): void;
}

// Text Alignment Functions
function alignLeft(str: string, len: number, pad?: string): string;
function alignCenter(str: string, len: number, pad?: string): string;
function alignRight(str: string, len: number, pad?: string): string;
function alignAuto(obj: any, len: number, pad?: string): string;
```

Included Google Clojure `StringBuffer` API for optimal string concatenation in JS VMs:

```ts
class StringBuffer {
    buffer_: string;
    constructor(opt_a1?: any, ...var_args: any[]);
    set(s: string): void;
    append(a1: any, opt_a2?: any, ...var_args: any[]): this;
    clear(): void;
    getLength(): number;
    toString(): string;
}
```

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

### Declarative Nested Validators

[Declarative validation](https://docs.servicestack.net/declarative-validation) is our preferred way to declare DTO validation rules
since they're a natural extension for defining additional validation rules on Typed DTOs which provide more detailed information about
your Service Contracts which can't be expressed by a programming languages Type System.

The new support for nested child validators removes another use-case that previously required manual configuration using Fluent Validation
APIs, now any child validators on nested types or nested collection types will be automatically registered, e.g:

```csharp
public class CreateSite : ICreateDb<Site>, IReturn<CreateSiteResponse>
{
    [ValidateMaximumLength(20)]
    public string Site { get; set; }
    public List<SiteProperty> Properties { get; set; }
    public SiteContact Contact { get; set; }
}

public class SiteProperty
{
    [ValidateMaximumLength(20)]
    public string Name { get; set; }
    [ValidateNotEmpty]
    public string Value { get; set; }
}

public class SiteContact
{
    [ValidateNotEmpty]
    public string FirstName { get; set; }
    [ValidateNotEmpty]
    public string LastName { get; set; }
    [ValidateEmail]
    public string Email { get; set; }
}
```

When a child collection validator fails, it includes the item index and nested property name in the validation error field name, e.g:

```js
{
    ErrorCode: "MaximumLength",
    Message: "The length of 'Name' must be 20 characters or fewer. You entered 32 characters.",
    Errors: [{
        FieldName: "Properties[0].Name"
        ErrorCode: "MaximumLength",
        Message: "The length of 'Name' must be 20 characters or fewer. You entered 32 characters.",
    }]
}
```

Note when mixing with existing custom validators:

- It will also auto register custom child validators defined using **FluentValidation** APIs
- It wont register child validators for containing Type custom validators defined using FluentValidation APIs

This is to avoid duplicate registration of validators, so when defining a custom validator for a containing Type (e.g. Request DTO)
it will take over fluent validation registration for that type and you would need to manually register its child validators in its custom implementation.

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
