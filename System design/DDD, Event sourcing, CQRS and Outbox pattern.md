
[[#What is Event Sourcing?]]

[[#What is an Aggregate root?]]

[[#What is CQRS]]


---
### What is Event Sourcing?
You store a sequence of **immutable** events that described what happened instead of just the current state. Current state is always derived by roleplaying these events.

1. Events are immutable and append only. You never update or delete events.
2. An event store - a specialised db for appending, replaying and querying events
3. Replay -  to get current state, you start from beginning or a snapshot and apply the events in order

#### When to use Event Sourcing?

**Use it when:**

- You need a **full audit trail** — finance, healthcare, legal, compliance
- **Temporal queries** matter — "what was the state last Tuesday?"
- You need to **replay history** to fix bugs (reprocess old events with new logic)
- The **business domain is event-driven** by nature (orders, payments, bookings)
- You need **multiple read models** from the same data

**Don't use it when:**

- The domain is simple CRUD with no audit/history needs
- The team isn't familiar with it — the operational overhead is real
- You need strong, simple consistency guarantees with minimal complexity


#### What is projection?

Projection - process of replaying events to build a read model, a particular view of the data optimised for querying.

You can have multiple projections for the same event, each for different use cases.

You can add projections later and calculate it using replay.

```
Same events →  Projection A: current account balance (for UI)
            →  Projection B: monthly transaction summary (for reports)
            →  Projection C: fraud detection feed (for alerts)
```


#### ES projection
Projection - mapping a DB row to a DTO _is_ a trivial form of projection. There are two different things people call "projection":
```cs
// DB has: { id, status, seatCount, createdAt }
// You map it to a DTO — you're just reshaping existing stored state
var dto = new BookingSummaryDto { Id = row.Id, Status = row.Status };
```


ES projection
```cs
// You process a stream of raw events to BUILD a read model that
// doesn't exist anywhere yet
foreach (var event in allEvents)
{
    if (event is BookingConfirmed e)
        readModel.ConfirmedCount++;

    if (event is BookingCancelled e)
        readModel.CancellationReasons.Add(e.Reason);
}
```

**You can build read models that cross aggregate boundaries.** Say you want "total revenue per venue this month." In a traditional DB you'd join across tables. In ES, you replay `BookingConfirmed` events from _all_ bookings and aggregate them into a `VenueRevenueSummary` read model. The read model is a _materialised_ computation over raw events.

**You can rebuild any projection from scratch, at any time.** This is the real power. If you add a new business requirement — say, "track how many bookings were made within 10 minutes of an event starting" — in a traditional system you have no historical data. In ES, you **replay all past events** through your new projection logic and instantly have historical data too. You can't do this with a current-state DB because the history is gone.

#### What is event schema evolution and why is it a problem?

**Event schema evolution** is the sneaky hard problem — once you have events in production, you can never change their shape without a migration strategy (upcasting, versioned events, etc.)

We can periodically save the computed state, and only replay events _after_ that snapshot.


**Where ES does have a specific consistency concern** — and this is the one worth knowing for a senior interview — is **cross-aggregate uniqueness**:

#### Cross-Aggregate Consistency
In DDD, each aggregate is a **consistency boundary**. This means:

> _Within one aggregate, all your business rules are enforced atomically. You load it, change it, save it — all or nothing.

```cs
// This is safe — everything happens inside ONE aggregate 
var booking = repository.Get(bookingId); 
booking.Confirm(); // validates internally 
repository.Save(booking); // saves atomically
```
No other aggregate is involved. No race conditions. Strongly consistent.

#### Why ES makes it harder

In Event Sourcing, each aggregate is an independent event stream. There is no shared table with a unique index. The streams look like this:

```
stream: user-abc →  [UserRegistered { email: "alice@gmail.com" }]
stream: user-xyz →  [UserRegistered { email: "alice@gmail.com" }]  ← ES doesn't prevent this
```

The event store just appends to independent streams. It has no concept of _"check all other streams before appending."

Race condition
```cs
Time 0ms:  Request A checks — is "alice@gmail.com" taken? → No
Time 0ms:  Request B checks — is "alice@gmail.com" taken? → No
Time 1ms:  Request A appends UserRegistered { email: "alice@gmail.com" } ✅
Time 1ms:  Request B appends UserRegistered { email: "alice@gmail.com" } ✅ ← duplicate
```


##### Solving it

**Option 1 — Use a separate unique index store** Maintain a dedicated lookup table (e.g. in Redis or a regular DB table) that maps email → userId, with a unique constraint. Check and reserve the email _before_ appending the event. This is outside the event store but gives you the atomic guarantee.

**Option 2 — Make email itself the aggregate ID** If the email _is_ the unique identifier, you can use it as the stream key. The event store can enforce uniqueness at the stream level. This only works for simple cases.

**Option 3 — Accept eventual consistency with compensation** Allow the duplicate, detect it in a projection, and send the second user a "this email is already registered" email. This is the _saga_ pattern — you compensate after the fact rather than preventing upfront. Uncommon for email uniqueness but used in financial systems.


Each aggregate in ES is its own isolated stream. Enforcing invariants _within_ one aggregate is strongly consistent (you can use optimistic concurrency with version numbers). But enforcing invariants _across_ aggregates is genuinely hard, and that's an ES-specific problem, not a CQRS one.


- **How do you handle scale?"** — Snapshots. After N events, persist the current state as a snapshot. Replay only from the snapshot forward.
- **"What if the projection falls behind?"** — Eventual consistency on the read side. Acceptable for reporting views; flag if it's not acceptable for the use case.
- **"What about cross-aggregate consistency?"** — E.g. enforcing that a seat can't be double-booked. ES alone doesn't solve this — you need optimistic concurrency on the aggregate stream version, or a separate seat reservation step with its own consistency guarantee.
- **"What if event schema changes?"** — Upcasting: when reading old events, a migration function transforms them to the new shape before the aggregate sees them.

---

### What is DDD and Aggregate root?

Aggregate root from DDD, a pattern for breaking down system into microservices. It's a cluster of domain objects treated as a single unit. It is the entry point outside world talks to. 

An aggregate correspond to a bounded context which usually maps to a microservice. 

```
HTTP Request
     ↓
[API Controller]        ← Infrastructure / Presentation layer
     ↓
[Application Service]   ← Orchestration (handles commands/queries)
     ↓
[Aggregate Root]        ← Domain layer (your business rules live here)
     ↓
[Repository]            ← Persistence layer (saves to DB / event store)
```

Note DDD and Event Source are not related, both can work without each other.
CQRS can also work without either.


**DDD without ES**:
The only thing ES _adds_ to DDD is replacing "save current state" with "append events." The aggregate root concept, validation, and command handling all exist in plain DDD regardless.
```csharp
booking.Confirm();
await _repository.Save(booking); // just saves current state to db
```

**DDD with ES:**
```csharp
booking.Confirm(); // internally records a BookingConfirmed event
var events = booking.UncommittedEvents;
await _eventStore.Append(events); // saves events instead of state
```


|Concept|What it is|Without the other?|
|---|---|---|
|**DDD**|A way to _model_ your domain with Aggregates, Entities, Value Objects|✅ Works without ES|
|**Event Sourcing**|A _persistence strategy_ — store events instead of current state|✅ Works without DDD|
|**CQRS**|Separate read and write paths|✅ Works without either|

---
### Eventual Consistency in DDD

In Domain-Driven Design (DDD), maintaining invariants across domain boundaries is a classic challenge. The core principle is that **invariants must be enforced within a transaction boundary**, and an Aggregate is that boundary.

When a business process requires consistency across multiple Aggregates (or Bounded Contexts), you must move from _immediate consistency_ to _eventual consistency_.

### 1. Handling Cross-Aggregate Logic

If two entities must always be consistent, they often belong in the same Aggregate. If they are logically distinct enough to be separate Aggregates, you must accept that they cannot be updated in a single ACID transaction.

#### The "Eventual Consistency" Pattern

Use **Domain Events** to propagate state changes. When Aggregate A changes, it publishes a domain event. A handler listens for that event and triggers the necessary update in Aggregate B.

- **Process:**
    
    1. **Command:** Update Aggregate A.
        
    2. **Transaction:** Persist Aggregate A and the Domain Event in the same local database transaction (using the **Outbox Pattern** to ensure the event is actually sent).
        
    3. **Reaction:** An event handler consumes the event and issues a command to Aggregate B.
        

### 2. Ensuring Invariants (Techniques)

| **Technique**               | **When to Use**                                                                                                                                      |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Aggregate Consolidation** | If the rule is a strict, non-negotiable invariant that must be consistent _instantly_, the Aggregates are likely one aggregate. Merge them.          |
| **Domain Events**           | For business rules that can tolerate a slight delay in consistency. This is the most common DDD approach.                                            |
| **Saga / Process Manager**  | Use when a business process spans multiple steps, multiple aggregates, or requires compensating actions (e.g., "If step B fails, roll back step A"). |
| **Optimistic Concurrency**  | Use versioning on your aggregates to prevent "lost updates" when multiple processes are reacting to state changes.                                   |



### Where the "Problem" Actually Lies

Eventual consistency becomes a problem only when the **Business Language** doesn't account for it.

- **The Technical Reality:** When you use a message queue (like Azure Service Bus), there is a delay (latency) between Service A finishing and Service B starting. During that window, the system is "inconsistent."
    
- **The DDD Solution:** Instead of trying to "fix" the latency with technical hacks (like distributed locks), DDD suggests you **model the latency into the business process.**
    

> **Example:** In an e-book store, if a user buys a book, "Immediate Consistency" would mean the user waits on the checkout page until the payment is processed AND the book is added to their library. "Eventual Consistency" means the user gets a "Thank you, your book is being prepared" message. The business has accepted a state of "Pending."

### How to Maintain Invariants with Buffers

When using message queues, you ensure invariants through these specific DDD/Microservice patterns:

1. **Idempotency:** Service B must be able to receive the same message twice without causing side effects (e.g., charging a customer twice).
    
2. **Outbox Pattern:** To ensure you don't save to your DB but fail to send the message, you save the message _to the same DB_ in one transaction, then a separate process sends it.
    
3. **Compensating Transactions (Sagas):** If Service A completes but Service B fails (e.g., out of stock), you must trigger an "Undo" event back to Service A to refund the user.

If you find yourself fighting eventual consistency (e.g., "I absolutely need this data to be updated in both places at the exact same millisecond"), you likely have a **Leaky Abstraction**. You should probably merge those two services into a single Bounded Context.

---
### What is CQRS

Basically it splits your system into read and write.

For example you can read and write into different databases.


#### Problem of Eventual Consistency with CQRS
**Eventual consistency comes from CQRS, not ES.**

This lag is a CQRS problem. You'd have the same eventual consistency issue with CQRS using a normal database. ES has nothing to do with it.

```
Command → Write Side → Event Store
                            ↓ (async, small lag)
                       Projection updates Read Model
                            ↓
Query ← Read Model  ← (might be slightly stale)
```



#### CQRS vs direct SQL

Start with direct SQL for both reads and writes when:
- The app is straightforward CRUD
- Read and write shapes are similar
- Team is small, iteration speed matters
- Consistency between what you write and what you read matters for UX


#### Introduce CQRS when a specific pain point emerges:

**Pain point 1 — Read and write models are very different shapes**

Your write model is normalised (good for integrity), but your read model needs a heavily joined, denormalised view (good for performance). Maintaining that as a single model becomes painful.

```
Write model: Order + OrderLines + Customer + Product (normalised)
Read model:  OrderSummary { orderId, customerName, totalItems, totalPrice }
             → This is a JOIN across 4 tables, expensive to recompute on every request
                    
```

Your system gets 1000 reads per second but only 5 writes per second. With a shared model you're scaling your write infrastructure unnecessarily. CQRS lets you scale read and write sides independently.

ES naturally produces events. CQRS is the obvious way to consume them — the read side just subscribes and builds projections. They pair naturally.

For CQRS, there is a buffer

The buffer is typically a message queue (Kafka, RabbitMQ, Azure Service Bus). The write side publishes an event, the read side consumes it asynchronously. That async gap is exactly where eventual consistency lives.

**How much lag are we actually talking about?**

Usually milliseconds to low seconds under normal load. But under failure conditions — queue backlog, handler crash, reprocessing — it can be longer. This is why you need to design your UX around it.


some cases where it's fine:

- Reporting dashboards — slightly stale data is acceptable
- Admin views — not time-critical
- Analytics — eventual is fine


user confirms a booking → gets redirected to "My Bookings" page → their new booking isn't there yet (read model hasn't updated) → user thinks it failed and books again

**Option 1 — Read your own write.** After a command, read directly from the write store for _that specific user's immediate next request_, bypassing the read model.

**Option 2 — Optimistic UI.** Don't wait for the read model. Immediately show the result in the UI based on what the command returned, and sync later.

**Option 3 — Version token.** The command returns a version number. The read request polls until the read model has caught up to that version.

Is your read model the same shape as your write model? AND read/write loads are similar? → Just use SQL directly. No CQRS needed. Do your reads need very different shapes, OR is the system read-heavy? → Consider CQRS, accept the consistency trade-off, design UX around it. Are you using Event Sourcing? → CQRS is almost mandatory — you have no choice but to project reads separately.


Most systems are read-heavy. But read-heavy alone isn't the trigger — the question is whether your **existing simpler solutions are failing.**

Before CQRS, fix via below

Read replicas     → scales reads horizontally, zero complexity cost
Caching (Redis)   → handles the vast majority of read load
Database indexes  → cheap, effective

CQRS makes sense when these aren't enough, or when the read/write _models themselves_ are fundamentally different — not just the load.


**Normalised SQL (write-optimised):**

```
orders table:        { id, userId, status }
order_lines table:   { orderId, productId, quantity }
products table:      { id, name, price }
customers table:     { id, name, email }
```

To show an order summary page you need to JOIN all four tables. That's expensive at scale.

**What you do in MongoDB** — you store documents like:

json

```json
{
  "orderId": "abc",
  "customerName": "Alice",
  "items": [{ "name": "Shoes", "qty": 1, "price": 99 }],
  "total": 99
}
```

This is already denormalised and read-optimised. And you're right — **this is essentially what CQRS read models do.** MongoDB sidesteps the problem by letting you store data in a read-friendly shape natively. You've been solving the "different shapes" problem all along, just without the CQRS label.

The problem only bites you with normalised SQL where your write model and read model needs are in tension. MongoDB largely avoids that tension.



### ES Without CQRS — You're Technically Right, But...

Yes, you can query ES without CQRS. It would look like:

csharp

```csharp
// Every read request replays the full event stream
var events = eventStore.GetEvents(bookingId);
var booking = new Booking();
foreach (var e in events) booking.Apply(e);
return booking; // current state
```

This works fine for a single aggregate with few events. But:

```
Booking with 20 events   → replay 20 events per read   → fine
Booking with 10,000 events → replay 10,000 events per read → slow
"Show all bookings for user" → replay ALL aggregates → completely impractical
```

CQRS isn't _logically_ mandatory — you're right. It's _practically_ mandatory because replaying events on every read doesn't scale. Projections (the CQRS read side) exist to pre-compute the read model so you're not replaying on every query.

**Real CQRS means the read data lives somewhere physically different, maintained separately from the write store.**

```
Write → PostgreSQL (normalised, consistent)
Read  → Elasticsearch / Redis / separate denormalised table (fast, shaped for queries)
```


### Buffer Is Not Mandatory — Correct

You're right. CQRS doesn't require a buffer. There are two modes:

**Synchronous CQRS (no buffer):**

```
Command → Write → immediately update Read Model → done
```

Strongly consistent. Simpler. Less scalable. Perfectly valid for many systems.

**Async CQRS (with buffer):**

```
Command → Write → publish event to queue → Read Model updates later
```

Scales better. Decoupled. But eventual consistency.

The buffer is about **scalability and decoupling**, not about CQRS itself. You nailed it.


### Stream Key and Email as Aggregate ID

A **stream key** is just the unique identifier for an event stream. Every aggregate instance gets one stream, identified by its key.

```
stream key: "booking-abc123"  →  [BookingCreated, SeatHeld, BookingConfirmed]
stream key: "booking-xyz789"  →  [BookingCreated, BookingCancelled]
```

Normally the stream key is a UUID — a meaningless technical ID. The aggregate is `User-a3f9c2b1`.

**The email-as-aggregate-ID trick:**

Instead of `User-{uuid}`, you make the stream key `User-alice@gmail.com`.

```
Request A: create stream "user-alice@gmail.com" → ✅ stream created
Request B: create stream "user-alice@gmail.com" → ❌ stream already exists, conflict error
```

The event store enforces that **stream keys are unique**. You're not checking across streams anymore — you're trying to _create a new stream_, and that either succeeds or fails atomically. One request wins, the other gets rejected.


// What if Alice wants to change her email? alice.ChangeEmail("newalice@gmail.com"); // Your aggregate ID is now wrong. Your stream is still "user-alice@gmail.com" // You'd have to create a new stream and migrate — messy


### The Honest Summary

| Your challenge                                    | Verdict                                                    |
| ------------------------------------------------- | ---------------------------------------------------------- |
| Read-heavy doesn't mean always CQRS               | ✅ Correct — try replicas/cache first                       |
| MongoDB sidesteps the "different shapes" problem  | ✅ Correct — you've been solving it implicitly              |
| ES without CQRS is technically possible           | ✅ Correct — just impractical at scale                      |
| Buffer isn't mandatory for CQRS                   | ✅ Correct — it's a scalability choice                      |
| Email stream key works differently than I implied | ✅ Correct — it's about stream creation, not event matching |

Examples

Aggregate root is responsible for the following
- Can I do this - validate
- Emitting events - this happened, it never mutate states directly
- Applying events to rebuild state



### Command handlers
---

### Real Life Examples

#### 1. LinkedIn — Profile Updates vs Search

```
Write side:  You update your job title → saved to PostgreSQL
Read side:   Your profile appears in recruiter searches → served from Elasticsearch

Why CQRS?
  Searching "senior engineer in Sydney with React skills" across
  800 million profiles is not a SQL query. Elasticsearch handles
  full-text search, faceting, ranking. PostgreSQL handles the
  authoritative profile data.

  Sync: profile update → event published → Elasticsearch index updated async
```

#### 2. Amazon — Order Placement vs Order History

```
Write side:  Place order → hits inventory, payment, order service (strong consistency needed)
Read side:   "Your orders" page → served from a denormalised read store

Why CQRS?
  The orders page needs data joined across orders, products, shipping,
  returns — all denormalised into one fast read. At Amazon's scale,
  recomputing that join on every page load from normalised tables
  is not viable.

  The write side doesn't care about that shape at all.
```

#### 3. Banking — Transactions vs Statements

```
Write side:  Record a transaction → append to ledger (event sourced, immutable)
Read side:   Monthly statement, balance, spending breakdown → projections

Why CQRS?
  "Show me all transactions grouped by category with running balance"
  is a completely different computation from "record that £50 left account X."
  The read model is pre-computed so statement generation is instant.
  This is also where ES + CQRS genuinely belong together.
```

#### 4. Uber — Ride Booking vs Driver Availability

```
Write side:  Book a ride → strongly consistent (one driver, one rider)
Read side:   "Cars near you" map → served from an in-memory geospatial store

Why CQRS?
  Querying which drivers are within 2km, updating in real time
  as drivers move, serving this to millions of users simultaneously
  — this cannot live in the same PostgreSQL table as ride bookings.
  It's a fundamentally different data access pattern needing
  a geospatial store (Redis with geo indexes, or similar).
```





Going back to your MongoDB experience — here's why you likely never needed CQRS:

MongoDB lets you store documents in whatever shape your reads need. You're not fighting a normalised schema. So the tension that drives CQRS never builds up.

CQRS becomes relevant when:

```
You're on PostgreSQL (or similar) AND
your read patterns can't be solved by indexes + read replicas + caching AND
you're at a scale where that actually matters
```



really makes sense is when it’s used with event sourcing. The system doesn’t store the final state of an entity (like a Person record) directly. Instead, it stores events e.g. “PersonCreated,” “AddressUpdated,” “EmailChanged.” To rebuild the current state, you replay those events in order. That’s what the command side does: accept commands, validate business rules, and append new events

But replaying all events every time you want to show data can be slow. That’s why the read side (or query side) exists. Mostly a NoSql Db like MongoDb or something like that. It stores the final state of the entity and uses a separate read model or database, designed for fast querying. The read store is only ever updated by triggers from the events store whenever new events happen, so reading data is quick.

So in short:

- Command side: handles writes, stores domain events.
    
- Query side: handles reads, optimized for performance.






Outbox pattern