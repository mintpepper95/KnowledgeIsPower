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
