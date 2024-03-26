
[[#APIs (REST OR RPC)]]


In System design interviews, you are playing the role of a tech lead. System design prompts tend to be light on detail. There should be a back and forth convo about problem constraints and parameters, so avoid making assumptions. Remember there is not RIGHT way to design a system.

A common failure point occurs when candidate don't make decisions. Candidates will say we could use this or that, and these are the pros and cons... **We also need to say, based on all the trade-offs, we will use that.** We need to make decisions and explain why.

Say generic name of a component, eg. a message queue, and not something specific like Kafka, unless you know how it works.
#### What to say when you don't know something
E.g. You are asked about loading balancers. 
You only know very basics.
You can say things like 
* I don't have experience with that, but I would approach it like xxx*
* I don't have any hands on experience, but here's what I know
* This reminds me of...
* This is super interesting, tell me more

#### Push back against interviewer
E.g. Interviewer says I wouldn't use a cache. You can say, sure, we don't have to use it. I think it's useful because xxx, so why not, do you think there's a better way to approach this. Always acknowledge and affirm your interviewer by starting with words like sure, ok and yes. You can then say I'm sorry, but if we don't use cache wouldn't that result in xxx. Use collaborative languages like we or let's. 

It's more important to cover everything broadly than it is to explain things in detail. You can 'handwave' some detailed stuff to avoid getting derailed.

A lot of candidate fail because they expect 'warm' interviewers and don't know how to adjust when they get a cold interviewer. Practice by doing mocks.

Cold interviewers get turned off when you constantly check in with them by asking questions. They would prefer you to proceed without involving them much.

You can try say things like 'correct me if I'm wrong, but I think we can do xxx becaue of yyy'. 'Stop me if I'm going off track, I think the next thing to do is X because of Y.'



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
3. Isolation - All read and write requests within a transaction are not impacted by other transactions.
4. Durability - In case of failure of a given transaction, data is not lost and can be recovered.


![[store-data-flowchart.webp|700]]





##### Scaling

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

CAP is about trade-offs between the above when designing a distributed system
* Consistency means every node in the network will have access to the same data
* Availability means system is always available to the users
* Partition tolerance means in case of fault in network/communication, system will still work

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



##### Caching


##### Message queues


##### Indexing


##### Failovers


##### Replication


##### Consistent hashing



Daniel petrucci 
Daniel Ko
Ask about career growth, oppoturnities
Infotrack awards, culture settleIT, what they are trying to














