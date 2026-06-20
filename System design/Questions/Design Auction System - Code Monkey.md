From https://pyemma.github.io/How-to-design-auction-system/

### FR
* User can create auction
* User can place bid and get real time update on highest bid
* Auction closed if no higher bid for 1 hour
* Winner of auction would get a notification and has 10 min to make payment

### NFR
* High availability
* High scalability
* Low latency
* Eventual consistency is acceptable for live bidding part (This example contrast with other auction examples where strong consistency is required)
* Strong consistency when determining winner
* 1B users, 100k auctions a day, 10% users place 1 bid, 10:1 read write ratio

### Questions to clarify
- What happens to multiple bids with same price? First bidder is winner
- Do we allow a bidder to place multiple bids in same auction? No, each bidder only 1 bid, but they can update their bid
- Do we need to record all bids placed? Yes

---

### Auction creation
Auction Service write a new entry into `auction_table` within the db and update to cache.

![[Pasted image 20260620223542.png]]

When we create a new auction, we write it to db, update it in cache (helps with read) and mark auction as active. If db write or cache fails, we return failure to client so they can retry. We will later address cache and db atomicity.

### Auction Bidding
Two things we need to solve
1. connection mechanism between client and our service
2. mechanism for real time update to show current highest bid to users

Both of these can be addressed with WebSocket.
In this example, author used http for placing bids, and SSE with a Bid Update Service to do live updates. Their argument of WebSocket being a over kill is that we don't expect all users viewing an auction to actively place bids, so single direction connection is enough.

For showing the current highest bid, author mentioned let Bid Update Service to poll the db to see if new bids works but can put pressure on db.

When user first visit the auction page, use regular HTTP to fetch auction data. If auction is active, user initiate a SSE connection with one Bid Update Service `bus1`. `bus1` would update its in memory subscription table to record that user `u1` is viewing auction `a1`. This server would also make a request to Dispatcher saying that it's listening to `a1` and Dispatcher would update its in memory subscription table.

```python
# bus1 subscription
{ 
	'a1': ['u1', 'u2'], 
} 
# dispatcher subscription
{   'a1': ['bus1'], 'a2': ['bus2'], }
```

When user makes a bid, client send a HTTP request to Auction Service, and Auction Service makes a request to Dispatcher. Dispatcher then check its subscription table to see which Bid Update Service need this update. Once `bus1` receives the request, it can figure out which connected user it needs to send this update to.

![[Pasted image 20260620224943.png]]

Dispatcher is stateful because it needs to maintain a subscription table. If down, we can't fowad bid update anymore, we could replicate it to an external store (KV store) so other nodes can pick it up. Or have replication, where standby node is promoted to primary once original one fails.

We can also remove dispatcher, just use a distributed kv store to maintain subscription. Bid Update Service would directly make update, and Auction Service directly query to figure out which Bid Update Service it needs to send update to.

With Dispatcher:
 - pros: simplify Auction Service responsibility
 - cons: a bit more complex

Without Dispatcher:
 * pros: simpler architecture
 * cons: Auction Service needs to handle forwarding and entry





---


### The Meta Engineer's Article, Decoded

Here is a simplified, digested version of the author's architecture, translated back into standard system design concepts.

#### 1. The Core Setup (Reads vs. Writes)

The author splits the system into two primary paths to handle the massive 10:1 Read/Write ratio:

- **The Write Path (Auction Service):** A stateless HTTP service that handles incoming bids and writes them to the database. _(Critique: As we discussed, a true design should split this into an Auction Service for metadata and a Bidding Service for the actual bids. The author bundled them)._
    
- **The Read Path (Cache):** The database is too slow to serve millions of viewers checking the current price. Every time a bid succeeds, the Auction Service updates a Redis/Memcached cluster. Viewers get their data from this cache.
    

#### 2. The Real-Time Network Fan-Out (The "Dispatcher")

When someone bids, you need to tell everyone watching the item.

- **The Problem:** Standard WebSockets require a lot of overhead.
    
- **The Author's Solution:** Use HTTP POST for placing a bid (lightweight) and Server-Sent Events (SSE) for pushing the live price down to viewers.
    
- **The "Dispatcher" Hack:** Instead of broadcasting the new bid to a massive standard Kafka topic (which would overwhelm the network if thousands of servers are listening), the author introduces a custom routing layer.
    
    - The **Bid Update Service (BUS)** holds the SSE connections for the users.
        
    - The **Dispatcher** acts as a traffic cop. It keeps a mapping in memory: _"Auction A is being watched by users connected to BUS-1 and BUS-5."_ * When a bid happens, the Dispatcher surgically sends the update _only_ to BUS-1 and BUS-5.
        

#### 3. Protecting the Database (Cache Leases)

The author addresses the "thundering herd" problem: a highly anticipated item goes live, and 100,000 people bid at once.

- **The Problem:** If the cache update fails, or the cache key expires, those 100,000 requests bypass the cache and hit the database simultaneously (The Thundering Herd), crashing the system.
    
- **The Author's Solution:** Use **Cache Leases**. When the cache is missing, the system doesn't let 100,000 threads hit the database. It grants a "lease" (a temporary lock) to _one_ thread to go fetch/write the data from the DB, while making the other 99,999 threads wait a few milliseconds and try the cache again.
    

#### 4. Closing the Auction (The "Fulfillment Service")

This is where the author's design shows its "Big Tech On-Premise" bias.

- **The Author's Solution:** They built a "Fulfillment Service" (basically a massive cron job). It constantly scans the Cache to look for `expire_at` timestamps that have passed in the last minute. When it finds one, it verifies it against the DB, then triggers the payment system.
    
- **Why this is flawed:** The author admits a major weakness: _“Notice that in our design, the Fulfillment Service depends on the cache to trigger the bid execution.”_ If the cache cluster goes down or evicts keys early due to memory pressure, auctions simply _won't close_ because the cron job will never see them.
    
- **Why your way is better:** Using a Managed Delay Queue (like AWS SQS or EventBridge) is vastly superior. It guarantees at-least-once delivery, doesn't require sweeping a massive cache every 60 seconds, and is fully decoupled from the caching layer's health.
    

### Summary of the Author's Architecture

Ultimately, the author built:

1. An HTTP write-path that updates a DB and a highly protected Cache.
    
2. A custom, stateful TCP router (Dispatcher) to handle surgical real-time fan-out.
    
3. A cron-job sweeper that relies on the cache to know when to close auctions.
