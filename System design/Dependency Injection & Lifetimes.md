
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

* When you need to use `HttpClient`, transient means everytime you have to re-establish the network connection, making pool useless

* Disposing `HttpClient` doesn't immediately release the underlying socket — it lingers in for a while (not reusable). Under load, you exhaust the port pool: **socket exhaustion** and will make app unresponsive, as it can only open a certain number of sockets.

**2. One static/Singleton `HttpClient`**
* Domain names like `github.com` is just human friendly, routers only understand raw ip - `140.282.121.1`. When you tell `HttpClient` to connect to `github.com`, it does a DNS lookup to get ip address. `HttpClient` then opens a TCP connection to it.
* Once connection is opened, domain name is completely ignored. With singleton lifetime, connections are opened indefinitely. So if GitHub changes their ip address, `HttpClient` won't pick it up as it already has an active open socket to old ip address, it will never look at the domain name again until app is restarted.

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