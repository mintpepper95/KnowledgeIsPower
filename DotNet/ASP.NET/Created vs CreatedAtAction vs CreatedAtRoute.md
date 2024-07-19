[[#What are these methods for and explains the difference between the 3 methods]]
[[#Explain the difference between RouteAttribute and Http{Verb} and why we should return IActionResult]]

---
## What are these methods for and explains the difference between the 3 methods

When you create a resource via HTTP POST, it's a good practice for the Web Api to return a `201 Created` response that includes a `Location` header with an URI that points to the newly created resource.

![[Pasted image 20240720002209.png]]


`Created` passes in a hard-coded uri for the new resource.
`CreatedAtAction` finds uri by method name.
`CreatedAtRoute` finds uri by route name.

### Created
`Created(string/Uri uri, object value)` - takes in a string or Uri and the created resource.
```cs
[Route("api/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{
    [HttpGet("{id}")]
    public IActionResult GetValue(int id, string version)
    {
        var value = $"Value {id} of version {version}";
        return Ok(value);
    }

    [HttpPost]
    public IActionResult Create()
    {
	    // anonymous type
        var createdResource = new { Id = 1, Version = "1.0" };
        string uri = $"https://www.example.com/api/values/{createdResource.Id}?version={createdResource.Version}";
        return Created(uri, createdResource);
        // Location: https://www.example.com/api/values/1?version=1.0
    }
}
```

### CreatedAtAction
Allows us to set location header by specifying name of action instead of hardcoded uri

#### CreatedAtAction (string actionName, object routeValues, object value)

```cs
[Route("api/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{
	// 'id' is a route param
    [HttpGet("{id}")]
    public IActionResult GetValue(int id, string version)
    {
        var value = $"Value {id} of version {version}";
        return Ok(value);
    }

    [HttpPost]
    public IActionResult Create()
    {
        var createdResource = new { Id = 1, Version = "1.0" };

		// Defines the route params and query strings
		// Location header will auto hold the right values
        var routeValues = new { id = createdResource.Id, version = createdResource.Version };
        
        return CreatedAtAction(nameof(GetValue), routeValues, createdResource);
        // Location: .../api/Values/1?version=1.0
    }
}
```


### CreatedAtRoute
```cs
[Route("api/[controller]")]
[ApiController]
public class ValuesController : ControllerBase
{
	// Setting the RouteName
    [HttpGet("{id}", Name = "NameForGetValueEndpoint")]
    public IActionResult GetValue(int id, string version)
    {
        var value = $"Value {id} of version {version}";
        return Ok(value);
    }

    [HttpPost]
    public IActionResult Create()
    {
        var createdResource = new { Id = 1, Version = "1.0" };
	    var routeParams = new { id = createdResource.Id, version = createdResource.Version }
        return CreatedAtRoute("NameForGetValueEndpoint", routeParams, createdResource);
        // Location: .../api/Values
    }

```



### Explain the difference between RouteAttribute and Http{Verb} and why we should return IActionResult

In `WebAPI` just use `Http{Verb}` instead or route attributes, also return `IActionResult` or `Task<IActionResult>` if async, to allow proper HTTP Verb returned from the action.

```cs
public class MyController : Controller
{
    // APPROACH 1
    // Route is method un-specific
    // Should only be used to specify base path
    [Route("api/books")]
    [HttpGet]
    public async IActionResult GetBooks() 
    {
        // Implementation
    }

    // APPROACH 2
    // This implies only GET requests are accepted on this route
    [HttpGet("api/books")]
    public async Task<IActionResult> GetBooks()
    {
        // Implementation
    }
}
```