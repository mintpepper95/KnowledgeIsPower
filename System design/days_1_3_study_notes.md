# Study Notes — Days 1–3: DDD, SOLID & Clean Architecture
> Senior / Staff .NET Interview Prep · C# focused

---

# DAY 1–2: Domain-Driven Design (DDD) & SOLID

---

## Part 1 — Domain-Driven Design (DDD)

### What is DDD and why does it matter at a senior level?

DDD is a software design approach that centres the codebase around the **business domain** — the real-world problem the software is solving. Rather than modelling data tables or API shapes, you model the business concepts, rules, and language directly in code.

> **Senior framing:** "DDD helps us ensure that complexity lives in the right place — in the domain logic — rather than leaking into controllers, database schemas, or service classes. It gives teams a shared language so that the code reads like the business problem it solves."

---

### Strategic DDD — The Big Picture

#### Ubiquitous Language

A single, shared vocabulary used by both developers and domain experts. The same words used in conversation must appear in the code.

**WooliesX example:**

| Business says | Bad code name | Good (ubiquitous) name |
|---|---|---|
| "Place an order" | `CreateOrderRecord()` | `PlaceOrder()` |
| "Fulfilment" | `ShippingManager` | `FulfilmentService` |
| "Scan item at checkout" | `AddProduct()` | `ScanItem()` |

---

#### Bounded Contexts

A **Bounded Context** is a clear boundary within which a model applies. The same word can mean different things in different contexts — that's fine, as long as each context is explicit about its own meaning.

**WooliesX monolith decomposition:**

```
┌────────────────────────────────────────────────────────────┐
│                    WOOLIES MONOLITH                        │
│  OrderController  ProductController  CustomerController    │
│  OrderService     ProductService     CustomerService       │
│  One giant DB schema with 200 tables                       │
└────────────────────────────────────────────────────────────┘

         Decompose by business capability ↓

┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│   CART CONTEXT   │  │CATALOGUE CONTEXT │  │FULFILMENT CONTEXT│
│                  │  │                  │  │                  │
│ - CartItem       │  │ - Product        │  │ - Order          │
│ - Promotion      │  │ - Category       │  │ - Shipment       │
│ - CartTotal      │  │ - StockLevel     │  │ - DeliverySlot   │
└──────────────────┘  └──────────────────┘  └──────────────────┘

┌──────────────────┐  ┌──────────────────┐
│ LOYALTY CONTEXT  │  │IDENTITY CONTEXT  │
│                  │  │                  │
│ - PointsBalance  │  │ - Customer       │
│ - Redemption     │  │ - Address        │
│ - Tier           │  │ - PaymentMethod  │
└──────────────────┘  └──────────────────┘
```

> **Interview insight:** Notice that "Product" exists in both the Cart and Catalogue contexts — but they are *different models*. In the Catalogue context, Product has rich data (description, images, ingredients). In the Cart context, a CartItem only needs ProductId, Name, and Price. You don't share the entity — you share an ID and translate at the boundary.

---

#### Context Map — How Bounded Contexts talk to each other

```
CATALOGUE ──(publishes ProductPriceChanged event)──► CART
CART      ──(publishes OrderPlaced event)──────────► FULFILMENT
FULFILMENT──(publishes OrderShipped event)─────────► LOYALTY
```

**Integration patterns between contexts:**
- **Anti-Corruption Layer (ACL)** — Translate a foreign model into your own. Prevents a legacy system's bad model from polluting your clean domain.
- **Published Language** — A shared event schema (e.g., Avro, JSON Schema) that all contexts agree on.
- **Conformist** — Downstream adopts the upstream model as-is. Simpler but riskier.

---

### Tactical DDD — Building Blocks in C#

#### 1. Entities

An object defined by its **identity**, not its attributes. Two customers with the same name are still different customers.

```csharp
public class Order
{
    public OrderId Id { get; private set; }
    public CustomerId CustomerId { get; private set; }
    public OrderStatus Status { get; private set; }
    private readonly List<OrderLine> _lines = new();
    public IReadOnlyList<OrderLine> Lines => _lines.AsReadOnly();

    // Business rule enforced here, not in a service
    public void AddItem(ProductId productId, int quantity, Money price)
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Cannot add items to a non-draft order.");

        var existing = _lines.FirstOrDefault(l => l.ProductId == productId);
        if (existing != null)
            existing.IncreaseQuantity(quantity);
        else
            _lines.Add(new OrderLine(productId, quantity, price));
    }
}
```

**Senior insight:** Notice there are no public setters. All state changes go through methods that enforce business rules. This is called an **Aggregate with an invariant** — the rule "cannot add items to a non-draft order" is always upheld because the only way to change state is through `AddItem`.

---

#### 2. Value Objects

An object defined by its **value**, not its identity. Two `Money` objects with the same amount and currency are identical and interchangeable. They are **immutable**.

```csharp
public record Money(decimal Amount, string Currency)
{
    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new InvalidOperationException("Cannot add different currencies.");
        return new Money(Amount + other.Amount, Currency);
    }

    public Money ApplyDiscount(decimal percent) =>
        new Money(Amount * (1 - percent / 100), Currency);
}

// Usage — reads like the domain
var subtotal = new Money(49.99m, "AUD");
var shipping = new Money(9.95m, "AUD");
var total = subtotal.Add(shipping); // new Money(59.94, "AUD")
```

**Why use records here?** C# `record` gives structural equality for free — two `Money(50, "AUD")` instances are equal without writing `Equals()` yourself.

**WooliesX examples of Value Objects:** `Money`, `Address`, `ProductSku`, `PhoneNumber`, `EmailAddress`, `DateRange` (for a delivery slot).

---

#### 3. Aggregates and the Aggregate Root

An **Aggregate** is a cluster of entities and value objects treated as a single unit. The **Aggregate Root** is the single entry point — nothing outside the aggregate can hold a reference to anything inside it directly.

```csharp
// Aggregate Root: Order
// Aggregate Members: OrderLine (cannot be accessed directly from outside)
public class Order  // <-- Aggregate Root
{
    public OrderId Id { get; private set; }
    private List<OrderLine> _lines = new();  // <-- internal to aggregate

    public void PlaceOrder()
    {
        if (!_lines.Any())
            throw new DomainException("Cannot place an empty order.");

        Status = OrderStatus.Placed;
        AddDomainEvent(new OrderPlacedEvent(Id, CustomerId, _lines));
    }
}

// Outside code — CORRECT
var order = orderRepository.GetById(orderId);
order.PlaceOrder();

// Outside code — WRONG (bypasses aggregate rules)
var line = orderLineRepository.GetById(lineId); // ❌ Never do this
line.SetPrice(newPrice);                         // ❌ Bypasses invariants
```

**Rule of thumb:** Each Aggregate should be saved in a single transaction. If you find yourself saving two aggregates in one transaction, you may have the wrong aggregate boundaries.

---

#### 4. Domain Events

Something significant that happened in the domain. Used to communicate between aggregates and bounded contexts without tight coupling.

```csharp
// Domain Event — something that happened
public record OrderPlacedEvent(
    OrderId OrderId,
    CustomerId CustomerId,
    IReadOnlyList<OrderLine> Lines,
    DateTime OccurredOn) : IDomainEvent;

// Raise it inside the aggregate
public class Order
{
    private List<IDomainEvent> _domainEvents = new();
    public IReadOnlyList<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();

    public void PlaceOrder()
    {
        // ... business rule checks ...
        Status = OrderStatus.Placed;
        _domainEvents.Add(new OrderPlacedEvent(Id, CustomerId, Lines, DateTime.UtcNow));
    }

    public void ClearDomainEvents() => _domainEvents.Clear();
}

// In your repository/UoW — dispatch events after saving
public async Task SaveChangesAsync()
{
    await _dbContext.SaveChangesAsync();

    var events = _dbContext.ChangeTracker
        .Entries<Entity>()
        .SelectMany(e => e.Entity.DomainEvents)
        .ToList();

    foreach (var @event in events)
        await _eventDispatcher.DispatchAsync(@event);
}
```

**Senior insight:** Domain events are dispatched *after* the DB save. This ensures you don't fire events for transactions that rolled back.

---

#### 5. Repositories

An abstraction over persistence. The domain doesn't know or care if data is in SQL Server, Cosmos DB, or an in-memory list.

```csharp
// In the Domain layer — just an interface
public interface IOrderRepository
{
    Task<Order> GetByIdAsync(OrderId id, CancellationToken ct = default);
    Task AddAsync(Order order, CancellationToken ct = default);
    Task<IEnumerable<Order>> GetByCustomerAsync(CustomerId customerId, CancellationToken ct = default);
}

// In the Infrastructure layer — the EF Core implementation
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _db;

    public OrderRepository(AppDbContext db) => _db = db;

    public async Task<Order> GetByIdAsync(OrderId id, CancellationToken ct = default)
        => await _db.Orders
            .Include(o => o.Lines)
            .FirstOrDefaultAsync(o => o.Id == id, ct)
            ?? throw new OrderNotFoundException(id);

    public async Task AddAsync(Order order, CancellationToken ct = default)
        => await _db.Orders.AddAsync(order, ct);
}
```

**What NOT to put in a repository:** Query logic that belongs in CQRS read models (e.g., `GetOrdersSortedByValueForDashboard()`). Repositories return domain aggregates. Report queries go elsewhere.

---

#### 6. Domain Services

Logic that doesn't naturally belong to any single entity or value object.

```csharp
// Can't put this on Order or Product — it involves both
public class PricingService
{
    private readonly IPromotionRepository _promotions;

    public PricingService(IPromotionRepository promotions) =>
        _promotions = promotions;

    public async Task<Money> CalculateFinalPrice(
        CartItem item, CustomerId customerId)
    {
        var promotions = await _promotions.GetActiveAsync();
        var applicable = promotions
            .Where(p => p.AppliesTo(item, customerId))
            .OrderByDescending(p => p.DiscountPercent)
            .FirstOrDefault();

        return applicable != null
            ? item.Price.ApplyDiscount(applicable.DiscountPercent)
            : item.Price;
    }
}
```

---

### When NOT to use DDD *(the staff-level answer)*

> "DDD adds real overhead — a learning curve for the team, more layers, and more ceremony for simple operations. For a CRUD-heavy microservice like a config management service or a simple notification preferences store, DDD is overkill. I'd reach for DDD when the domain is complex and frequently changing — like a cart, a pricing engine, or a fulfilment workflow — where encoding the business rules clearly in the domain layer pays dividends long-term."

---

## Part 2 — SOLID Principles in C#

### S — Single Responsibility Principle (SRP)

A class should have one reason to change.

```csharp
// ❌ Wrong — this class does too much
public class OrderService
{
    public void ProcessOrder(Order order)
    {
        // Validates the order
        if (order.Lines.Count == 0) throw new Exception("Empty order");

        // Saves to DB
        _db.Orders.Add(order);
        _db.SaveChanges();

        // Sends confirmation email
        _emailClient.Send(order.Customer.Email, "Order confirmed!");

        // Updates loyalty points
        _loyaltyService.AddPoints(order.Customer.Id, order.Total);
    }
}

// ✅ Correct — each concern is separated
public class OrderValidator { ... }
public class OrderRepository { ... }
public class OrderConfirmationEmailSender { ... }
public class LoyaltyPointsService { ... }

// Orchestrated by a thin command handler
public class PlaceOrderCommandHandler
{
    public async Task Handle(PlaceOrderCommand cmd)
    {
        _validator.Validate(cmd);
        await _repository.AddAsync(order);
        await _emailSender.SendConfirmationAsync(order);
        await _loyaltyService.AwardPointsAsync(order);
    }
}
```

---

### O — Open/Closed Principle (OCP)

Open for extension, closed for modification. Add new behaviour without touching existing code.

```csharp
// ❌ Wrong — every new promotion type requires editing this method
public decimal CalculateDiscount(Order order, string promotionType)
{
    if (promotionType == "PERCENTAGE") return order.Total * 0.1m;
    if (promotionType == "FLAT") return 5.00m;
    if (promotionType == "BOGO") return order.Lines.Min(l => l.Price);  // new requirement
    return 0;
}

// ✅ Correct — new promotion types are added without changing existing code
public interface IPromotion
{
    bool IsApplicable(Order order);
    decimal CalculateDiscount(Order order);
}

public class PercentageDiscount : IPromotion { ... }
public class FlatDiscount : IPromotion { ... }
public class BogoDiscount : IPromotion { ... }  // added without touching anything else

public class DiscountCalculator
{
    private readonly IEnumerable<IPromotion> _promotions;

    public decimal Calculate(Order order) =>
        _promotions
            .Where(p => p.IsApplicable(order))
            .Sum(p => p.CalculateDiscount(order));
}
```

---

### L — Liskov Substitution Principle (LSP)

Subtypes must be substitutable for their base types without breaking the program.

```csharp
// ❌ Wrong — Square breaks the contract of Rectangle
public class Rectangle
{
    public virtual double Width { get; set; }
    public virtual double Height { get; set; }
    public double Area() => Width * Height;
}

public class Square : Rectangle
{
    public override double Width { set { base.Width = base.Height = value; } }
    public override double Height { set { base.Width = base.Height = value; } }
}

// This breaks — caller expects Width and Height to be independent
Rectangle r = new Square();
r.Width = 4;
r.Height = 5;
Console.WriteLine(r.Area()); // Expected 20, got 25 — LSP violated

// ✅ Correct — use composition or a shared abstraction
public interface IShape { double Area(); }
public class Rectangle : IShape { ... }
public class Square : IShape { ... }
```

**Practical .NET example:** If you have `IReadOnlyRepository` and `IWriteRepository`, don't make `ReadOnlyRepository` throw `NotImplementedException` for write methods — that violates LSP.

---

### I — Interface Segregation Principle (ISP)

Clients should not be forced to depend on interfaces they don't use. Prefer many small interfaces over one fat one.

```csharp
// ❌ Wrong — every implementer must implement everything even if irrelevant
public interface IOrderService
{
    Task PlaceOrder(Order order);
    Task CancelOrder(OrderId id);
    Task GetOrderHistory(CustomerId id);
    Task GenerateInvoice(OrderId id);
    Task ExportToCsv(DateRange range);
}

// ✅ Correct — split by consumer needs
public interface IOrderPlacer { Task PlaceOrder(Order order); }
public interface IOrderCanceller { Task CancelOrder(OrderId id); }
public interface IOrderHistoryReader { Task<IEnumerable<Order>> GetHistory(CustomerId id); }
public interface IOrderExporter { Task ExportToCsv(DateRange range); }

// Checkout only depends on what it needs
public class CheckoutService
{
    private readonly IOrderPlacer _placer;   // just this
    public CheckoutService(IOrderPlacer placer) => _placer = placer;
}
```

---

### D — Dependency Inversion Principle (DIP)

High-level modules should not depend on low-level modules. Both should depend on abstractions.

```csharp
// ❌ Wrong — business logic depends directly on infrastructure
public class OrderService
{
    private readonly SqlOrderRepository _repo;  // concrete dependency
    private readonly SmtpEmailSender _email;    // concrete dependency

    public OrderService()
    {
        _repo = new SqlOrderRepository();       // hard-coded creation
        _email = new SmtpEmailSender();
    }
}

// ✅ Correct — both depend on abstractions (interfaces)
public class OrderService
{
    private readonly IOrderRepository _repo;    // interface
    private readonly IEmailSender _email;       // interface

    public OrderService(IOrderRepository repo, IEmailSender email)
    {
        _repo = repo;
        _email = email;
    }
}

// Wired up in Program.cs / Startup.cs
builder.Services.AddScoped<IOrderRepository, SqlOrderRepository>();
builder.Services.AddScoped<IEmailSender, SendGridEmailSender>();

// In tests — swap the real implementation for a fake
var sut = new OrderService(new InMemoryOrderRepository(), new FakeEmailSender());
```

---

### SOLID — The staff-level summary

> "SOLID principles aren't rules to follow mechanically — they're heuristics that push code toward being easier to change, test, and understand. SRP and DIP are the most impactful day-to-day. Taken too far, they can lead to over-engineering: dozens of tiny interfaces, an explosion of classes, and a codebase where the simplest feature requires touching ten files. The goal is enough structure to manage complexity — not structure for its own sake."

---
---

# DAY 3: Clean Architecture & Hexagonal Architecture

---

## What problem are we solving?

In a monolith or naive microservice, business logic leaks everywhere:
- Controllers call `_dbContext.Orders.ToList()` directly
- Services are hard-coded to `SmtpClient` or a specific HTTP client
- Unit tests require a real database

The result: **you can't change the database without rewriting business logic, and you can't test business logic without spinning up infrastructure.**

Clean and Hexagonal Architecture solve this by creating a strict rule: **business logic has zero knowledge of and zero dependency on infrastructure.**

---

## Clean Architecture

Popularised by Robert C. Martin ("Uncle Bob"). Organises code into concentric layers:

```
          ┌─────────────────────────────────────┐
          │           FRAMEWORKS & DRIVERS       │  ← Web, DB, External APIs
          │   ┌─────────────────────────────┐   │
          │   │        INTERFACE ADAPTERS    │   │  ← Controllers, Repositories (impl)
          │   │   ┌─────────────────────┐   │   │
          │   │   │   APPLICATION CORE   │   │   │  ← Use Cases / Command Handlers
          │   │   │  ┌───────────────┐  │   │   │
          │   │   │  │    DOMAIN     │  │   │   │  ← Entities, Value Objects, Domain Services
          │   │   │  └───────────────┘  │   │   │
          │   │   └─────────────────────┘   │   │
          │   └─────────────────────────────┘   │
          └─────────────────────────────────────┘

Dependency Rule: arrows point INWARD only.
Domain knows nothing about Application.
Application knows nothing about Infrastructure.
```

### The Dependency Rule — the single most important concept

> Inner layers never reference outer layers. Ever. The `Domain` project has zero NuGet references to Entity Framework, Azure Service Bus, or any HTTP client. It just defines interfaces (ports) and domain logic.

---

## Hexagonal Architecture (Ports & Adapters)

Coined by Alistair Cockburn. Equivalent concept to Clean Architecture, different framing.

```
                    ┌──────────────────────┐
  HTTP Request ────►│     PRIMARY ADAPTER   │
  gRPC Call ───────►│  (ASP.NET Controller) │
  Message Bus ─────►│  (Service Bus Consumer│
                    └──────────┬───────────┘
                               │ uses
                    ┌──────────▼───────────┐
                    │                      │
                    │      APPLICATION     │
  ┌─────────────┐  │         CORE         │  ┌─────────────┐
  │  PRIMARY    │  │                      │  │  SECONDARY  │
  │   PORTS     │◄─┤  Business Logic      ├─►│   PORTS     │
  │(Driving)    │  │  Domain Rules        │  │ (Driven)    │
  └─────────────┘  │  Use Cases           │  └──────┬──────┘
                    │                      │         │ implemented by
                    └──────────────────────┘  ┌──────▼──────┐
                                              │  SECONDARY  │
                                              │  ADAPTERS   │
                                              │  (EF Core   │
                                              │   SendGrid  │
                                              │   Redis)    │
                                              └─────────────┘
```

- **Primary (Driving) Ports** — How the outside world calls your app (controllers, consumers, CLI handlers)
- **Secondary (Driven) Ports** — How your app calls the outside world (repository interfaces, email sender interfaces)
- **Adapters** — The concrete implementations of ports (EF Core repo, SMTP sender)

---

## Project Structure in C#

```
WooliesX.Order/
├── Domain/                        ← Zero dependencies
│   ├── Entities/
│   │   ├── Order.cs
│   │   └── OrderLine.cs
│   ├── ValueObjects/
│   │   ├── Money.cs
│   │   └── OrderId.cs
│   ├── Events/
│   │   └── OrderPlacedEvent.cs
│   ├── Exceptions/
│   │   └── DomainException.cs
│   └── Interfaces/                ← Ports (the contracts, not implementations)
│       ├── IOrderRepository.cs
│       └── IPaymentGateway.cs
│
├── Application/                   ← Depends on Domain only
│   ├── Commands/
│   │   ├── PlaceOrderCommand.cs
│   │   └── PlaceOrderCommandHandler.cs
│   ├── Queries/
│   │   ├── GetOrderByIdQuery.cs
│   │   └── GetOrderByIdQueryHandler.cs
│   └── DTOs/
│       └── OrderSummaryDto.cs
│
├── Infrastructure/                ← Depends on Application + Domain
│   ├── Persistence/
│   │   ├── AppDbContext.cs
│   │   ├── OrderRepository.cs     ← Implements IOrderRepository
│   │   └── Configurations/
│   │       └── OrderConfiguration.cs
│   ├── Messaging/
│   │   └── ServiceBusPublisher.cs
│   └── ExternalServices/
│       └── StripePaymentGateway.cs  ← Implements IPaymentGateway
│
└── API/                           ← Depends on Application + Infrastructure
    ├── Controllers/
    │   └── OrdersController.cs
    ├── Middleware/
    │   └── ExceptionHandlingMiddleware.cs
    └── Program.cs
```

---

## Worked Example: PlaceOrder

### Domain layer

```csharp
// Domain/Entities/Order.cs — knows nothing about EF Core or HTTP
public class Order
{
    public OrderId Id { get; private set; }
    public CustomerId CustomerId { get; private set; }
    public OrderStatus Status { get; private set; } = OrderStatus.Draft;
    private List<OrderLine> _lines = new();
    private List<IDomainEvent> _domainEvents = new();

    public IReadOnlyList<OrderLine> Lines => _lines.AsReadOnly();
    public IReadOnlyList<IDomainEvent> DomainEvents => _domainEvents.AsReadOnly();

    public void AddItem(ProductId productId, int quantity, Money unitPrice)
    {
        if (Status != OrderStatus.Draft)
            throw new DomainException("Can only add items to a draft order.");

        _lines.Add(new OrderLine(productId, quantity, unitPrice));
    }

    public void Place()
    {
        if (!_lines.Any())
            throw new DomainException("Cannot place an empty order.");

        Status = OrderStatus.Placed;
        _domainEvents.Add(new OrderPlacedEvent(Id, CustomerId, _lines, DateTime.UtcNow));
    }
}

// Domain/Interfaces/IOrderRepository.cs — a port
public interface IOrderRepository
{
    Task<Order> GetByIdAsync(OrderId id, CancellationToken ct = default);
    Task AddAsync(Order order, CancellationToken ct = default);
}
```

---

### Application layer

```csharp
// Application/Commands/PlaceOrderCommand.cs
public record PlaceOrderCommand(
    Guid CustomerId,
    List<OrderLineDto> Lines);

// Application/Commands/PlaceOrderCommandHandler.cs
// Depends only on domain interfaces — no EF Core, no HTTP
public class PlaceOrderCommandHandler
{
    private readonly IOrderRepository _orders;
    private readonly IPaymentGateway _payments;
    private readonly IEventPublisher _events;

    public PlaceOrderCommandHandler(
        IOrderRepository orders,
        IPaymentGateway payments,
        IEventPublisher events)
    {
        _orders = orders;
        _payments = payments;
        _events = events;
    }

    public async Task<OrderId> Handle(
        PlaceOrderCommand cmd,
        CancellationToken ct = default)
    {
        var order = new Order(new CustomerId(cmd.CustomerId));

        foreach (var line in cmd.Lines)
            order.AddItem(
                new ProductId(line.ProductId),
                line.Quantity,
                new Money(line.UnitPrice, "AUD"));

        order.Place(); // enforces domain rules, raises domain event

        await _orders.AddAsync(order, ct);

        // Dispatch domain events
        foreach (var @event in order.DomainEvents)
            await _events.PublishAsync(@event, ct);

        return order.Id;
    }
}
```

---

### Infrastructure layer

```csharp
// Infrastructure/Persistence/OrderRepository.cs
// This is where EF Core lives — not in Domain or Application
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _db;

    public OrderRepository(AppDbContext db) => _db = db;

    public async Task<Order> GetByIdAsync(OrderId id, CancellationToken ct = default)
        => await _db.Orders
            .Include(o => o.Lines)
            .AsNoTracking() // on read-side CQRS; omit on write-side
            .FirstOrDefaultAsync(o => o.Id == id, ct)
            ?? throw new OrderNotFoundException(id);

    public async Task AddAsync(Order order, CancellationToken ct = default)
        => await _db.Orders.AddAsync(order, ct);
}

// Infrastructure/Persistence/AppDbContext.cs
public class AppDbContext : DbContext
{
    public DbSet<Order> Orders => Set<Order>();

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Apply all IEntityTypeConfiguration in this assembly
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
    }
}

// Infrastructure/Persistence/Configurations/OrderConfiguration.cs
// EF Core mapping kept separate from the domain entity
public class OrderConfiguration : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        builder.HasKey(o => o.Id);
        builder.Property(o => o.Id)
            .HasConversion(id => id.Value, value => new OrderId(value));

        builder.OwnsMany(o => o.Lines, lines =>
        {
            lines.Property(l => l.UnitPrice)
                .HasConversion(
                    m => m.Amount,
                    amount => new Money(amount, "AUD"));
        });
    }
}
```

---

### API layer

```csharp
// API/Controllers/OrdersController.cs
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IMediator _mediator; // MediatR dispatches to the handler

    public OrdersController(IMediator mediator) => _mediator = mediator;

    [HttpPost]
    public async Task<IActionResult> PlaceOrder(
        PlaceOrderRequest request,
        CancellationToken ct)
    {
        var command = new PlaceOrderCommand(request.CustomerId, request.Lines);
        var orderId = await _mediator.Send(command, ct);
        return CreatedAtAction(nameof(GetOrder), new { id = orderId }, null);
    }
}
```

---

### Wiring it all together — Program.cs

```csharp
// API/Program.cs
var builder = WebApplication.CreateBuilder(args);

// Infrastructure — concrete implementations registered to abstractions
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("OrderDb")));

builder.Services.AddScoped<IOrderRepository, OrderRepository>();
builder.Services.AddScoped<IPaymentGateway, StripePaymentGateway>();
builder.Services.AddScoped<IEventPublisher, ServiceBusEventPublisher>();

// Application — MediatR finds all handlers
builder.Services.AddMediatR(typeof(PlaceOrderCommandHandler).Assembly);

var app = builder.Build();
app.MapControllers();
app.Run();
```

---

## How to keep EF Core out of your Domain (The key interview demo)

The most common trap is letting EF Core concepts leak into domain entities. Here's how to avoid it:

| Concern | Wrong (EF Core in Domain) | Right (EF Core in Infrastructure) |
|---|---|---|
| Primary key | `[Key] public int Id` attribute on entity | `builder.HasKey(o => o.Id)` in `IEntityTypeConfiguration` |
| Navigation props | `public virtual ICollection<OrderLine> Lines` | `builder.OwnsMany(...)` in config |
| Value Object mapping | `[ComplexType] public Money Price` | `.HasConversion(...)` in config |
| Concurrency | `[Timestamp] byte[] RowVersion` on entity | `builder.IsRowVersion()` in config |

The **`IEntityTypeConfiguration<T>`** pattern is your answer to "how do you keep EF Core out of the domain?" — all mapping lives in the Infrastructure project, the domain entity is a plain C# class.

---

## Hexagonal Architecture — The interview drawing

When asked to draw it, keep it simple:

```
                  ┌──────────────────────────────────┐
                  │           APPLICATION CORE         │
  ┌──────────┐   │                                    │   ┌──────────────┐
  │ HTTP API │──►│  PlaceOrderCommandHandler           │──►│ SQL Server   │
  └──────────┘   │  (uses IOrderRepository)           │   │ (EF Core)    │
                  │  (uses IPaymentGateway)            │   └──────────────┘
  ┌──────────┐   │  (uses IEventPublisher)            │   ┌──────────────┐
  │ Svc Bus  │──►│                                    │──►│ Stripe API   │
  │ Consumer │   │  Domain: Order, OrderLine, Money   │   └──────────────┘
  └──────────┘   │                                    │   ┌──────────────┐
                  │                                    │──►│ Azure Svc Bus│
                  └──────────────────────────────────┘   └──────────────┘

  PRIMARY ADAPTERS (left)          SECONDARY ADAPTERS (right)
  [Drive the application]          [Are driven by the application]
```

---

## Tradeoffs — The staff-level answer

> "Clean and Hexagonal Architecture solve a real problem: they make business logic independently testable and make infrastructure swappable. The cost is more boilerplate — you get more projects, more interfaces, and more mapping between layers. For a simple CRUD microservice (like a feature flag store), this overhead isn't worth it. But for a domain with complex rules that changes frequently — like the Woolies cart or a pricing engine — the payoff in testability and maintainability is significant. The heuristic I use: if the domain layer warrants its own unit test suite, the architecture is worth the overhead."

---

## Senior Gotchas to mention

- **Don't put business rules in the Application layer.** A handler that checks `if (order.Lines.Count == 0) throw new Exception(...)` is a smell. That rule belongs in the `Order.Place()` method on the domain entity.
- **Application layer is for orchestration, not decisions.** It coordinates repositories, publishes events, and calls domain methods — but it doesn't make domain decisions itself.
- **CQRS fits naturally here.** Read queries (returning DTOs) don't need to go through the domain at all — they can query the DB directly from the handler and map to a DTO. Only writes go through the aggregate.
- **EF Core's `DbContext` should never appear in Domain or Application projects.** If you see an `ApplicationDbContext` import in a command handler, something's wrong.

---

## Quick Reference Cheat Sheet

| Concept | One-liner | WooliesX example |
|---|---|---|
| Bounded Context | Explicit boundary where a model applies | Cart, Catalogue, Fulfilment, Loyalty |
| Ubiquitous Language | Same words in conversation and code | `PlaceOrder()` not `CreateOrderRecord()` |
| Entity | Identity-based object | `Order`, `Customer` |
| Value Object | Value-based, immutable | `Money`, `Address`, `ProductSku` |
| Aggregate | Cluster with one root entry point | `Order` (root) + `OrderLine` (member) |
| Domain Event | Something significant that happened | `OrderPlacedEvent` |
| Repository | Persistence abstraction | `IOrderRepository` |
| Domain Service | Logic spanning multiple aggregates | `PricingService` |
| Port | Interface defined by the domain | `IOrderRepository`, `IPaymentGateway` |
| Adapter | Concrete implementation of a port | `OrderRepository (EF Core)`, `StripePaymentGateway` |
| Primary Adapter | Drives the application | ASP.NET Controller, Service Bus Consumer |
| Secondary Adapter | Driven by the application | EF Core, SendGrid, Redis |

Missing
DI over new
Http Clients - life time, singleton/transient/scoped
auth
exception handling

eventhub

ebay auction
- how to find winner

