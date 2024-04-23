
[[#APIs (REST OR RPC)]]
[[#Db (SQL vs NoSQL)]]
[[#Scaling]]
[[#CAP theorem]]
[[#Web auth and basic security]]
[[#Load balancers]]
[[#Caching]]
[[#Message queues]]
[[#Indexing]]
[[#Failovers]]
[[#Replication]]
[[#Consistent hashing]]
	What is consistent hashing? Example to achieve consistent hashing when servers are dynamic?


---

#### Some general tips
In System design interviews, you are playing the role of a tech lead. System design prompts tend to be light on detail. There should be a back and forth convo about problem constraints and parameters, so avoid making assumptions. Remember there is no RIGHT way to design a system.

A common failure point occurs when candidate don't make decisions. Candidates will say we could use this or that, and these are the pros and cons... **We also need to say, based on all the trade-offs, we will use that.** We need to make decisions and explain why.

Say generic name of a component, eg. a message queue, and not something specific like Kafka, unless you know how it works.
##### What to say when you don't know something
E.g. You are asked about loading balancers. 
You only know very basics.
You can say things like 
* I don't have experience with that, but I would approach it like xxx*
* I don't have any hands on experience, but here's what I know
* This reminds me of...
* This is super interesting, tell me more

##### Push back against interviewer
E.g. Interviewer says I wouldn't use a cache. You can say, sure, we don't have to use it. I think it's useful because xxx, so why not, do you think there's a better way to approach this. Always acknowledge and affirm your interviewer by starting with words like sure, ok and yes. You can then say I'm sorry, but if we don't use cache wouldn't that result in xxx. Use collaborative languages like we or let's. 

It's more important to cover everything broadly than it is to explain things in detail. You can 'handwave' some detailed stuff to avoid getting derailed.

A lot of candidate fail because they expect 'warm' interviewers and don't know how to adjust when they get a cold interviewer. Practice by doing mocks.

Cold interviewers get turned off when you constantly check in with them by asking questions. They would prefer you to proceed without involving them much.

You can try say things like 'correct me if I'm wrong, but I think we can do xxx because of yyy'. 'Stop me if I'm going off track, I think the next thing to do is X because of Y.'



### 12 fundamental system design concepts

#### APIs (REST OR RPC)
Set of rules or contracts through two entities communicate. Think Web APIs (REST), or Canva WebX APIs (RPC).

When we design API, we need to explore the architectures, review their strengths and explain why one is more suitable. There are primarily two styles, REST and RPC.

##### REST
HTTP Verbs
* Uniform interface, provides a structured way of getting and modifying info from your db.
* Client only needs to know the resource urls and what actions the endpoint supports. Meaning client and server can evolve separately.
* Stateless, each request must contain all info necessary to be understood by the server, rather than the server remembering prior requests. Stateless helps in scaling API to millions of concurrent users. Also less complex and easier to cache.
* Caching can be applied to resources.

Weakness - needs to write request for each type of entity, in contrast to things like GraphQL where no separate inquires are required to grab all the data. Also not as space efficient as RPC.


##### Remote Procedure Call
Calling a procedure/command in a remote machine. API is thought of as an action or command.

* Can be stateless or stateful.

Weakness - Tighter coupling between client and server compared to REST, where clients interact with servers through a standardised set of operations.

In RPC, client uses HTTP POST to call a specific function by name. Client side dev must know the function name and parameters in advance for RPC to work.

![[Pasted image 20240325224257.png|400]]

Strength - More space efficient than REST. Used for triggering a remote process on the server. 

Weakness - Can only be used for internal communication. 

More suited for internal communication, as RPC won't work for external entities to engage with your service.

Unlike REST, where you can pass in data format  like JSON and XML within same API, in RPC data format is selected by server and fixed during implementation. Client has no flexibility.


##### GraphQL
Query language that defines how client should request data from server.
Good for large, complex and interrelated data sources.

GraphQL can be a better choice, when
 * client requests vary significantly, and you expect different responses (complex data querying)
 * you have multiple data sources, want to combine them at one point
 * limited bandwidth, you want to minimise number of requests and responses


Strength - Works well for customer facing web and mobile, frontend engineers can create requests without requiring backend to build more routes.

Weakness - Initially upfront dev work to use this. Also less friendly for external users compared with REST. Not suitable for use cases where certain data needs to be aggregated on the backend.



##### Db (SQL vs NoSQL)

![[differences-between-sql-databases-and-nosql-databases.webp|650]]

SQL is a relational db composed of tables. Very specific and powerful query but Does not work well for mixed schema data. NoSQL is unstructured or semi structured. More flexible when it comes to changing your schema, or storing new fields on the go.

SQL db have predefined schema, NoSQL db have dynamic schemas as data is unstructured. And as such SQL is suitable for data that are consistent with well defined relationships. Where noSQL is more suitable for data with no predefined schema and relationships between data elements are not well defined.

SQL is vertically scalable, you can increase load on a server by giving it better hardware.
NoSQL is horizontally scalable. Eg. via sharding.

SQL is better suited for complex data quries and transactional support, while NoSQL is suited if you need fast and scalable db.

Typically SQL has slower writing than NoSQL db due to the way data is stored. NoSQL typically have slower reads.

###### ACID
SQL ensures ACID.
1. Atomicity - transaction completes in its entirety without failure, else transaction is reverted. No case where only half the writes are applied.
2. Consistency - SQL has strong consistency, achieved by locking certain parts of db when it's been written, forcing other requests to wait. NoSQL have eventual consistency by default, meaning your query maybe stale before it becomes consistent. This means SQL will have much higher latency. Customer experience might not require strong consistency.
3. Isolation - All read and write requests within a transaction are not impacted by other transactions. So executing transactions concurrently has same results as executing them serially.
4. Durability - In case of failure of a given transaction, data is not lost and can be recovered.


![[store-data-flowchart.webp|700]]




##### Scaling

First let's talk about performance vs scalability.

Performance problem - your system is slow for a single user
Scalability problem - your system is fast for a single user, but slow under heavy load

Latency - time required to perform some action
Throughput - number of actions we can execute  per unit of time



Two major ways to scale. 
Vertical scaling is making server more powerful so it can handle more loads. 
Reduces latency (how long it takes a packet to travel from source to destination) as different parts of your architecture is done locally on the same pc rather than through a network.
There is a diminishing return to better hardware.


Horizontal scaling is adding more servers.
There are two major forms of horizontal scaling.

* Database scaling
	Database sharding - where you split a large dataset into separate data so they can be stored into separate db. With db sharding, every time you store something, there's a hash function which determines which db to store it.


* Compute scaling
	Eg. Divide a video into pieces and designating each piece as a separate job, so multiple servers can work together in parallel.

In interview they will ask how to scale to a larger number of users. Do not assume a particular scaling strategy is required, instead try and understand the number of users and the amount of data that is required to make a choice.



##### CAP theorem
Consistency, Availability, Partition tolerance.

CAP is about trade-offs between the above when designing a distributed system (interconnected nodes that share data)
* Consistency means every node in the network will have access to the same data, so we will always get the latest data
* Availability means system is always available to the users, even if some nodes go down.
* Partition tolerance means in case of fault in network/communication, e,g, some nodes can't communicate with each other, system will still work.

###### Trade-off
CA (consistency and availability) - Data is consistent between all nodes as long as all nodes online. However if partition then breaks.

CP (Consistency and partition tolerance) - Data is consistent between all nodes, and has partition tolerance ( no de-sync ), however system becomes unavailable if nodes go down.

AP (Availability and partition tolerance) - Nodes remain online even if they can't communicate, so data is de-sync. However the system is available.

In most cases, fault tolerance is necessary. So choice is to decided whether consistency or availability is more important.

For systems like financial systems, consistency is very important. For others like TikTok, where it's okay for some users to get access to videos later, we try to aim availability.



##### Web auth and basic security
Question of security is about trade-off between total safety (a wall) vs total convenience (a hole in the wall).

Authentication - verifying identify of our users.

Login handlers should always use HTTPS to ensure credentials don't pass in plain text over the network where they could be intercepted when signing in.

###### Session token
A classic way to track authentication once user has logged in is to generate a token the user can submit with subsequent requests to make sure they are signed in ( think plenti JWT token ). Now you can check for this token in a cookie on subsequent requests to verify they are still the same user. Session token should come with an expiration date.

###### Cookies
Cookies is a way of storing the token on the client side and send it back to the server with each request. By storing session token or JWT in a cookie, we can ensure subsequent requests will include it and allow server to validate current user session.

We can set cookies to `Secure`, such that browser can only include it in HTTPS requests, and can not access the cookie through js.

###### Summary
1. User signs up. We hash the password and store the hashed values.
2. User logs in and we verify the hashed entry to the stored hash. We then send some kind of id token, like a JWT back to client with a Set Cookie header.
3. On subsequent requests, browser sends cookie back to server, where we can verify session token or check signature/decrypt a JWT.
4. Periodically, session token or JWT expires and new one is generates and sent to the client.
5. Eventually, user's session may expire from inactivity, we go back to step 2.



##### Load balancers
Helps direct requests to different servers to distribute incoming workload evenly among available servers. Helps with horizontal scaling, improving performance and availability,

###### Round robin
If there are N machines, load balancer will send request to each of them sequentially.

###### Least Connection/Response time
We send requests to machines with least connections or min response time. This is useful for incoming requests that have varying connection time and we have a set of servers are that relative similar in terms of computing resources.

###### Hashing
The key can be a request id given user's ip address. Hash will return the server id.


##### Caching
Used to reduce latency of expensive computation or network calls or db queries or asset fetching.

Trading off space for better performance.

Caching is used for read-heavy systems, e.g.. twitter and YouTube.

E.g. If there is a tweet from a very popular person, we would want to cache it everywhere ( on a device, on a CDN, on the app ), so that we can fetch it quickly for our users.

Client caching - caching on client side

CDN caching - A CDN is a something that's used to deliver cached data. Its purpose is to deliver web content, e.g. images, videos, assets to users quickly. They primarily handle static content that doesn't change frequently. CDNs store copies of content on distributed servers located in various places, when a user requests content, the CDN serves it from the nearest server in terms of geographic, minimising latency and reduce load on the original server.

Application Caching
Redis which sits between server and db.


###### Popular caching patterns

1. Cache aside pattern
	 Think memo in dp. App will try to fetch data from the cache, if cache miss (data not found), it will fetch from db/do expensive computation and then put that data back into the cache before returning the query. In this pattern we only cache the data we need. 
	 However if lots of updates, then data can go stale. Also if lots of cache misses, then additional work as app will need to check for a cache miss, and then write the computed data back to the cache before going back to server. This actually worsens latency.

2. Write-through and Write-back
	 App directly writes data to the cache. And then cache is sync or async writes the data to the db. When cache write it sync, it's called Write-through. When cache write it async, it's called Write-back.

In both patterns, we are writing all the data to the cache, which might not even be read.

###### Caching on client side
Eg. Caching thumbnails of Netflix recommendations in the device or browser. So when user visit the app again, it's quickly served from cache or local memory instead of fetching from the network.

###### Cache invalidation
We mentioned that a problem of caching is data can go stale if lots of updates to the db. Therefore it's important to expire or invalidate data from cache. Good practice to include a brief point about cache invalidation during interviews.

Many policies to invalidate data from the cache, most common one is LRU ( least recently used ). The idea is that we can have a timestamp to the cached data, and discard the one that's least recently used when the cache reaches its capacity.

Using LRU will result in fewer cache misses, as more commonly accessed cached data will be prioritised. We are assuming that recently accessed data will more likely to be accessed again.



##### Message queues
Queues can send messages to multiple systems, instead of client having to send them to all the systems by itself. It de-couples client from server. Also serve as a buffer between client and server.

In a queue, there are producers, message queue and consumers.

Based on different implementations of message queues, there can be different combinations of the following properties
* Guaranteed delivery
* No dupe messages
* Order of message is maintained
* At least once delivery with idempotent consumers (a consumer that can safely process the same message multiple times without dupe processing)


##### Indexing
Indexing is about mapping the underlying data for faster retrieval.


##### Failovers
One of the ways to support high availability.

It's about switching to a backup or redundant system component when primary component fails or becomes unavailable. The goal is to ensure high availability and reliability by minimising downtime.


##### Replication
Another complementary way to support high availability.

Have servers ready to take over when a node fails.

Useful for scaling relational DB

Replication is done to avoid a single point of failure and increase availability of the system, also to better serve global users by serving copies closest to the user by geological locations. And to increase throughput (bandwidth), such that more requests can be served.

Replica - copy of data
Leader - machines that handles write requests to data store
Follower - machines that are replicas of the leader node, and cater to read requests.

###### Sync vs async replication

Sync replication - when a write request to a replica is acknowledged, so leader waits for acknowledgement from followers. This can be nice when it's vital nothing is missed, downside is slows down process.

Async replication - when leader don't wait for acknowledgement from followers before marking client's write requests as successful.

###### Common types of replication systems
Single leader - a single machine acts as a leader, and all write requests ( updates to data store ) go through that machine. All other machines are for read requests.
Leader needs to pass down info about writes to follower nodes to keep them update to date. In case leader goes down, one of follower nodes is promoted to be a leader ( failover ).


Multi leader - More than one machine take write requests. Makes system more reliable if leader goes down. Every machines needs to catch up with writes on the leaders including leaders.


Leaderless replication - all machines can read and write.



##### Consistent hashing

Consistent hashing is a hashing technique used in distributed systems to efficiently distribute data across multiple nodes while allowing us to add/remove nodes without incurring a large performance hit. 

It is commonly used in distributed caching systems, distributed databases, and content delivery networks (CDNs) to achieve load balancing, fault tolerance, and scalability.



![[hashing-algorithm.webp|670]]
Consider this example where we have a cache key 1234. Normally we can just hash(key) % `n` where n is number of servers. But servers can be added/removed. We have to change `n`. And now we are getting results that are no longer consistent with our earlier hashed results because `n` has changed.

We would have to relocate the existing keys into our nodes following the updated hash. This can be very expensive. And particulary bad if your node went down due to excess traffic, as now you are trying to reallocate all the keys on top of dealing with the traffic.


![[pilot-hash.webp|550]]
Consistent hashing works as above. Instead of having fixed number of servers. We map our hash result to a ring. Pretend ring goes through 0 to 100. We assign each node to a point on this ring. When we write or read from this cache, we plot the hash position on the ring and then go clockwise til we find next node.



![[pilot-hash-3.webp|560]]
Now, we only have to reallocate the keys assigned to the down node.
Instead of putting a single point on the ring for each node, we can put a bunch of point for each node - virtual nodes. When one node gets knocked offline, we reallocate keys such they are even distributed among the entire system.

















