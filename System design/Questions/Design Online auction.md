From Hello Interview
This guy says uh a lot, reduce how often you say it, and replace it with let me think.
Also always mention approaches and trade offs.


Instagram Auction Post
 - ability to auction it off on IG
 - auction has fixed end time - highest bid wins


In this video, we first started with analysing requirements.


---
#### Functional Requirements
1. Users can create auction post (clarified just sell one item)
2. Users can submit bids
3. Determine and notify auction winner at end
4. Users viewing items should see bids coming in (real-time updates)

A question we could also ask here is do we want to persist non highest bid.

##### Non-Functional Requirements
1. Strong consistency for bidding (This guy is very vague, should be strong consistency for bidding and determining winner, and high availability over consistency for auction creation)
2. Scalability (Asked for scales, interviewer mentioned mils of auction bids, and also most users will just view and not bid, so much more reads than writes)
3. Low latency for bid updates
4. This guys missed fault-tolerance: fault-tolerance is very important for bidding, we don't wanna drop bids


---
#### Core entities
- User
- Auction
- Auction Item
- Bid

---

API (inferring from functional requirements)
* POST auction, mentions auth through header via JWT
* POST bid
* Mentioned that determining Auction winner is a server side thing, so no endpoint needed
* GET auction

This guy missed GET APIs for live subscription

---

* Note this guy missed talking about CDNs like Cloudflare often sits in front of API Gateway which handles global load balancing, CDN caching etc


#### Initial design

Client -> API Gateway (authenticate, rate limiting, request/response transformation, routing to different services) -> Auction Service -> DB  <- cron job runs every min figuring out winner 

* I think using cron job every min is a bad approach, should push events onto message queues, and have background services handle them.
* This guy so far also missed out CQRS - separating reads and writes.


![[Pasted image 20260619155732.png]]

#### WebSocket vs SSE vs polling for notify users and update bid prices


#### How to determine winner (for both fixed time and variable time)

You need to schedule something that needs to happen at a time is very common in SD.

##### Cron is not good
With a cron job, every few seconds, query `SELECT * FROM auctions WHERE ends_at <= NOW() AND status = 'open'`. Works but hammers the DB, has latency proportional to poll interval, and doesn't scale.


```md
Bid arrives
    │
    ▼
Validate & write bid ──► Redis ZADD auction:{id}:bids <amount> <userId>
    │
    ▼
Fixed end?          Variable end?
    │                   │
Schedule job        Extend soft_deadline
at ends_at          Cancel old job → enqueue new job at new deadline
    │                   │
    └──────┬────────────┘
           ▼
    Job fires at deadline
           │
     ZREVRANGE 0 0  ← winner
     Atomic status flip (CAS / DB transaction)
     Charge + notify
           │
    Reconciliation cron (safety net, 60s)
```

Consider Idempotency, also bid writing to redis and db must be atomic or else need some reconciliation

Can fall back to cron (as a fallback, not as a replacement)

Dead letter queues for observability


### Azure Service Bus (and equivalents)

However unlike Kafka, Azure Service Bus does not guarantee ordering, but we can cancel scheduled messages.

|Feature|Azure Service Bus|AWS SQS|RabbitMQ|
|---|---|---|---|
|Native scheduled delivery|✅ `ScheduledEnqueueTimeUtc`|⚠️ 15 min max delay|⚠️ via plugins only|
|Message TTL / DLQ|✅|✅|✅|
|Max schedule horizon|7 days|15 minutes|N/A|
|At-least-once guarantee|✅|✅|✅|
|Cancellable scheduled msg|✅ (sequence number)|❌|❌|




#### Fault tolerance and misc
This guy ran out of time.

We can talk about replication here.




