We need the following nuget packages to sue DbContext class
microsoft.entityframeworkcore 

choosing a database
We use entityframeworkcore.sqlite


This allows us to add migration, drop databse etc
microsoft.entityframeworkcore.tools

migration, we can use it for initial migration, from no db to creating a db
`add-migration CityInfoDBInitialMigration`

This will generate a snapshot ( of current context model ) and our migration

`update-database` to apply migration

To remove a bad migration that hasn't applied yet, do
`Remove-Migration`

#### Seeding the databse - provide some sample data


#### SQL injection
![[Pasted image 20240711021411.png]]



#### Automapper
automapper



ActionResult vs IActionResult

- **Use `IActionResult` when:**
    
    - You want the flexibility to return different types of responses.
    - You don't need the benefits of type inference and strong typing.
- **Use `ActionResult<T>` when:**
    
    - You want to take advantage of strong typing and type inference.
    - You want to improve API documentation and client code generation.
    - You are returning a specific type but might also return other `ActionResult` types (like `NotFound`, `BadRequest`, etc.).


If you have a parameter in controller without specifying source, it's assumed to be a query string.
`GetCity(int id, bool includePointsOfInterest = false)`
Here if we mark `includePointsOfInterest` as `[FromBody]` it would not work.
`[FromBody]` needs to be marked on complex objects.


##### Searching, Filtering and Paging
Search is like searching for substring. It allows you to search for things that may exist in the collection.
Filtering is like filtering all cities with name Sydney. It allows you to be precise.

#### Deferred Execution
By working with `IQueryable`, we can build the query bit by bit, and then execute when we really need it, rather than build a query and execute, and then execute again etc.

Execution is deferred until query is iterated over. Query is iterated over when
* foreach loop
* ToList or ToListAsync or ToArray etc
* SingletonQueries - Average, Count, FirstOrDefault

So something like `query.Where(xxx => yyy)` is not executed until for example `FirstOrDefault` is called


##### Paging
Collections may grow very large. We can implement paging on them.

Paging params are typically passed through query param
`https://host/api/cities?pageNumber=1&pageSize=5`

We want to make sure paging goes all the way to the db, otherwise we would still be fetching many many resources from the db which adds to performance.

We can use `Skip` and `Take` for paging data, they are both part of LINQ's deferred executions. They do not invoke the query immediately, instead modifying underlying query.



