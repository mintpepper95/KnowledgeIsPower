## 3-Week Preparation Schedule (2–3 Hours/Day)

### **Week 1: Domain Modeling & Internal Architecture**

_Focus: Mastering the "Inside" of a microservice._

- **Day 1-2: DDD & SOLID in C#.** Don't just define them; practice explaining how DDD helps decompose a monolith into Woolies-sized subdomains (e.g., Cart vs. Inventory).
    
- **Day 3: Clean/Hexagonal Architecture.** Mapping out how to keep the business logic (Core) independent of the database (EF Core) or external APIs.
    
- **Day 4: CQRS & Command Handlers.** Why separate reads from writes? When is it overkill? (Hint: Staff roles often identify when _not_ to use CQRS).
    
- **Day 5: Dependency Injection & Lifetimes.** Deep dive into `Transient`, `Scoped`, and `Singleton`. Specifically, understand **Captive Dependencies** and why `IHttpClientFactory` exists to solve socket exhaustion.
    
- **Day 6-7: Event Sourcing.** Focus on the "Audit Trail" benefit and the complexity of "Snapshots."
    

---

### **Week 2: Distributed Systems & Infrastructure**

_Focus: How services talk and scale._

- **Day 8-9: Event-Driven Systems.** Azure Service Bus or Kafka. Compare **Competing Consumers** vs. **Fan-out**. Discuss handling "Poison Pills" and dead-letter queues.
    
- **Day 10: SQL vs. NoSQL.** When to use SQL (ACID for transactions) vs. NoSQL (CosmosDB/Mongo for high-scale catalog data). Discuss **Sharding** and **Partition Keys**.
    
- **Day 11: Resiliency Patterns.** Using **Polly** in C#. Explain the difference between a Retry, a Circuit Breaker, and a Bulkhead.
    
- **Day 12: Real-time Communication.** SignalR vs. WebSockets vs. Webhooks. Which one is best for a "Live Order Tracking" feature?
    
- **Day 13: Redis & Caching.** Distributed locking with RedLock and caching strategies for high-traffic "Special Buys" events.
    
- **Day 14: Telemetry & Logging.** Structured logging (Serilog), OpenTelemetry, and why "Distributed Tracing" is the only way to debug a microservice web.
    

---

### **Week 3: The "Staff" Mindset & Mocking**

_Focus: Synthesis, Trade-offs, and Communication._

- **Day 15-16: High-Level Scaling.** Load balancers, CDNs, and Horizontal vs. Vertical scaling. Practice drawing the "Big Picture."
    
- **Day 17-18: Trade-off Analysis (CAP Theorem).** For every design choice, practice saying: _"We gain X (Availability), but we sacrifice Y (Strong Consistency)."_
    
- **Day 19-21: Mock Interviews.** Take a standard prompt (e.g., "Design a grocery delivery tracking system") and spend 45 minutes drawing and explaining it.
    
    - _Self-Mock Tip:_ Record yourself. Listen for "umms" and ensure your "Why" is stronger than your "What."
        

* Misc
	Outbox pattern, Idempotent consumer, 
---

## Deep Dive: C# Specifics to Keep in Mind

Since the interview is C#-focused, expect "Senior" gotchas:

|**Topic**|**The "Senior" Insight**|
|---|---|
|**IHttpClientFactory**|Don't just use `new HttpClient()`. Mention it manages the underlying `HttpClientHandler` lifetime to prevent DNS issues and socket exhaustion.|
|**Async/Await**|Understand `Task.WhenAll` for parallelizing external calls and how `ConfigureAwait(false)` impacts context (though less critical in .NET Core+, it shows depth).|
|**Middleware**|Know how to use the Middleware pipeline for cross-cutting concerns like global error handling or telemetry.|
|**EF Core Performance**|Mention `AsNoTracking()` for read-heavy CQRS sides and the dangers of N+1 queries.|
