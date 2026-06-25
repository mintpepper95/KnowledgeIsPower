

Flow 1: Strong-consistency write (bid, ride request, payment, seat booking, inventory decrement)

The write must be validated against current state and committed atomically. Pattern:

- Synchronous DB transaction with a conditional update or row lock to serialize concurrency: `UPDATE ... WHERE current_max < :bid` or `SELECT ... FOR UPDATE`.
- Outbox pattern + CDC to publish the resulting event without a dual-write problem.
- Idempotency key to make client retries safe.
- For multi-entity atomicity across services (reserve seat + charge card), you need a **saga** (orchestrated compensating transactions) since you can't hold a distributed ACID transaction cheaply.



kafka durable: Kafka achieves durability by immediately writing data to disk, copying that data across multiple independent machines, and allowing the system administrator to mandate that a write isn't "complete" until multiple machines have safely stored it.


Note the _propagation to other viewers / downstream systems_ (leaderboards, notifications, other users' displays) is still allowed to be eventually consistent. Strong where the invariant is enforced; eventual everywhere the lag doesn't break an invariant. A payment must be decided strongly; the recipient seeing "you got paid" a second later is fine.

---

**Flow 2: Eventual-consistency write (social post, comment, like, analytics event)**

No authoritative cross-system invariant to maintain. Pattern:

- Durable queue as the handoff (acks=all), ack the user fast.
- Idempotent consumer persists at-least-once.
- No outbox needed _if_ the producer has no local DB write to keep in sync.

Core tension: latency and throughput over immediate consistency.

---

**Flow 3: Write that fans out (post → followers' feeds, tweet → timelines)**


Imagine someone post something and it needs to appear to a bunch of subscribers.

Do I push a copy to every subscriber's feed?
Or do I push leave it until and gather it until someone opens/refresh their app?

So the defining feature is recipient has different views (personalisation).

So showing highest bidding price in auction is not a fan out problem. It's a single value broadcasted to everyone. Same with leaderboard, everyone gets same values.

These are about once something changes, how do I push it to all the clients.

The write itself is easy; the _propagation_ is the problem. This is the fan-out-on-write vs fan-out-on-read decision:

- **Fan-out on write (push):** precompute each follower's feed at post time. Fast reads, expensive writes, terrible for celebrities (1 post → 100M feed writes).

- **Fan-out on read (pull):** assemble feed at read time. Cheap writes, expensive reads.
- **Hybrid:** push for normal users, pull for high-follower accounts. This is what real systems do.

**What real systems actually do — hybrid:**

- **Normal users:** fan-out on write. Their followers' feeds get the post pushed at write time → fast reads for everyone.
- **Celebrities / huge accounts:** fan-out on read. Don't push their posts to millions of feeds; instead, at read time, _merge_ the precomputed feed (from normal authors) with a live pull of the few high-follower accounts the user follows.

Per-user view?Mechanism**Highest bid**No — one shared valueBroadcast: event → pub/sub → WebSocket. Not fan-out-write/read.**Leaderboard**No — one shared rankingSame broadcast (or just let clients poll the ZSET). Not fan-out-write/read.**Feed**Yes — every timeline differsFan-out write/read decision applies; hybrid in practice.


---

### Read flows

**Flow 4: Read-heavy, tolerates staleness (product page, profile, feed)**

- Cache aggressively (CDN → Redis → DB).
- Cache-aside is the default; decide TTL vs explicit invalidation.
- Replicas for the DB; reads hit replicas, writes hit primary (accept replication lag).

---

**Flow 5: Read that must be fresh (account balance, current auction price, seat availability)**

- Read from primary, or use read-your-writes consistency.
- Often the same entity that's under a strong-consistency write — keep the read on the authoritative path.

We can still use cache, it's **the cache must never be authoritative on the path where correctness depends on it.** You can still use a cache; you just can't _trust it for the decision.

So if we need the value for making critical correctness action, then can't use cache.





---

**Flow 6: Search / query by attribute (search posts, find drivers nearby)**

- Primary DB isn't the query engine. Replicate into a specialized index: Elasticsearch for text, a geospatial index (geohash / S2 / quadtree) for "nearby."
- Populated via the same CDC/event stream — eventually consistent with the source.

---

**Flow 7: Scheduled / delayed action (auction ends at T, ride timeout, reminder)**

- Don't poll the whole table. Use a delay queue, a timer service, or a sorted set (Redis ZSET keyed by timestamp) you pop from.

---

**Flow 8: Real-time push to client (bid updates, ride location, chat)**

- WebSockets / SSE / long-poll, fed by the event stream (Kafka → fan-out service → connected clients).
- Need a connection registry (which user is on which server) — often Redis pub/sub or a sticky routing layer.

For auction prices using Redis pub/sub for showing the current highest bidding price, (the next bid corrects it, and a client reconnect can re-fetch current state from Redis/DB). If you need guaranteed delivery you'd use Kafka itself or a stream with retention rather than pub/sub. But for "push the latest price to whoever's currently watching," pub/sub is the standard, pragmatic choice.


### How to apply this in an interview

The decision tree that selects the flow:

1. **Is it a read or a write?**
2. **If write: is there a cross-system invariant that must hold atomically?** Yes → Flow 1 (strong, outbox, saga). No → Flow 2 (eventual, durable queue).
3. **Does the write need to propagate to many entities?** → Flow 3 (fan-out decision).
4. **If read: how stale can it be?** Tolerant → Flow 4 (cache + replicas). Must be fresh → Flow 5 (primary read).
5. **Is the access pattern not "by primary key"?** → Flow 6 (secondary index / search store).
6. **Is there a time or push dimension?** → Flow 7 (scheduling) or Flow 8 (real-time).

Your auction example actually touches almost all of these: Flow 1 (the bid), Flow 5 (current price read), Flow 8 (live bid updates to other watchers), Flow 7 (auction-end trigger), Flow 9 (bid count). That's why "design an auction" is a good interview question — it's a stress test across the whole catalog.



---

### Websocket bidrection or Websocket for stream, HTTP for actions

**Streaming, high-frequency, latency-coupled client→server → put it on the WebSocket.** **Discrete, occasional, "fire an action and get a result" → plain HTTP request, even if you're also holding a WebSocket for the server→client stream.**
















### What is Saga?

Saga is basically undo.

Example, booking a trip: reserve a seat, charge card, issue ticket
These are done on separate systems

If charge fails after seat selection, we must be able to undo seat reservation.
Sure you can emit a charged failed event that cancels the seat reservation.

### So what are you actually choosing between?

Not "saga vs no saga." You're choosing **how to coordinate the compensations**:

- **Ad-hoc events (what you described):** charge-fail event → seat-release handler. Works great for 2–3 steps. This is choreography.
- **An orchestrator:** a coordinator that knows the whole sequence and issues the undo calls. Worth it when there are many steps or branching failure modes.

Both are sagas. The question is only how much structure you want around the events.

### When your event-driven undo starts to hurt

Your lightweight approach is genuinely fine for the seat+charge example. The reason the "saga pattern" gets formalized is that the naive event approach degrades as complexity grows. Watch what happens when you add steps:

```
reserve seat → charge card → issue ticket → send confirmation
```

Now if "issue ticket" fails, you must undo _two_ prior steps (refund card, release seat), in order. With pure ad-hoc events you now need:

- the ticket-fail event to trigger a refund,
- the refund-done event to trigger a seat release,
- and you have to make sure a _late_ success event for a step you already compensated doesn't re-apply it.


