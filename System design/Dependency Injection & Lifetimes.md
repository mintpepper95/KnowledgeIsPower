
### Dependency Injection in .NET

#### Why DI over `new`?

When you write `new MyService()`, you're hardcoding a dependency. 
The class that calls this now owns the lifetime, knows the concrete type, and makes it harder to test as it uses real implementation. 

Problems compound as your graph grows: 

```
new OrderService(
	new PaymentService(
		new HttpClient(...), new Logger(...)), 
	new EmailService(...)
)
```

DI flips this. You declare _what_ you need, and a container resolves _how_ to build it. This gives you:

- **Testability** — swap real services for mocks without changing consuming code
- **Centralised configuration** — one place to control how things are built and wired
- **Lifetime management** —  container decides when to create and dispose, not scattered call sites
- **Loose coupling** — depend on interfaces, not concrete types; swap implementations freely
- **Automatic disposal** — `IDisposable` services get cleaned up when their scope ends

---
#### Transient, Scope and Singleton

##### Transient — `AddTransient<T>()`

```csharp
builder.Services.AddTransient<IMyService, MyService>();
```

- Created fresh on every injection point, every request, so never shared between consumers, even within same HTTP request
- Disposed at the end of the scope in which they were resolved
- **Best for:** Lightweight, stateless services with no shared state concerns (validators, formatters, calculators)
- **Avoid when:** Construction is expensive, or the service holds state that should survive a unit of work

```
Request A:  [Controller]   → MyService (instance #1)
            [OtherService] → MyService (instance #2)   ← different!

Request B:  [Controller]   → MyService (instance #3)
```

---
##### Scoped — `AddScoped<T>()`

**One instance per scope** — in web apps, one per HTTP request.

```csharp
builder.Services.AddScoped<IMyService, MyService>();
```

- Shared across all injection points _within the same request_
- A new instance is created for each new request/scope
- Disposed when the scope ends (end of the HTTP request)
- **Best for:** `DbContext`, unit-of-work patterns, anything that should share state within a request but not across requests
- **Classic example:** Entity Framework's `DbContext` — you want one per request so change tracking is consistent, but not shared across requests (which would cause threading bugs)

```
Request A:  [Controller]   → MyService (instance #1)
            [OtherService] → MyService (instance #1)   ← same!

Request B:  [Controller]   → MyService (instance #2)   ← new instance
            [OtherService] → MyService (instance #2)
```

---

##### Singleton — `AddSingleton<T>()`

**One instance for the entire application lifetime.**

```csharp
builder.Services.AddSingleton<IMyService, MyService>();
```

- Created only once
- Shared across all requests and all threads simultaneously
- Disposed only when the application shuts down
- **Best for:** Expensive-to-create services, caches, configuration wrappers, connection pools, `IMemoryCache`, `IHttpClientFactory`
- **Danger zone:** Must be thread-safe. If your singleton holds mutable state, may cause race condition.

```
Request A:  [Controller] → MyService (instance #1)
Request B:  [Controller] → MyService (instance #1)   ← exactly the same object
Request C:  [Controller] → MyService (instance #1)   ← still the same
```

---

### The Captive Dependency Problem

**A longer-lived service captures a shorter-lived one**, which then behaves as if it has the longer lifetime:

```csharp
// ❌ WRONG — Singleton captures a Scoped service
public class MySingleton
{
    private readonly IScopedService _scoped; // This scoped instance is now stuck forever!
    
    public MySingleton(IScopedService scoped) => _scoped = scoped;
}

builder.Services.AddSingleton<MySingleton>();
builder.Services.AddScoped<IScopedService, ScopedService>();
```

At runtime, .NET will **throw an `InvalidOperationException`** in development (scope validation is on by default). In production without validation it silently breaks — the "scoped" service becomes effectively a singleton, sharing state across requests.

**The safe rule:**
```
Singleton  → can consume → Singleton only
Scoped     → can consume → Scoped + Singleton
Transient  → can consume → Transient + Scoped + Singleton
```

---

#### HttpClient Lifetime — a Special Case

`HttpClient` looks like it should be `Transient` — but that's a trap.

Note `HttpClient` in Blazor is just `fetch()` API, so this whole thing doesn't apply.
##### The two failure modes:

**1. `new HttpClient()` everywhere or Transient registration** 
Each instance of the client will create a new connection pool. A connection pool keeps a collection of established sockets warm and ready so when a request comes in, you can use the socket in the pool instead of re-establishing network connection.

* When you need to use `HttpClient`, transient means every time you have to re-establish the network connection, making pool useless

* Disposing `HttpClient` doesn't immediately release the underlying socket — it lingers in for a while (not reusable). Under load, you exhaust the port pool: **socket exhaustion** and will make app unresponsive, as it can only open a certain number of sockets.

**2. One static/Singleton `HttpClient`**
* Domain names like `github.com` is just human friendly, routers only understand raw ip - `140.282.121.1`. When you tell `HttpClient` to connect to `github.com`, it does a DNS lookup to get ip address. `HttpClient` then opens a TCP connection to it.
* Once connection is opened, domain name is completely ignored. With singleton lifetime, connections are opened indefinitely. So if GitHub changes their ip address, `HttpClient` won't pick it up as it already has an active open socket to old ip address, it will never look at the domain name again until app is restarted.

| Lifetime                           | Socket exhaustion | Stale DNS |
| ---------------------------------- | ----------------- | --------- |
| Transient/Scoped (new per request) | Yes               | No        |
| Singleton                          | No                | Yes       |
| `IHttpClientFactory`               | No                | No        |

##### The solution: `IHttpClientFactory`

```csharp
// This will register IHttpClientFactory
builder.Services.AddHttpClient(
	"github", 
	c => { c.BaseAddress = new Uri("https://api.github.com/"); }
);


// We can now use the injected named client
public class XXXService { 
	private readonly HttpClient _client; 
	public MyService(IHttpClientFactory factory) {
		_client = factory.CreateClient("github"); // Created on demand } 
	}
}
```

`IHttpClientFactory`                         - Singleton
`HttpClient` (from `CreateClient`)    - **Transient**, new instance each call`HttpMessageHandler` (underneath)    - **Pooled** — reused for ~2 minutes


Under the hood, `IHttpClientFactory` manages a pool of `HttpMessageHandler` instances:
- **Handlers are pooled** — sockets are reused, no exhaustion
- **Handlers are rotated** every **2 minutes** by default — so DNS changes are picked up
- **`HttpClient` itself is lightweight** — `IHttpClientFactory.CreateClient()` creates a new `HttpClient` wrapper each time, but it shares the pooled handler underneath



---

When you wrap an `HttpClient` in a `using` block, or when a Scoped/Transient lifecycle ends and .NET disposes of it, you aren't actually closing the underlying network socket right away.

Instead, `HttpClient` tells the operating system, _"I am done with this connection."_ The operating system replies, _"Okay, I will close it, but I have to put this socket into a safety state called **`TIME_WAIT`** for the next 2 to 4 minutes."_

### Why does the OS do this?

The `TIME_WAIT` state is a built-in feature of the global TCP/IP network protocol. The OS keeps that specific port reserved just in case any stray, delayed network packets from the server arrive late. If it reopened that port immediately for a different request, those late packets would corrupt the new data stream.

### The Crash Scenario

If your web API gets a spike in traffic and handles 1,000 requests per second, and each request creates, uses, and immediately disposes of an `HttpClient`:

1. You open 1,000 sockets.
    
2. You immediately dispose of them.
    
3. Those 1,000 sockets sit in `TIME_WAIT` for 4 minutes.
    

Within those 4 minutes, you will have accumulated **240,000 sockets** sitting completely dead in the `TIME_WAIT` state. Because an operating system has a hard limit on how many outbound ports it can open, your server completely runs out of sockets and throws a fatal `SocketException` (_"No buffer space available"_).

## How `IHttpClientFactory` Solves the Paradox

This is exactly why `IHttpClientFactory` was created. It splits the `HttpClient` into two pieces:

1. **The Wrapper (`HttpClient`):** This is the lightweight object injected into your Transient or Scoped service. When your service lifetime ends, this wrapper **is** disposed of immediately. It's cheap and goes straight to the garbage collector.
    
2. **The Connection Pool (`HttpMessageHandler`):** This is the heavy piece that actually holds the open OS network sockets. The factory keeps this pool alive in the background.
    

When your Scoped service spins up, the factory hands it an `HttpClient` wrapper hooked up to an _already open_ socket from the pool. When your service dies, the wrapper is destroyed, but the factory slides that underlying socket right back into the pool to be reused for the next request.

No sockets get put into `TIME_WAIT`, and your server stays completely stable.



---




**DI container** (or IoC container). In .NET the built-in one is `IServiceCollection` / `IServiceProvider`.

- **`IServiceCollection`** — where you _register_ services (the "write" side). This is what you use in `Startup.cs` / `Program.cs` during app startup.
- **`IServiceProvider`** — where you _resolve_ services at runtime (the "read" side). The framework builds this from the collection and uses it to inject dependencies automatically.

The container looks at the registered graph, instantiates everything in the right order, and injects them.

**Third-party alternatives** exist (Autofac, Lamar, DryIoc) that plug into the same `IServiceCollection` abstraction but offer more advanced features like property injection, decorators, or dynamic registration. The built-in one covers most scenarios.


### Cartesian explosion

### The Simple Example: A Blog

Imagine you have a single **Blog**.

- This Blog has **3 Posts**.
    
- This Blog also has **4 Tags** (like "Tech", "News", "Coding", "Fun").
    

You want to load the Blog, its Posts, and its Tags all at once using this code:

C#

```
var blog = dbContext.Blogs
    .Include(b => b.Posts)
    .Include(b => b.Tags)
    .First();
```

### The "Explosion"

In reality, there are only **8 pieces of data** in your database: 1 Blog + 3 Posts + 4 Tags.

However, because EF Core uses a single `LEFT JOIN` query by default, the database creates a grid combining every Post with every Tag.

The database calculates the rows by multiplying: **1 (Blog) × 3 (Posts) × 4 (Tags) = 12 Rows.**

The database will send this flat grid back to your app:

|**Blog Data**|**Post Data**|**Tag Data**|
|---|---|---|
|My Blog|Post 1|Tech|
|My Blog|Post 1|News|
|My Blog|Post 1|Coding|
|My Blog|Post 1|Fun|
|My Blog|Post 2|Tech|
|...and so on|...|...|

Notice how "My Blog" and "Post 1" are completely duplicated over and over again?

If your blog had **100 Posts** and **50 Tags**, the database would send back **5,000 rows** of heavily duplicated data across your network, just to deliver 151 actual items. This chokes your database CPU, network bandwidth, and your app's memory.