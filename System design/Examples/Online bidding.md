

This excalidraw diagram below shows my system design attempt for an online bidding service.

![[excali1.excalidraw]]

---

### Online bidding prep with Claude

Prompt

Help me prepare for staff level software engineering design interview

Let's focus on the topic, how to design auction with bidding, which allows us to find highest winner. It should focus on backend. We can start from something very simple, like client -> service -> db. And then iterate to something that really scales to millions of users.

First, help me identity an overview of how to approach this problem specifically, don't give me a general overview for any system design problem.

Secondly, you come up with a list of questions for me about the approach depicted in the overview, from the point of a staff level engineering interviewing me. This is for simulating interactions. Can provide some bullet points briefly describe what do you expect in the answers for staff level.

Third, deep dive into how to tackle this design problem. With discussions about API (eg. rest or graphql), db design (sql or noSql like mongo and ACID requirements, indexing), scaling (vertical vs horizontal, scaling for DB), CAP theorem, app caching (maybe something like redis, cache invalidation strategy), message queues (if needed), failovers and resiliency, replication, and anything else that maybe missing.


# Auction System Design — Staff Level Interview Prep

---

## Overview: How to Approach This Problem

Auction with bidding is a deceptively tricky system design because it sits at the intersection of several hard distributed systems problems simultaneously.

**The core tension:** An auction has one correct winner. Bids arrive concurrently from millions of users. The system must never accept two winning bids, never lose a bid, and surface the current highest bid in near real-time — all under extreme write pressure in a short time window (the last 30 seconds of an auction gets most of the traffic).

**The iterative path this problem naturally follows:**

1. **Naive baseline** — single service, single relational DB. Establish correctness first. Identify where it breaks under load.
2. **Concurrency hardening** — optimistic locking, database-level constraints to prevent duplicate winners. The correctness layer before you touch scale.
3. **Read scaling** — most users are watching, not bidding. Separate read and write paths. Introduce caching for current price display.
4. **Write scaling** — the hard part. Bids are writes that must be serialised and consistent. Introduce a bid ingestion queue to absorb spikes, decouple acceptance from processing.
5. **Real-time fanout** — pushing live bid updates to millions of watchers. WebSocket + pub/sub layer.
6. **Auction lifecycle management** — starting auctions, closing them exactly once, determining the winner atomically. Distributed locking problem.
7. **Failure modes** — what happens when each component goes down. Circuit breakers, graceful degradation, idempotency.

**What makes this problem staff-level specifically:**

- The ending of an auction is a distributed transaction problem. Multiple services need to agree on a winner at a precise moment.
- The read/write ratio is wildly skewed and changes over time (quiet phase vs. last-minute frenzy).
- Correctness (exactly one winner, no phantom bids) must be preserved even under partial failure.
- You need to reason about consistency models explicitly — not just "use strong consistency everywhere" but where you can afford eventual consistency and where you cannot.

---

## Interview Simulation Questions

---

**Q1. Walk me through the simplest possible version of this system. Just make it work correctly first.**

_What I expect at staff level:_

- Identifies the core entities: Auction, Item, Bid, User
- Proposes a relational DB immediately and justifies why (ACID, strong consistency needed for winner determination)
- Describes a single bid endpoint that writes to DB and returns acceptance/rejection
- Critically: mentions how you prevent two concurrent bids from both being accepted as the highest — optimistic locking, `SELECT FOR UPDATE`, or a DB unique constraint on (auction_id, winning status). A candidate who skips straight to Kafka without solving correctness first is a red flag.

---

**Q2. The auction is ending in 30 seconds and 50,000 users are all placing bids simultaneously. What breaks and how do you fix it?**

_What I expect at staff level:_

- Names the specific failure: write contention on the bids table / auction row. Not vague "it won't scale" but precisely where the bottleneck is.
- Considers a bid ingestion queue (Kafka/SQS) to absorb the spike — decouple receiving bids from processing them. Understands the tradeoff: bids are now async, so you can't immediately tell the user "you're currently winning."
- Discusses whether that async UX is acceptable and how to communicate bid status back (polling, WebSocket acknowledgement).
- Mentions rate limiting per user — prevents a single user from hammering the system.

---

**Q3. 1 million users are watching the current highest bid update in real time. How does that work?**

_What I expect at staff level:_

- Separates the read path (displaying current price) from the write path (accepting bids) — these have completely different scale requirements.
- WebSocket connections for live updates. Mentions that a single server can't hold 1M WebSocket connections — need a fleet of servers.
- Redis Pub/Sub or Kafka as the fanout backbone between servers. Server 1 processes a new highest bid, publishes to Redis, all WebSocket servers receive it and push to their connected clients.
- Caches the current highest bid in Redis so new connections don't have to query the DB just to show the current price.

---

**Q4. How do you guarantee exactly one winner when the auction ends?**

_What I expect at staff level:_

- Recognises this as a distributed coordination problem, not just a query.
- Auction close must happen exactly once — mentions distributed locking (Redis SETNX / Redlock) or a leader election mechanism to ensure only one process runs the close logic.
- The winner determination query must be atomic — ideally a single DB transaction that reads the highest bid and marks it as the winner, preventing any new bids from being accepted simultaneously.
- Discusses the edge case: a bid that arrives in-flight exactly as the auction closes. How do you handle it? (A grace period + idempotent bid processing, or a hard DB constraint on auction status.)
- Considers the auction close as an event published to a message queue — downstream services (payment, notifications) consume it independently without coupling to the close transaction.

---

**Q5. How does your DB schema handle this, and what indexes do you put on it?**

_What I expect at staff level:_

- Core tables: `users`, `items`, `auctions`, `bids` — and knows why bids is a separate append-only table, not a field on auctions.
- Index on `(auction_id, amount DESC)` for fastest highest-bid lookup.
- Index on `(auction_id, created_at)` for bid history ordering.
- Composite unique constraint considerations for winner status.
- Mentions that bids is a write-heavy, append-only table — a good candidate for partitioning by `auction_id` or time range at scale.
- Knows the difference between a covering index and why it matters here.

---

**Q6. Where can you afford eventual consistency and where absolutely cannot you?**

_What I expect at staff level:_

- **Must be strongly consistent:** Winner determination, bid acceptance/rejection (you can't tell two users they're both winning), payment processing.
- **Can be eventually consistent:** The displayed current highest bid (a slight lag of a few hundred ms is acceptable — users don't notice), bid history for past auctions, search/browse of active auctions.
- Frames this in terms of CAP theorem: during a network partition, the bid acceptance service must choose CP (consistency over availability) — better to reject bids than to accept conflicting ones. The read/display service can choose AP.
- Mentions that this is why you might have two separate services with different consistency models.

---

**Q7. The Redis instance holding your current-bid cache goes down. What happens?**

_What I expect at staff level:_

- Circuit breaker pattern — detect Redis is down, open the circuit, fall back to DB reads.
- Graceful degradation — the system keeps working, just slower (more DB load). Live WebSocket updates may lag or pause briefly but auctions don't stop.
- Thundering herd risk: if Redis comes back up with a cold cache during peak load, every request hits the DB simultaneously. Mentions probabilistic cache warming or a distributed lock on cache repopulation.
- Considers whether the WebSocket fanout also goes through Redis Pub/Sub — if so, a Redis outage breaks live updates too. Discusses whether to have a fallback (polling) or a Redis cluster with replication for HA.

---

## Deep Dive

### API Design

**REST over GraphQL here.** Auctions have clear, well-defined resource semantics. GraphQL's flexibility is a liability when you need aggressive caching and rate limiting at the bid endpoint. The bid endpoint in particular needs to be as lean and fast as possible.

Key endpoints:

```
POST   /auctions                          # create auction
GET    /auctions/{id}                     # get auction detail + current price
GET    /auctions/{id}/bids                # paginated bid history
POST   /auctions/{id}/bids                # place a bid  ← the hot path
GET    /auctions/{id}/winner              # get winner after close

WebSocket: /auctions/{id}/live            # real-time price updates
```

The bid endpoint deserves its own discussion. It must be:

- **Idempotent** — a client retry on network failure must not create a duplicate bid. Include a client-generated `idempotency_key` in the request, stored in Redis with a short TTL.
- **Rate limited** — per user, per auction. Redis atomic `INCR` on a key like `ratelimit:user:123:auction:456`.
- **Fast to accept, async to process** — at scale, "accepting a bid" means writing to a queue. The final confirmation of whether the bid is the current highest comes asynchronously.

---

### Database Design

**SQL (PostgreSQL), not NoSQL.** Bids require ACID transactions. You must never accept a winning bid and lose it, never have two winners, never record a bid twice. MongoDB's document model gives you nothing here and costs you transactions.

**Schema:**

```sql
users
  id, email, created_at

items
  id, title, description, seller_id

auctions
  id, item_id, seller_id,
  reserve_price, start_time, end_time,
  status (PENDING | ACTIVE | CLOSED | CANCELLED),
  winning_bid_id (FK → bids, nullable until close),
  version (for optimistic locking)

bids
  id, auction_id, bidder_id,
  amount, placed_at,
  status (PENDING | ACCEPTED | REJECTED | RETRACTED),
  idempotency_key (unique)
```

**Why `bids` is append-only:** Never update or delete bids. Every bid is a permanent record. The "current highest bid" is derived — a query on the bids table, not a field you mutate.

**Indexes:**

```sql
-- Fastest query: what's the current highest accepted bid on this auction?
CREATE INDEX idx_bids_auction_amount
  ON bids (auction_id, amount DESC)
  WHERE status = 'ACCEPTED';

-- Bid history, paginated
CREATE INDEX idx_bids_auction_time
  ON bids (auction_id, placed_at DESC);

-- Idempotency enforcement
CREATE UNIQUE INDEX idx_bids_idempotency
  ON bids (idempotency_key);

-- Auction lookup by status (for scheduler/cleanup jobs)
CREATE INDEX idx_auctions_status_end_time
  ON auctions (status, end_time)
  WHERE status = 'ACTIVE';
```

**Preventing concurrent bid conflicts:**

When accepting a bid, you need to ensure no higher bid was accepted between when you read and when you write. Two approaches:

Option A — `SELECT FOR UPDATE` (pessimistic locking):

```sql
BEGIN;
  SELECT * FROM auctions WHERE id = ? FOR UPDATE;
  -- now you hold the lock, check current highest bid
  -- insert new bid if it's higher
  -- update auction.winning_bid_id
COMMIT;
```

Simple and correct. Becomes a bottleneck under heavy concurrency because every bid waits on the lock.

Option B — Optimistic locking with version column:

```sql
BEGIN;
  SELECT version FROM auctions WHERE id = ?;
  -- check bids, decide to accept
  UPDATE auctions SET winning_bid_id = ?, version = version + 1
    WHERE id = ? AND version = <read_version>;
  -- if 0 rows updated, someone else won the race, retry
COMMIT;
```

Better throughput under contention. You retry on conflict rather than waiting. At very high bid rates, retry storms become a concern — which is why at scale you move bid serialisation out of the DB entirely.

---

### Scaling Strategy

**Phase 1 — Vertical scaling.** Before anything else, right-size your DB instance. A single powerful PostgreSQL instance can handle a lot. Don't prematurely distribute.

**Phase 2 — Read replicas.** Read traffic (browsing auctions, viewing bid history, displaying current price) is routed to read replicas. Writes (placing bids, closing auctions) go to the primary only. This alone handles most read load.

**Phase 3 — Caching layer (Redis).** Current highest bid is the most frequently read piece of data and changes at most every few seconds. Cache it:

```
key:   "auction:456:current_bid"
value: {amount: 1500, bidder_id: 789, timestamp: ...}
TTL:   30 seconds (or invalidated explicitly on new accepted bid)
```

Use **write-through** invalidation — when a new highest bid is accepted, immediately update Redis. TTL is a safety net, not the primary invalidation mechanism.

**Phase 4 — Horizontal app server scaling.** Stateless app servers behind a load balancer. Session state in Redis, not local memory. WebSocket connections require sticky routing (consistent hashing by auction ID to a server, or a Redis Pub/Sub coordination layer as described above).

**Phase 5 — Bid ingestion queue.** For the last-minute spike problem, decouple bid receipt from bid processing:

```
Client → Bid API → Kafka (bids topic) → Bid Processor → DB
                        ↓
                   Immediate ACK to client ("bid received")
                        ↓
                   Async result via WebSocket ("bid accepted / outbid")
```

The Bid API is now a thin layer that validates the bid (is the auction active? is the amount above reserve? passes rate limit?) and writes to Kafka. It can scale horizontally and independently of the DB. The Bid Processor consumes from Kafka and serialises DB writes — you can control the rate of DB writes independently of ingestion rate.

**Phase 6 — DB partitioning (when you outgrow a single primary):**

Partition the `bids` table by `auction_id`. All bids for a given auction live on one shard — queries never cross shard boundaries. The partition key matches the access pattern perfectly.

---

### CAP Theorem Application

This system has two distinct subsystems with different CAP requirements:

**Bid acceptance — CP (Consistency over Availability):**

During a network partition, it is better to reject bids than to accept conflicting bids on two different nodes. Two users thinking they're both winning is a worse outcome than temporarily being unable to place a bid. You choose consistency.

**Current price display — AP (Availability over Consistency):**

During a partition, it is better to show a slightly stale price than to show nothing. A user seeing a price that's 500ms stale is fine. You choose availability. This is why caching with short TTL is acceptable here — you're explicitly choosing eventual consistency for the display layer.

This split is the answer interviewers want when they ask about CAP in this context. Not "we need strong consistency everywhere" — but a deliberate, justified split.

---

### Auction Lifecycle and Distributed Coordination

The hardest single problem in this design: **closing the auction exactly once and determining exactly one winner.**

If you have multiple app servers and a scheduler fires "close auction 456" — what prevents two servers from both running the close logic simultaneously?

**Distributed lock with Redis:**

```
SETNX "auction_close_lock:456" <server_id> EX 30
```

Only one server acquires the lock. It runs the close transaction. Lock expires after 30 seconds as a safety net against the server dying mid-close.

**The close transaction itself:**

```sql
BEGIN;
  -- Lock the auction row
  SELECT * FROM auctions WHERE id = 456 FOR UPDATE;

  -- Verify not already closed (idempotency guard)
  IF auction.status != 'ACTIVE' THEN ROLLBACK; RETURN;

  -- Find the winner
  SELECT * FROM bids
    WHERE auction_id = 456 AND status = 'ACCEPTED'
    ORDER BY amount DESC LIMIT 1
  FOR UPDATE;

  -- Mark winner, close auction
  UPDATE auctions SET status = 'CLOSED', winning_bid_id = <bid_id>
    WHERE id = 456;

  UPDATE bids SET status = 'WINNING' WHERE id = <bid_id>;
COMMIT;
```

After the transaction commits, publish an `auction.closed` event to Kafka. Payment service, notification service, and any other downstream consumers react independently. The close transaction itself does not call those services — decoupled via event.

---

### Real-Time Updates Architecture

```
New highest bid accepted
        ↓
Bid Processor publishes to Redis channel "auction:456:updates"
        ↓
All WebSocket servers (subscribed to that channel) receive it
        ↓
Each server pushes to its connected clients watching auction 456
```

For 1 million concurrent watchers, WebSocket servers are stateless workers. A client watching auction 456 connects to any server via load balancer. That server subscribes to the Redis channel for auction 456. When it receives a pub/sub message, it pushes to all its local connections watching that auction.

This means the number of Redis pub/sub messages is constant (one per bid update), not proportional to the number of watchers. Redis fans it out to N WebSocket servers; each server fans it out to its local connections.

---

### Failover and Resiliency

**Redis goes down:**

- Circuit breaker opens. Current bid reads fall back to DB read replica.
- WebSocket live updates pause or fall back to client-side polling at 2-second intervals.
- Bid rate limiting falls back to an approximate in-memory counter (slightly less accurate, acceptable for degraded mode).

**Kafka goes down:**

- Bid API can temporarily fall back to synchronous DB writes with aggressive rate limiting. Volume must be throttled or bids are rejected with a retryable error.
- Alternatively, buffer to a local disk queue and replay when Kafka recovers.

**DB primary goes down:**

- Read replicas are promoted. This takes 30–60 seconds typically with automatic failover (e.g. RDS Multi-AZ).
- During promotion window, bid acceptance is unavailable. Return `503 Service Unavailable` with `Retry-After` header — clients know to retry.
- Auctions do not silently accept or reject bids during this window. They pause.

**Bid Processor goes down:**

- Kafka retains messages. When Processor restarts, it picks up from its last committed offset.
- Bids are delayed but not lost. Idempotency keys prevent duplicates on replay.

---

### Summary: The Scaling Journey

```
Phase 1:  Client → App Server → PostgreSQL
          (correct, simple, breaks at ~1k concurrent bidders)

Phase 2:  + Optimistic locking / SELECT FOR UPDATE
          (correct under concurrency, still single DB bottleneck)

Phase 3:  + Read replicas + Redis cache for current bid
          (read traffic handled, write path still bottleneck)

Phase 4:  + Bid ingestion via Kafka
          (write spike absorbed, bid processing serialised)

Phase 5:  + WebSocket servers + Redis Pub/Sub fanout
          (real-time updates at scale)

Phase 6:  + Distributed lock for auction close
          + Auction closed event → downstream consumers
          (correct lifecycle management at scale)

Phase 7:  + DB partitioning by auction_id
          + Circuit breakers on every external dependency
          (full production resilience)
```

Each phase solves a specific, nameable problem. At staff level, you're expected to know _why_ each layer was added, not just _what_ it is.