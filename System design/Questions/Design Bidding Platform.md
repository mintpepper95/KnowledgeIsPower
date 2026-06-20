Based on Jordan has no life video

Bidding platform
* Current bid
* Number of bids
* Highest bidder
* End time

Users can bid on items (highest price wins)
Auctions can have fixed end time or variable depending on last bid
Users should get price update in real time
Needs to be able to support many incoming bids per second

### Approach

`Data Models`
Bid: bidId, auctionId, userId, price, server_timestamp (don't want to use timestamp from clients), status (is it the current highest bid)

We want all bids per auction for auditing purpose ( e.g. primary buyer somehow leaves, then we want to give it to second highest bidder). We could put it into a time series database, and shard by AuctionId.

Auction State: AuctionId, currentHighestBidId, price, endtime

Each field in Auction state has 4 fields, so 8 bytes each and 32 in total.


#### Philosophy of problem

Multiple concurrency requests with same bidding price - who wins?

We need a single choke point for bids to go through to order our bids.

#### Kafka
All bids go into Kafka queue, ordered by log.

![[Pasted image 20260618151048.png]]
A partition is a smaller part of a topic. Kafka topic is split into multiple partitions, which are ordered, immutable logs of messages.

Bottleneck as only one machine can handle read and write.
Here we can use auctionId as key to compute the partition to go to. So each partition will be an auctionId.

Kafka can only determine ordering within same partition, not across the topic.

Kafka is bad for random reads (something equivalent to select * from auctions where auctionId = 123)

Pro: Kakfa has high throughput
Con: Users notified of their own bid status is now async (eventual consistency)

So when we have two concurrent bids with same price, the first one to go to kafka wins.


#### Sync bids
For good UX, we want to allow users to bid AND get the response of accepted or rejected in sync compared to using kafka.

Bid Engine is going to accept incoming bids and process them, this is now our choke point instead of Kafka.

Throughput needs to be high.

![[Pasted image 20260618152150.png]]

For Bid Engine
* Ideally, we don't want to do disk reads/network/db calls to decide whether bid is valid, since auction object is small and has just 4 states, can keep in memory
* As each request comes in, lock auction state, compare bids and proceed
* If we get a lot of traffic, we can partition in traffic, such that all bids with same auctionId go to the same engine (have to ensure consistent hashing on auctionId)


#### Fault tolerance
Backup Bid Engine, in case primary goes down.

We don't want to send concurrent bids to both primary and back up, as these two may disagree.

All bids go to primary. And primary orders them. ( can assign sequence numbers)

Primary sends ordered bids to backup. (replication)

Now both have identical state. only now do we return a OK response to the client.

If Primary dies before sending to backup, then bid is uncommitted.


![[Pasted image 20260618153101.png]]

#### Message delivery to interested parties
When a bid is handled by primary, many parties interested
* backup bid engine
* Audit database
* Other users

We can send the processed bids to kafka, and multipple consumers can subscribe, kafka is fault tolerant, depending on replication configuration.


#### Updated flow

![[Pasted image 20260618154412.png]]

* This prevents bid engine failing before publishing to kafka
* We still have risk of bids getting lost if Kafka goes down
* The more safe we are with replication, the slower the write

Kafka is the source of truth here.
Since Kafka is the truth, outbox pattern of Bid Engine writing to DB then to Kafka is unnecessary. Recall Outbox pattern is about writing to DB and inserting to outbox in the same transaction.

So how can we keep bid engine throughput high with extra network call of Kafka?

Bid Engine - receive Auction events.
Get the auction state using auctionId. Lock it, and update the auction from new bid. This is critical section.

Then upload to Kafka. # However, because multiple bids can be processed at same time, a bid with sequence number 6 might reach kafka before sequence number 5. We still have sequence numbers, so downstream consumers can use it to process in order. So basically if they see a sequence x, they don't process it until x - 1 has been processed.

However if bidding engine goes down, and downstream has 6 but waiting for 5, then 5 is never sent.

How to avoid it. We just have to publish in proper order. So put into an concurrent queue as part of the things to do when auction state is locked. And another thread can poll from that queue every once in a while to actually publish to kafka. This is a very similar idea to Outbox.

Basically we avoid holding auction lock while waiting for kafka.

- Sequence number assignment happens under the auction lock.
- Messages are enqueued in sequence order.
- Publisher thread publishes in FIFO order.

This is solving Ordering, not Durability.


#### Bid consumers

TimeSeriesDB can be shard by auctionId.

![[Pasted image 20260618161259.png]]
Stateful consumer will be aggregating the auction states and see the highest auction price, and use websocket to send events to server.


#### Popular Auctions
E.g. million of people watching, maybe too much load to consume all bids and send to all users.


Backup Engine after receving bids from Kafka queue, can put the new accepted bids into the Kafka queue (so always current highest) and then subscribers can subscribe to this queue. So significantly fewer price updates.

![[Pasted image 20260618161705.png]]



#### Variable time auction
As bids come in and get accepted, we update auction finish time in memory based on bid timestamp. Bid engine can check in interval for auction finish time. Once auction is finished, Bid engine can write the winning bid id to db and publish to kafka with outbox for durability.


#### Architecture
![[Pasted image 20260618162057.png]]

Client can submit bid and view highest bid.
Bid Engine can be shared per auctionId.
We ensure bids in kafka before returning status back to client.
Auction state is a filtered queue of highest bids. So downstream can subscribe to a server that reads from the auction state.


---

### My attempt




