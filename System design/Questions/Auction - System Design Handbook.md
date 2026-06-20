Imagine two bidders submit offers within milliseconds of each other.
One expect to win, other see a confirmation screen. Your system just accepted both bids and now auction has two winners.

How do you guarantee the highest bid wins? What happens when two bids arrive at same time? How do you handle chaotic final time of bids rushing in before auction close?

Strong candidates identify early and design around them. It's important to express that there are no perfect solutions and trade-offs are selected carefully.

System design is driven by requirements. It's important to align expectations so your design is correct.

---
### Function and non-functional requirements

#### FR
* User can place bids
* User can create auctions
* Need to determine winners

Questions we can ask is
* How do we end auctions, fixed or variable end time?
* Should bids be strictly increasing over current highest
* Can users retract bids - then we need to consider undo and audit trail

#### NFR
* Latency expectations for bid placements, can we afford db writes or do we need optimistic acknowledge with background processing
* Consistency - do we need to guarantee bid ordering (of course)
* Scalability
* Should system prioritise fairness over availability? Like can we reject a bid if system is overloaded, or should every bid attempt succeed.
* How critical are real time updates? Should they be immediate, or a slight delay like a second is fine?

---
### Entities

| Entity  | Key attributes                                              | Primary index                                           | Notes                                                                                                                  |
| ------- | ----------------------------------------------------------- | ------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| User    | user_id, email, payment_info                                | user_id                                                 | Authentication handled separately                                                                                      |
| Auction | auction_id, seller_id, end_time, current_highest_bid, state | auction_id                                              | Denormalized bid info for fast reads, however now every accepted bid needs to update both auction table and bids table |
| Bid     | bid_id, auction_id, bidder_id, amount, server_timestamp     | (auction_id, server_timestamp as we can't trust client) | Append-only audit log                                                                                                  |
| Payment | payment_id, winning_bid_id, status                          | payment_id                                              | Created only after auction close                                                                                       |

---
### Core services
Auction service - for creating and managing auctions
Bidding service - for recording bids, critical service
Real-time service - managing WebSocket connections and pushing updates to connected clients. Subscribe to bid events and fan out to relevant users.

For auctions, reads are high volume but tolerant of slight staleness. Writes (bid submissions) must be handled with strict correctness and ordering guarantees.

Reads can scale through caching and read replicas, Writes route to strongly consistent primary storage.

The bidding process requires sync handling for immediate feedback. Users need to know whether their bid was accepted. Secondary tasks like notifications, analytics, logging can happen async.

---
### Bidding workflow

Needs to consider things like race conditions, dupe writes, ordering issues when multiple users bid at same time.

When a bid is submitted, we must check things like auth token, whether bid is valid, comparing it to current highest bid etc.

The critical insights is validation and persistence must happen atomically. **If you validate a bid is highest, then persist it in another step, another bid might slip in.** This is a race condition.


#### Handling concurrent bids
Many users may submit bids within milliseconds, the system must guarantee only highest valid bid wins. There are several strategies, pessimistic locking, optimistic locking and FIFO queue processing.

Pessimistic locking locks on the auction record before processing any bid. This serialise all bid attempts, guaranteeing correctness but limiting throughput. Okay to use if moderate traffic.

Optimistic locking allows concurrent bids but detects conflicts when writing to sql. Each bid attempt reads current highest bid and its version number, then attempts an conditional update on the version being unchanged. If another bid is committed first, version mismatch cause the update to fail, and client must retry. Good for high throughput but requires retry logic. During high contention, retrying maybe less performant than pessimistic locking.

FIFO queue route all buds through a message queue. Message queues like kafka supports ordering (within same partition, so partition by auction id). This eliminates concurrency. But now introduces eventual consistency.

|Strategy|Throughput|Complexity|Best for|
|---|---|---|---|
|Pessimistic locking|Lower|Low|Most auctions, simple implementation|
|Optimistic locking|Higher|Medium|High-traffic auctions with retry tolerance|
|FIFO queue|Medium|High|Strict ordering requirements, audit needs|


---
### Idempotency and duplicate prevention
Network failures and retries can cause dupe bid submissions.
User click 'place bid', experience a timeout, and clicks again.

Without protection, system might record the same bid twice, or worse treat the retry as a new higher bid.

Idempotency keys solve this, client generates a unique id for each bid attempt, server rejects requests with previously seen keys.

Implementation can include storing idempotency keys in Redis with TTL matching retry window. Or alternatively, include key as a unique constraint in bids table. Latter provides durability but slower writes. Former is faster, but needs to consider cache failures.

---
### Correctness
It means storing bids reliably and also guarantee the highest valid bid at auction close wins, and all participants see consistent outcomes.

#### Ensuring correctness
When an auction closes, highest valid bidder wins, valid means bid was placed while auction was active and meeting all the requirements and bidder was eligible.

We can store the current highest bid on the auction record. Every accepted bid atomically update this field, we can read it at auction close time to find winner and don't need to scan all bids.

Strong consistency is required for bidding. When a bid is accepted, all subsequent reads must see that bid as the highest. Eventual consistency is only okay for secondary views, analytics, notifications etc.

Common approach is to use hybrid storage. Use in-memory cache like Redis for fast bid validation reads, backed by SQL as source of truth. Cache invalidation to maintain consistency.

---
### Time synchronisation and bid ordering
Near end of an auction, bid ordering becomes very sensitive. Two bids arriving at almost same time must be ordered deterministically.

Server-assigned timestamps to provide ordering basis. If two bids have identical timestamps, a tiebreaker rule must exist. Typically the bid that commit to db first wins.

Clock skew between servers poses a real threat in distributed deployments. Server A's clock maybe ahead of Server B, a bid processed by Server B might receive an earlier timestamp than a bid processed by Server A, even though A's bid was submitted first.

Mitigations include using logical clocks, routing all bids to a single server, or accept that guarantee of ordering between servers is just not realistic.

---
### Auction close semantics and edge cases
Auction close process requires careful design. Naive approach is check if the current time exceeds end time on each bid attempt. But what about a bid that arrives as clock strikes end time? Or a bid that was sent before close but arrives after due to network delay?

Cleanest approach is to define a hard cutoff. Bids must be fully committed before end time. Bids in flights are rejected.

Variable end time improves fairness. If any bid arrives within final N minutes, auction extends by M minutes. We need to track the ending time and update it atomically with bid acceptance.

---
### Real-time updates and UX
Polling is simplest and most naive approach, works but scales horribly.
Thousands of clients polling every second create massive load.

WebSocket connections allow server to push update immediately when bids are accepted. SSE offer simpler alternative but is one way server to client.

---
### Event driven updates and fan-out
For scalability, accepted bids are publishes as events to a message stream. Real-time Service subscribes to it and fans out updates to connected clients. This de-couples bidding from notification.

Fan-out challenge emerge for popular auctions. A celebrity may have 1m viewers. Pushing an update to 1m WebSocket connections create a thundering herd. Solutions include staggered delivery, connection sharding across multiple Real-time Service instances, batching updates. We can segment users into active bidders (WebSocket push) and audience (polling). This tiered approach match delivery urgency to user context.

We can batch updates together (merge multiple messages into one) to reduce traffic.

---
### Scaling via sharding

#### Identifying and addressing bottlenecks
* Write heavy bid submissions cause load on Bidding Service and its db
* Ready heavy browsing strain read services
* Popular auctions with many bidders create contention on specific auction records

We can shard by auction id to distribute load across database partitions.
Bids for different auctions route to different shards.

Kafka partitioning follow this principle, using auction id as partition key to put all related bids into the same partition, and partition also enforces ordering. Different auctions can be processed in parallel.

When an auction becomes very popular, the shard becomes a bottleneck. Might consider over-provisioning hot shards, rate limiting, or accept higher latency for very hot auctions.

Final minutes of an auction will tend to have a lot of traffic. Pre-scaling based on auction profile helps. Popular auctions can be flagged for additional resource allocation before they close. Auto-scaling based on active WebSocket connections or bid rate.
Graceful degradation, like temporarily disable non essential features like bid history pagination can preserve capacity for critical bidding path.

---
### Failure handling

Bidding should fail fast with clear errors rather than hanging. If db unreachable, return error immediately. If downstream service like notification which is not on the critical path fails, bidding should still succeed.

Circuit breakers to prevent cascading failures when dependent services are unhealthy.

Idempotency becomes critical during failures. A bid might succeed at db but time out before client receives confirmation. Client retries, without idempotency protection, system might reject retry as invalid (system see bid is already recorded), or just record a dupe. Idempotency must survive across retries.

Kafka idempotent producer and transactional messaging - exactly once.

### Common mistakes to avoid
* Don't ignore concurrency
* Don't assume perfect clocks which ignores real distributed system problem
* Don't over-engineer with unnecessary services
* Don't pretend design is flawless, talk about trade-offs

The strongest answers demonstrate clearly where consistency matters (bidding, winner determination) vs where eventual consistency is okay (notifications, analytics).

