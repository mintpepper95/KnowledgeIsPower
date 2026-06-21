From hello interview


---

### FR
- User can book tickets
- User can view events
- User can search events
- After selecting a seat type, for x min the ticket is held for you where you choose a seat and complete the payment. If not done, we cancel the reservation.

### NFR
Here is where we think about CAP (consistency, availability, fault tolerance)
- Strong consistency for booking tickets
- High availability for searching and viewing events
- Reads >> Writes
- Scalability to handle popular events

---
### Core entities
- Event
- Venue
- Performer
- User
- Ticket

---

### High level design

![[Pasted image 20260621213246.png]]
Very basic design

Usually for cancelling reservation if the user has not completed the purchase, the naive approach would be cron job. However cron job runs every y min so there is a delay between when seat reservation becomes expired and its status in the db is updated.

We need something a bit more real-time.

We can introduce a distributive lock, like Redis TTL for ticket lock.
So after 10 min, the ticket reservation is deleted from redis.

So when we try retrieve seats for user, we can first query the db to get all available tickets, and then check redis to see if they are reserved.

What happens if this redis lock goes down? We would need to bring a new one up. However as a consequence we lose the reservation information. So seats that are 'reserved' are now available, means user might have reserved a seat, but maybe unable to complete the purchase because someone else bought it already.

---
### Deep Dives

Improve search - we can use a search database like ElasticSearch, it builds an inverted index (basically map the keywords to the documents containing them) which allows searching by term really quick. Note not best practice to use it as primary data store.

So we need to sync between our primary db and ElasticSearch.
Naive way is now our service write to two places, the primary db and ElasticSearch.
Another common solution is use CDC (Change Data Capture). So any time something in the db changes, event is pushed onto a stream and consumed to update ElasticSearch. If doing a lot of writes, have a queue to buffer, have batching to reduce traffic etc

ElasticSearch support caching of top used queries.
We can also cache with redis.
We can also cache in CDN. So cache popular api calls and results.
Less useful if your search has a lot of permutations to your query terms which would change the query. If even system has personalised recommendations, then this wouldn't suffice.

For popular queries we can cache them.


Scalability to handle popular events

If clients sit on seat selection for a long time, they could select a seat that was available but now no longer available.

So we need real time updates.
So any time a seat becomes reserved, we need to inform the client.
Discuss long polling, SSE and WebSocket.
Here we only need to inform client, so SSE is suitable.

We also want a buffer for popular events, don't want all traffic to hit our backend at same time. So we can use a virtual waiting queue (only enable for popular events, inform user they are in the queue for x min). We could implement some logic, once first 100 seats booked, bring in next batch of users from the queue.

We could also introduce sharding.

We could split out reads and writes. We can introduce CQRS or use redis.


Updated architecture

![[Pasted image 20260621215300.png]]


