# Senior / Staff System Design Interview — 3-Week Prep Plan
> WooliesX · C# focused · 2–3 hrs/day

---

## Staff-Level "Missing Pieces" (Know These Before Week 1)

- **Idempotency** — Essential for distributed systems and event-driven architectures. Prevents double-charging or duplicate orders in a Woolies checkout context.
- **Consistency Models** — Sagas (Orchestration vs. Choreography) for distributed transactions.
- **Outbox Pattern** — How to reliably publish events after a database update. Implement with `IHostedService`.
- **Caching Strategies** — Beyond just "using Redis": Cache Aside, Write-Through, and handling Thundering Herds (your killer Woolies Special Buys example).
- **API Management & Security** — Rate limiting, YARP (Yet Another Reverse Proxy), and OAuth2/OpenID Connect flows.

---

## Week 1 — Domain Modelling & Internal Architecture
> Focus: Master the "inside" of a microservice

### Day 1–2 · DDD & SOLID in C#
Don't just define them — practice decomposing a monolith into Woolies subdomains (Cart, Inventory, Fulfilment). Explain how bounded contexts map to microservices. CAP reflex: apply it to every design decision you make this week.

### Day 3 · Clean / Hexagonal Architecture
Map how business logic (Core) stays independent of EF Core and external APIs. Draw the Ports & Adapters diagram from memory.

### Day 4 · CQRS & Command Handlers
Why separate reads from writes? Critically: articulate when CQRS is **overkill**. Staff roles identify when *not* to use it.

### Day 5 · Dependency Injection & Lifetimes
Deep dive into Transient, Scoped, and Singleton. Understand Captive Dependencies and why `IHttpClientFactory` solves socket exhaustion.

### Day 6 · Event Sourcing — Concepts & Tradeoffs *(half day)*
Focus on the Audit Trail benefit and Snapshot complexity. Frame it as a tradeoff discussion, not an implementation exercise.

### Day 7 · Outbox Pattern + Idempotency ⭐ *new*
How to reliably publish events after a DB update. Pair with idempotency keys to prevent double-charging. Implement the Outbox processor using `IHostedService`.

---

## Week 2 — Distributed Systems & Infrastructure
> Focus: How services talk, scale, and survive failures

### Day 8–9 · Event-Driven Systems + Messaging
Azure Service Bus vs Kafka. Competing Consumers vs Fan-out. Poison pills and dead-letter queues. **Azure-specific depth**: Service Bus Sessions for ordered delivery.

### Day 10 · SQL vs NoSQL
ACID for transactions (SQL) vs high-scale catalog data (CosmosDB). Deep dive: Cosmos DB partition key choice is **irreversible** — explain why. Cover sharding strategies.

### Day 11 · Resiliency Patterns with Polly
Retry, Circuit Breaker, Bulkhead. Explain the difference precisely — interviewers will probe here. When does each pattern apply in a grocery order flow?

### Day 12 · Real-Time Communication
SignalR vs WebSockets vs Webhooks. Which for Live Order Tracking? Answer: SignalR with Azure SignalR Service — explain the fallback transport (long-polling).

### Day 13 · Redis & Caching Strategies
Cache Aside, Write-Through. RedLock for distributed locking. Thundering Herd on Special Buys events — this is your killer Woolies-specific scenario.

### Day 14 · Telemetry & Observability
Structured logging with Serilog. OpenTelemetry (traces, metrics, logs). Why distributed tracing is the *only* way to debug a microservice mesh.

---

## Week 3 — Staff Mindset, Tradeoffs & Mock Interviews
> Focus: Synthesise, communicate, and own the room

### Day 15–16 · High-Level Scaling & API Management
Load balancers, CDNs, horizontal vs vertical scaling. Add YARP, rate limiting, and OAuth2/OpenID Connect flows. Practice drawing the Big Picture diagram end-to-end.

### Day 17 · CAP Theorem — Synthesis Drill *(1 hour only)*
By now CAP should be a **reflex**, not a chapter. Rapid-fire drill: for every design decision, say "We gain X (availability) but sacrifice Y (consistency)." One hour only — do not over-invest here.

### Day 18 · Requirements Gathering Drill ⭐ *new*
Staff interviews start vague on purpose. Drill the first 5 minutes of an interview:
- Clarify scope
- Define SLOs
- Ask about read/write ratios, DAU, peak load

This is a standalone skill — practice it explicitly and time yourself.

### Day 19 · Mock 1 — Design a Grocery Order Tracking System
45 minutes timed. Structure:
1. Requirements gathering (5 min)
2. Draw architecture (20 min)
3. Walk through tradeoffs (20 min)

Record yourself. Listen for missing "whys".

### Day 20 · Mock 2 — Design Woolies Special Buys Sale Infrastructure ⭐ *new*
Focus on cache stampede, inventory reservation, and event-driven order placement. Apply **Thundering Herd + Outbox + Idempotency** together in one answer.

### Day 21 · Mock 3 — Design a Real-Time Loyalty Points System
Full synthesis: CQRS reads, event sourcing for audit trail, SignalR for live balance updates, Polly for resilience. Review all recordings. Fix the "umms" — your **why must beat your what**.

---

## C# Senior Gotchas — Know These Cold

| Topic | The "Staff" Insight |
|---|---|
| `IHttpClientFactory` | Never use `new HttpClient()`. Manages `HttpClientHandler` lifetime to prevent DNS staleness and socket exhaustion. |
| `Async` / `Await` | Use `Task.WhenAll` for parallelising external calls. Mention `ConfigureAwait(false)` for library code — shows depth. |
| `Middleware` pipeline | Cross-cutting concerns (global error handling, telemetry, auth) all belong here. |
| EF Core performance | `AsNoTracking()` on read-side CQRS. Always watch for N+1 queries in navigation properties. |
| `CancellationToken` | Propagate through the **entire** call chain — DB, HTTP, message bus. Shows production awareness. |
| `IHostedService` / `BackgroundService` | Natural fit for Outbox processors and event consumers in .NET. Expect this if you pitch the Outbox Pattern. |

---

## Key Themes to Thread Through Every Week

- **CAP as a reflex** — Don't save tradeoff analysis for Week 3. On every design decision say: *"We gain X, but we sacrifice Y."*
- **WooliesX context** — Anchor every answer to a real Woolies domain. Cart, Inventory, Fulfilment, Loyalty, Special Buys. Interviewers at product companies love hearing domain-specific reasoning.
- **When NOT to use X** — Staff-level thinking is about restraint. Knowing CQRS is overkill for a simple CRUD service shows more than knowing how to implement it.
- **Why > What** — Any candidate can describe a pattern. You need to explain *why* it's the right choice for *this* problem at *this* scale.

