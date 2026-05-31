
[[#What is Redis]]
[[#Briefly describe Storage vs RAM]]
[[#Explain storing data in Local Memory vs. Redis Memory]]
[[#What are the two messaging mechanisms in Redis?]]
[[#Pub/Sub is server-to-server, not server-to-browser (fanout)]]
[[#When to use Redis (Caching/Server-side session storage/rate limiting/leaderboards)?]]
[[#When NOT to use Redis (not durable/no delivery guarantee/not for large data/not for complex queries/not for ACID/not for scale)]]
[[#Operational concerns]]

---
### Redis
#### What is Redis
Redis is an **in-memory key value store**. It stores data in its own memory, not inside app's memory (local memory). Your app has to act as a client and connect to the Redis server over the network to read or write that data. 

Speed advantage comes from everything living in RAM, unlike databases where everything live in storage.

#### Briefly describe Storage vs RAM

|              | RAM                     | Storage |
| ------------ | ----------------------- | ------- |
| **Speed**    | Fast                    | Slow    |
| **Volatile** | Lose data on power loss | No      |
| **Cost**     | Higher                  | Lower   |
| **Capacity** | Smaller                 | Larger  |
#### Explain storing data in Local Memory vs. Redis Memory
You could just use a dict, hash map in your app's own memory to cache data.

**Problem with local memory:**
- **Local memory can't be shared:** If you scale up and run five instances of your web service behind a load balancer, "Instance 1" has no idea what is stored in the local memory of "Instance 2."
    
- **Lost on restart:** Every time you deploy a new version of your service, or if the service crashes and restarts, your local memory is wiped out.
    
**The Redis solution (Distributed caching):** Redis is essentially a remote RAM drive. It is a dedicated server (or cluster of servers) whose entire job is to hold data in its RAM and serve it over the network incredibly fast.

- **Shared State:** All five instances of your web service can connect to the exact same Redis server. If Instance 1 caches a database query, Instance 2 can instantly read that cached result from Redis.
    
- **Independent Lifecycle:** If your web services crash, scale up or down or reboot, data in Redis is perfectly safe because it lives on its own dedicated server.

---
### Redis for pub/sub

#### What are the two messaging mechanisms in Redis?

- **Simple Pub/Sub** — fire and forget. If nobody is listening when you publish or consumer down, the message is gone. No persistence.
- **Redis Streams** — a durable log. Messages are stored on disk. Think of it as a lightweight Kafka. Kafka scale better and offers guaranteed delivery.


#### Pub/Sub is server-to-server, not server-to-browser (fanout)

Do not confuse Pub/Sub with WebSocket. Pub/Sub is about how backend services talk to each other. WebSocket is how backend service talk to the browser.

```
[Event happens in backend]
        ↓
  Pub/Sub / Kafka / Redis Streams   ← this is server-to-server messaging
        ↓
  [Your notification service receives it]
        ↓
  WebSocket / SSE / Push notification ← this is server-to-client delivery
```


Redis Streams or its simpler pub/sub can fan out events to multiple consumers. It is appropriate for **non-critical, low-throughput fanout** where you already have Redis and don't want to operate Kafka:

- Cache invalidation signals across app servers ("everyone, evict key X")
- Simple notifications where losing an occasional message is acceptable

##### Example with Redis pub/sub for chat room

* Real-time chats: When you send a message in a chat room, it gets published to a Redis channel, and everyone who subscribe to it sees it instantly. For example, User A sends a message. Chat Server 1 receives it. User B and C are connected to Chat Server 2, a completely different process, possibly on a different machine. Server 1 can publish to Redis channel "room:123". Redis then fan out to all subscribers, and the subscribed chat servers push to its users via WebSocket.

Redis pub/sub operates on fire and forget. If message is somehow missed, then missed forever, redis itself does not store history of pub/sub messages (Redis streams do)


---
### When to use and not use Redis

#### When to use Redis (Caching/Server-side session storage/rate limiting/leaderboards)?
**Caching (cache-aside)**

The classic pattern is **cache-aside**: application checks Redis first, if misses go to the DB, result is written back to Redis with a TTL.

When your database query is expensive and the data doesn't change often and are read heavy (are we repeatedly fetching the same data?)  This also helps reducing requests to DB.


**Server side Session storage**
HTTP is stateless. When you log in, server create a session, the client gets a session id as a cookie. Browser sends that cookie with every request. Server can look up the session cookie in Redis. Also note relational DBs don't do TTLs like Redis beside being slower.

Modern alternative - JWT, session data is encoded in the JWT. Therefore no server side storage needed.

**Rate limiting**
Redis's atomic `INCR` and `EXPIRE` commands are atomic. 

Let's say if we want some data/count in redis to be < 100, we increment every time we hit Redis. If we have two concurrent incoming requests, since `INCR` is atomic, only 1 request will increment that data, the other can't read it until increment is over, no race condition.

**Leaderboards and counters** 
_Sorted sets_ in Redis (`ZADD`, `ZRANK`, `ZRANGE`) are built for this, Redis keep them sorted automatically. You can maintain a real-time leaderboard of millions of users.

If we sort in sql, even with index, constantly updating scores and querying ranks across millions of rows is expensive as we need to count all the rows with a higher score. Redis's sorted set is specifically designed for this — rank queries are O(log N) and updates are fast.

Rank query means a query doesn't just ask "What is PlayerX's score?". 
It asks, Based on everyone's scores, what place is PlayerX in.


##### Redis can also help a fleet of servers too.
Assume you have 5 app servers, each holding WebSocket connections for 200,000 users each (1 million total).

A score update comes in — say Djokovic just won a point. One server receives that event. How do all 1 million connected users see it?

Without Redis, users on other servers can't get the update. Server 1 has no way to reach WebSocket connections held by other Servers.

With Redis. Server 1 can publish to a Redis channel. And Redis can fan out to all 5 servers. Each server will push to its WebSocket connections. All 1 mil users will get the update.


---

#### When NOT to use Redis (not durable/no delivery guarantee/not for large data/not for complex queries/not for ACID/not for scale)

**Not for primary storage of important data.** Redis is not durable by default — a crash can lose recent writes unless you've configured AOF with `fsync always` (which kills performance). 

Even then, you're trading throughput for durability. If the data matters long-term, it lives in a relational or document DB; Redis is a complementary layer. It does not guarantee delivery of data.

**Not for large blobs or binary data.** Redis values are capped at 512MB per key, but even moderate sizes are expensive at RAM prices. Storing images, documents, or large JSON payloads in Redis is almost always a mistake — use object storage (S3) and cache metadata in Redis instead.

**Not for complex queries.** Redis has no JOIN, no aggregations, no secondary indexes (outside of specific patterns like sorted sets). If your access pattern requires querying by multiple fields or ad-hoc filtering, a relational DB or search engine is the right tool.

**Not for data that needs strict ACID guarantees.** Redis transactions (`MULTI/EXEC`) are optimistic and don't support rollbacks in the traditional sense. Financial records, inventory counts that must be exact — these belong in PostgreSQL, not Redis.

**Not as a Kafka replacement at scale.** Redis Streams are great for moderate event volumes, but Kafka has persistence, replay guarantees, and consumer group semantics that Redis can't fully match at high scale.

---
### The staff-level framing

What interviewers look for at staff level is not just knowing when to use Redis, but how you _reason_ about the tradeoffs. Here's the mental model worth demonstrating:

At staff level the question is never "should we cache?" but rather: what is the cost of a stale read, what is the cost of a cache miss, and who is responsible for invalidation? Answer those three questions and the architecture practically writes itself.

#### Operational concerns

##### Circuit Breaker

Circuit Breaker pattern can wrap Redis.

```
States:
CLOSED (normal) → calls Redis
OPEN (failing)  → skips Redis, goes straight to DB
HALF-OPEN       → tries one request to see if Redis recovered
```

Without a circuit breaker, if Redis is down, each request will wait for a timeout before falling through to the DB, your app hangs. Also app will continue to function (just slower since now need to go to DB every time).

##### Eviction Policies
When Redis run out of memory, it must evict keys, eviction policy determines how to evict. LRU (Least recently used), LFU (Least frequently used, good for popular items) or evict keys closest to expiry (TTL).

##### Thundering Herd Problem
A popular cache key expires. 500 requests come in, all get a cache miss, all go to the DB. The DB gets hit 500 times for the same query in milliseconds. This can overwhelm the DB.

1. Probabilistic early expiration - As key approach its TTL, a request will become more likely to reach DB to get the value and thus refresh the cache.
2. Distributed lock - When there is a cache miss,  only one process rebuilds the cache. Rest wait or return stale data.
