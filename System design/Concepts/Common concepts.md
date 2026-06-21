
### CAP theorem
CAP is about continuing serve requests despite node failures and network partitions, not about handle huge traffic.

- Consistency  - Does different parts of the system see the same data at the same time
- Availability - Does request to a non failing node receives a response, even if that response maybe stale
- Partition Tolerance - Does system works despite network failures (timeout/dropped/delayed messages) in the system

We identify which parts of CAP do we want to prioritise during NFR. In distributed systems, partition tolerance is usually a must. So we are really just choosing between consistency and availability. Enforcing consistency forces higher latency.
#### Consistency vs Availability
Strong Consistency (reads reflect latest write)
 - Locks maybe needed
 - May need to accept higher latency
 - Don't want eventual consistency to creep in

Strong Availability
- Multiple replicas
- CDC and eventual consistency is okay
- Data might be stale but it's okay

We can choose between consistency/availability in different parts of the system

Ticketmaster
- availability for viewing events
- consistency for booking tickets

---
### Redis caching deep dive

Redis is an in-memory key value store on a server.

Note if we write to both redis and db then it's not transactional, failures can cause data drift. 

#### Eviction policies
Memory is limited. We can set a limit on how much memory get used per node before evicting.

Common approach for expiring data - TTL

LRU - least recently used key get evicted, this is most common

LFU - least frequently used key get evicted (more for long term things, e.g persistent best sellers)

TTL - key with TTL expired or shortest remaining TTL

#### Redis transactions
Redis is single threaded.

MULTI - allow running multiple transactions atomically (pessimistic locking). So other operations have to wait for these transactions to finish.

WATCH - allow doing a compare and set on key vaules (optimistic locking). If the value of this key is what I thought 10 seconds ago, then execute the command.

#### Redis durability
Redis is in memory, so we will lose data if node fails.

Need to persist to disk if we want durability
- Redis Database - snapshot data on interval and store to disk
- Append only file - Writes to disk, configurable how often entries get flushes to disk, trading durability with performance.

Can use Redis Database and Append only file at same time.

#### Redis replication
Redis is a single node key value store, but it allows replicating data using single leader asynchronous replication

If timeout, we elect a new leader.

Single leader means all writes goes to the single leader, but reads can come from leader or from replicas which maybe stale.

Also replicas can be stale, like network failures when leader writing to replicas.

If replica fails, perform re-sync with master.
We should use at least one of RDB/AOF when using replicas.
If not, master can fail and restart. But because it lost data, it will replicate an empty state to the replicas.

Evictions are performed by master and sent to replicas.

READ_ONLY commands can be handled by stale replicas.

#### Redis Sharding

In Redis cluster there are always 16384 partitions of data.

Take a key, hash it to determine its bucket.
Transactions may only operate on keys within same bucket.

You can also move partitions. For example move partition `a` from node 1 to node 2.



