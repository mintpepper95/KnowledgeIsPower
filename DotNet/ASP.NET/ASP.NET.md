Based on plural sight ASP.NET Core Web API Fundamentals course by Kevin Dockx

---
[[#Overview of Solution]]
[[#What is Middleware?]]



---
#### Overview of Solution

##### Updating a package
We can update a package by either using nuget package controller or changing it directly in the project file
![[Pasted image 20240710121129.png]]

##### Dependencies
Web API projects that target `MS.NET.Skd.Web` implicitly references `MS.AspNetCore.App` framework. Which you will find in the frameworks part under Dependencies. They contain ASP.NetCore related packages. The other framework, `MS.NETCore.App` contains packages to run a dot net app eg, system.io, system.linq etc.

Analysers help analyse your code during development.

##### Properties
`launchSettings.json` is for setting up env for local development.

![[Pasted image 20240710121337.png]]

##### appsettings
Contain app settings.

##### Controller
Contain api controllers.

##### Program
Start point of application.


##### Running project via .NET Cli
`dotnet run` when you are under a solution to run it, when using `dotnet run`, http profile is used by default. To use https, `dotnet run --launch-profile https`



#### What is Middleware?
![[Pasted image 20240710124057.png]]
Software that assembled into the application pipeline to handle requests and responses.
Basically each component in this pipeline process the incoming requests and can pass it to the next middleware or handle directly (not calling next, eg. handling error, responding with cache).

They are executed in the order they are added to the pipeline.

Each middleware receives a `HttpContext` object that contains info about the current http request. Middleware can then process/modify the request and pass it to next middleware or handle it directly. After all middleware components have processed the request, response is passed back through the pipeline, allowing middleware to modify response.

```csharp
// A web app needs to be hosted, and we need a builder to do it.
var builder = WebApplication.CreateBuilder(args);

// Add services to the container, there is where we do deps injection.
builder.Services.AddControllers();
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Configure the HTTP request pipeline. Note we can configure Environment in launchSettings or under project properties
if (app.Environment.IsDevelopment()) {
    // Adding swagger to request pipeline
    app.UseSwagger();
    app.UseSwaggerUI();
}

// This middleware ensures http calls are re-directed to https
app.UseHttpsRedirection();

// This middleware ensures authorisation policies
app.UseAuthorization();

// This middleware ensures requests are routed to the appropriate controller actions.
app.MapControllers();

app.Run();
```


#### Model binding
Basically converting http rquest data to .Net objects or complex types (models).
The benefit of model binding is it reduces chance of errors and improves readability.

```csharp
// https://contoso.com/api/pets/2?DogsOnly=true

[HttpGet("{id}")]
public ActionResult<Pet> GetById(int id, bool dogsOnly)
```

By default, model binding gets data in the form of key value pair from searching through following sources in an http request.
1. form fields [FromForm]
2. request body [FromBody]
3. route data [FromRoute]
4. query string params [FromQuery]
If not correct, we can use an attribute to specify the source.