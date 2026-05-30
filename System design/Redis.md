
### Redis
#### What is Redis
Redis is an **in-memory data store**. It stores data in its own memory, not inside app's memory (local memory). Your app has to act as a client and connect to the Redis server over the network to read or write that data. The speed advantage comes from everything living in RAM, unlike databases where everything live in storage.

#### Briefly describe Storage vs RAM
RAM is faster than Storage but RAM is volatile, lose data if power is lost. Storage retains data without power. RAM are more expensive than Storage. Storage is a lot larger than RAM.

#### Local Memory vs. Redis Memory
If you just want to store data in memory, you could use a dict, hash map in your code.

**The problem with local memory:**
- **Local memory can't be shared:** If you scale up and run five instances of your web service behind a load balancer, "Instance 1" has no idea what is stored in the local memory of "Instance 2."
    
- **It's fragile:** Every time you deploy a new version of your service, or if the service crashes and restarts, your local memory is completely wiped out.
    
**The Redis solution (Distributed caching):** Redis is essentially a remote RAM drive. It is a dedicated server (or cluster of servers) whose entire job is to hold data in its RAM and serve it over the network incredibly fast.

- **Shared State:** All five instances of your web service can connect to the exact same Redis server. If Instance 1 caches a database query, Instance 2 can instantly read that cached result from Redis.
    
- **Independent Lifecycle:** If your web services crash, scale up, scale down, or reboot, the data in Redis is perfectly safe because it lives on its own dedicated server.


---
### When to use and not use Redis

#### When to use Redis (cache-aside)?
**Caching — the obvious case.** 
When your database query is expensive and the data doesn't change every millisecond, put the result in Redis with a TTL. 

The classic pattern is **cache-aside**: application checks Redis first, if misses go to the DB, result is written back to Redis. 

At staff level you're expected to also discuss invalidation strategies — write-through, write-behind, and when TTL-based expiry is sufficient vs. when you need explicit invalidation.

**Session storage** 
HTTP is stateless. Storing sessions in a relational DB is slow and doesn't scale horizontally. Redis fits perfectly: a session ID maps to a hash of user data, with a TTL that auto-expires stale sessions. Every app server can read/write it without sticky sessions.

**Rate limiting** 
Redis's atomic `INCR` and `EXPIRE` commands make it ideal for sliding window or token bucket rate limiters. You can implement "100 requests per minute per user" with two Redis commands and no race conditions. This is a very common staff-level deep-dive topic.

**Distributed locking** The Redlock algorithm (or a simpler single-instance lock with `SET key value NX PX ttl`) lets distributed services coordinate — e.g. ensuring only one worker processes a job, or preventing double-writes.

**Pub/Sub and message queues** Redis Streams or its simpler pub/sub can fan out events to multiple consumers. Not a full replacement for Kafka — but for low-throughput fanout (notifications, cache invalidation across nodes) it's often simpler and sufficient.

**Leaderboards and counters** Sorted sets (`ZADD`, `ZRANK`, `ZRANGE`) are built for this. You can maintain a real-time leaderboard of millions of users and do rank queries in O(log N). This is a well-known Redis showcase example.

**Autocomplete / type-ahead** Sorted sets with lexicographic ordering can power prefix-search on a moderate-sized dataset in memory.

#### When NOT to use Redis

This is where staff-level answers shine. Knowing the anti-patterns signals maturity.

**Not for primary storage of important data.** Redis is not durable by default — a crash can lose recent writes unless you've configured AOF with `fsync always` (which kills performance). Even then, you're trading throughput for durability. If the data matters long-term, it lives in a relational or document DB; Redis is a complementary layer.

**Not for large blobs or binary data.** Redis values are capped at 512MB per key, but even moderate sizes are expensive at RAM prices. Storing images, documents, or large JSON payloads in Redis is almost always a mistake — use object storage (S3) and cache metadata in Redis instead.

**Not for complex queries.** Redis has no JOIN, no aggregations, no secondary indexes (outside of specific patterns like sorted sets). If your access pattern requires querying by multiple fields or ad-hoc filtering, a relational DB or search engine is the right tool.

**Not for data that needs strict ACID guarantees.** Redis transactions (`MULTI/EXEC`) are optimistic and don't support rollbacks in the traditional sense. Financial records, inventory counts that must be exact — these belong in PostgreSQL, not Redis.

**Not as a Kafka replacement at scale.** Redis Streams are great for moderate event volumes, but Kafka has persistence, replay guarantees, and consumer group semantics that Redis can't fully match at high scale.


### The staff-level framing

What interviewers look for at staff level is not just knowing when to use Redis, but how you _reason_ about the tradeoffs. Here's the mental model worth demonstrating:

At staff level the question is never "should we cache?" but rather: what is the cost of a stale read, what is the cost of a cache miss, and who is responsible for invalidation? Answer those three questions and the architecture practically writes itself.

Also be ready to discuss operational concerns: what happens when Redis goes down? (Circuit breakers, graceful degradation, the DB should survive cache loss), how do you size it? (eviction policies — LRU, LFU, volatile-lru), and how do you handle the thundering herd problem (many cache misses simultaneously — probabilistic early expiration or a distributed lock on cache population).