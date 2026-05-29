
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

- Created once, reused forever (or until the app shuts down)
- Shared across all requests and all threads simultaneously
- Disposed only when the application shuts down
- **Best for:** Expensive-to-create services, caches, configuration wrappers, connection pools, `IMemoryCache`, `IHttpClientFactory`
- **Danger zone:** Must be thread-safe. Cannot safely hold request-specific state.

```
Request A:  [Controller] → MyService (instance #1)
Request B:  [Controller] → MyService (instance #1)   ← exactly the same object
Request C:  [Controller] → MyService (instance #1)   ← still the same
```

---

#### The Captive Dependency Problem ⚠️

This is the most common DI mistake. **A longer-lived service captures a shorter-lived one**, which then behaves as if it has the longer lifetime:

csharp

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
Transient  → can consume → Transient + Scoped + Singleton  (anything)
```

If a singleton needs scoped behaviour, inject `IServiceScopeFactory` and create a scope manually:


```csharp
public class MySingleton
{
    private readonly IServiceScopeFactory _scopeFactory;
    public MySingleton(IServiceScopeFactory f) => _scopeFactory = f;

    public async Task DoWorkAsync()
    {
        using var scope = _scopeFactory.CreateScope();
        var scoped = scope.ServiceProvider.GetRequiredService<IScopedService>();
        await scoped.DoSomethingAsync();
    } // scoped is disposed here
}
```

---

#### HttpClient Lifetime — a Special Case

`HttpClient` looks like it should be `Transient` — but that's a trap.

##### The two failure modes:

**1. `new HttpClient()` everywhere or Transient registration** Each instance holds a socket. Disposing `HttpClient` doesn't immediately release the underlying socket — it lingers in `TIME_WAIT`. Under load, you exhaust the port pool: **socket exhaustion**.

**2. One static/Singleton `HttpClient`** The instance never picks up DNS changes. If a service's IP changes, your singleton keeps hitting the old address until the app restarts: **DNS staleness**.

##### The solution: `IHttpClientFactory`

Introduced in .NET Core 2.1. Register it once:

```csharp
builder.Services.AddHttpClient(); // basic
// or named:
builder.Services.AddHttpClient("github", c =>
{
    c.BaseAddress = new Uri("https://api.github.com/");
    c.DefaultRequestHeaders.Add("Accept", "application/vnd.github.v3+json");
});
// or typed:
builder.Services.AddHttpClient<GitHubService>();
```

Under the hood, `IHttpClientFactory` manages a pool of `HttpMessageHandler` instances:

- **Handlers are pooled** — sockets are reused, no exhaustion
- **Handlers are rotated** every **2 minutes** by default — DNS changes are picked up
- **`HttpClient` itself is lightweight** — `IHttpClientFactory.CreateClient()` creates a new `HttpClient` wrapper each time, but it shares the pooled handler underneath

```
IHttpClientFactory
    └── Handler Pool
            ├── HttpMessageHandler (2 min lifetime) ← rotated for DNS
            ├── HttpMessageHandler (1.5 min, still active)
            └── HttpMessageHandler (being retired...)

Each CreateClient() call → new HttpClient wrapper → reuses a pooled handler
```

##### Typed clients and their gotcha:


```csharp
// Typed client — registered as Transient automatically
builder.Services.AddHttpClient<GitHubService>();

public class GitHubService
{
    private readonly HttpClient _client; // safe — factory manages the handler
    public GitHubService(HttpClient client) => _client = client;
}
```

Typed clients are **Transient** by design — get a fresh one each time, return it, let the factory handle the handler lifecycle. Don't cache a typed client in a singleton; you'd pin an old handler and reintroduce DNS staleness.

##### Configuring handler lifetime:


```csharp
builder.Services.AddHttpClient("myClient")
    .SetHandlerLifetime(TimeSpan.FromMinutes(5)); // default is 2 min
```


