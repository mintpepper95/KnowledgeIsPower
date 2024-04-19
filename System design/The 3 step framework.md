[[#Describe the 3 step framework in terms of inputs and outputs of each step]]





---


When reasoning about system designs, it can usually be broken down into 3 steps
1. Requirements
2. Data types, access patterns and scale
3. Design

#### Describe the 3 step framework in terms of inputs and outputs of each step

1. Requirements
	Inputs: Problem statement by the interviewer
	Outputs: List of functional and non-functional requirements
	
2. Data Types, API and Scale
	 Inputs: Function and non-functional requirements and problem statement
	 Outputs: List of Data Types we need, access pattern (API) for those data types. Scale of the data and requests the system needs to serve.

3. Design
	 Inputs: All of the outputs from above steps.
	 Output: Data storage. Microservices.



#### Step 1. Requirements
Important to understand what's needs to be built, before diving right into design.
Also problem is vague in general, so if interviewer mentions something specific like `you have 30 days to design a system`, then the mentioning of 30 days is important. When in doubt, always ask.

##### Functional Requirements
The core feature/use cases the system needs to support. Focus on what, what do we want to build?


###### Identify main business objects and their relations
E.g. In the case of Twitter, there are two main objects of interest, Accounts and Tweets.
Now think about relationship between accounts and tweets.
![[Pasted image 20240417190718.png|500]]

Then dive deeper into each object, like what makes up a tweet? Can it contain media?

###### Think about access patterns

Given object A, get all related object B.

E.g. Twitter

Given an account
	Get all its followers (Account to Account)
	Get all accounts it follows (Account to Account)
	Get all its tweets (Account to Tweet)
	Get a curated feed of tweets (Account to Tweet)
Given a tweet
	 Get all accounts that liked it (Tweet to Account)
	 Get all accounts that re-tweeted it (Tweet to Account)

You could ask, should we be able to get all accounts that liked a tweet? Or is it enough to just store the likes as a number?

Another thing you could ask, does ranking matters? E.g. Ranking matters for curated feed of tweets. Or we want to display them in chronological time order?

###### Consider mutability
Can the objects be mutated, or are they immutable?
Can we delete the objects?

E.g. Can tweets be edited after they are published? What happens to tweets if account is deleted? 

The reason we ask about mutability is because they can limit our ability to use caching in our design.


###### Example - Design Tiktok

If you are not familiar with something, start by asking high level overview.

1. First we identify the main objects and their relations.

Accounts, can post, like, comment on Video
Accounts, can follow other Accounts

2. What info do the objects hold?
Account: username, description.
Video: description

Are these info mutable?
Can videos be changed after uploading? Maybe the interviewer says no, once uploaded a video will stay immutable.

3. Think about access patterns

Given a user, get all videos they liked
Given a user, get their feed (videos by accounts they follow)
Given a video, get likes and comments

Post a video, follow an account, like/comment on a video.



###### Example 2 - Design a code deployment system aimed for devs in a company. They should be able to tag a release, and system will package it to some servers

Object: A single object in this case, an artifact, which contains product name, version and commit hash.

Triggering a release
This is an example of problem that's less intensive on requirements gathering.

In such cases, you can try and ask for how users will use system beyond its main purpose. E.g. Can a dev use this system to rollback? Do we want to show all the releases and their deployment status? Should we alert about failures?












##### Non-functional Requirements (NFR)

Quality attributes specify how the system should perform a certain function.
Most common ones are performance, availability, security and consistency.

NFRs help us to know what we need to optimise.

Why bother gathering NFRs?
Of course we should make a system to be performant, available, security and consistency and so on. One reason we want to gather non-functional requirements is we can clarify to relax certain requirements. E.g. It's okay for the xxx system to not be consistent (e.g. ok for some TikTok users to access certain videos later than some other ones)

Performance
It makes sense to optimise for performance when we have sync user facing workflows. That is, user expecting an immediate response from the system. We want to optimise for sync workflows that are accessed frequently.

E.g. User feed should show immediately


Availability 
How much downtime system can tolerate. 

In case of Twitter, the need for high availability is a must. System should not be down in anytime. So we want inconsistent rather than unavailable.

In other cases, hit on availability is okay where consistency is very important. E.g. A banking system, where operations need to be transactional. It maybe acceptable if our system is unavailable for small periods as long as it's consistent.

Security
Here we want to know if there's some workflow that might require us to account for security.


###### Example of NFR - Tiktok
After discussing with interviewer, we may find out we want to optimise for performance and availability.

###### Example of NFR - Code Deployment system
Performance - Maybe we want code artifacts to be deployed within x hours after being published.

Availability - System should be high available, otherwise would affect rollout of new releases. However, some downtime here and there is okay (e.g. on weekends)

Security - Maybe not so important in this case.











##### Data Types, API and Scale
Now we have the functional and non-functional requirements, we can ask the following questions.

What data types the system needs?
What does API look like?
What volumes of requests do we support?


###### What data types does the system needs?

There are two types of data we may need
1. Structured data - accounts, tweets, likes
2. Media and blobs - images, videos etc

###### What does the API look like
Most time user will interact system through  HTTPS requests, so think that.

E.g. In case of Twitter

![[Pasted image 20240418114327.png|500]]

###### What volume of requests do we need to support?
Is the system read heavy or write heavy?

Which endpoints are likely to be called more frequently?

In the case of Twitter, it would be read heavy.

If you are not sure, you can ask interviewer something like what's the behaviour of a typical user using this app? Or what does distribution of requests look like?




In the case of code deployment system.

![[Pasted image 20240418114744.png|500]]
Some good questions to ask about.
How many machines do we deploy the artifacts to?
How many artifacts do we expect to deploy daily?


##### Design
Once we know our cases and what to optimise for. We can design.
Want speed? Use a cache. Want availability? Put some redundancy>

Here we will focus about storage and microservices.

Storage - We know the data we want to store, where do we store it?

Microservices - How do we store and retrieve our data to give it to the API?

![[Pasted image 20240418115142.png|500]]

###### Data Storage

Blob storage
If you have any media/blobs to store. We can store them in blob storage, E.g, Amazon S3. In the case of twitter, media from tweets can be stored in a blob storage.

We may also want to couple the blob storage with a CDN.


Database

Relational vs Non-relational

![[sql-vs-no-sql.webp|500]]

We can also tell the interviewer about potential downsides of the one we pick.
If we picked relational, we can say the database will have a more rigid structure, so harder to incorporate changes, and we also need to scale up vertically, rather than dividing the work over more servers.

If we picked non-relational, we can say that we can scale horizontally but at the cost of not having ACID guarantees. So assuming no need for strong consistency.

In the case of Twitter, we probably don't need strong consistency. It's fine there's a delay for some users to see the tweets, same for seeing likes and followers. Eventual consistency is fine.

Twitter actually moved from MySQL to NoSQL seeking better scalability and availability. NoSQL typically have better availability.


Example - Design a bank system
Strong consistency is needed. Needs ACID guarantees, so pick SQL.

Example - Design a system to help doctors diagnose potential illness given symptoms
This is mainly a querying system. Data will be unstructured. So pick NoSQL, as we are fine with eventual consistency.

Example - Amazon
We may want both. We want consistency for product transactions, while flexible about it for product catalog. So we can use SQL for purchase and stock, NoSQL for product catalog.


###### Entities to store
This is where we design our database schema

Accounts: id, name, surname
Tweets: id, content, author_id, media_url

Be mindful of get all access patterns, these usually need to be guarded by paging. Also it maybe very expensive.

###### Microservices

Caching - are there any access patterns that would benefit from caching results in-memory. Note not all systems require caching.

Only consider cache when all 3 below is true
* Computation is costly
* Result doesn't change often
* Objects we are caching are read often

Redis is a way to cache parts of your db in memory for faster access.
Just say add a cache if you don't know Redis.

Caching's downsides includes introducing replication of data, which may introduce inconsistencies as cache maybe out of date. 


Load balancing
Helps us scale horizontally and maintain high availability.

2 benefits
* Horizontal scaling - we can add more servers to handle more load
* High availability - Available even if some servers go down due to fault or maintenance


##### End-to-end example
Design interview.io

###### Step 1: Requirements

2 main business objects - users and interviews.

We can start asking questions. E.g. Do we need to save video recordings? How do we match interviewers and interviewees? 

From interviewer's answer, we need to track the following info.

User (name, surname, pseudo name, availability)
Interview (interviewer, interviewee, video recording, programming language)
Booking (time, interviewer, interviewee)

Now let's talk about access patterns.
Given a user, get all interviews they took part in.
See showcased interviews.
Set availability.
Book interview.
Join interview.

Non-function requirements
Availability as platform needs to be up for candidates to do interviews.
We might also want good audio quality as a non-functional requirement.

For code execution, we want to execute code in isolation. Candidate submission should have no side effects on the system.

###### Step 2: Data Types, API and scale

Data Types
![[Pasted image 20240418172732.png|500]]

REST API
![[Pasted image 20240418172755.png|500]]

Scale
Ask how many interviews do we expect a day, how long is an interview. What do we need to store as part of recording (e.g. just audio and coder pad)

###### Step3: Design
Now we talk about where we want to store the data and how.
Maybe a table?



---
##### Practical examples

###### Paste bin

1. Functional Requirements

User can add a Paste: send in the text and get back a short, unique url (id)
A user can view a Paste knowing its url
Author of a Paste can delete it.
Author should be able to see all the Pastes they have created.
No editing Paste after creating.

2. Non-functional Requirements
Availability - service should be up 24/7 ideally. 
Consistency - Pastes should not change/disappear. They should only be removed by owner or the platform.
Durability - If parts of system go down, we need to try to preserve contents of Paste. By that we mean we don't want to store all Pastes on one server/single db in a single geographical location.
Scalability: System should not break under heavy load.
Latency: Retrieving a Paste by ID should be relatively fast.

Once you think you've gathered all the requirements, you can check with the interviewer by asking something like 'Are there any requirements I have overlooked' to confirm you have all the requirements.

In most cases, like here where you have an action for each functional requirement. We can simply use REST API. 

However, there are some cases where REST API are not suitable
* When real time responses /streaming are required.
* When one request can conflict with many others, eg. Ticketmaster, where a large number of users could be competing for same resource.



Common ways to ensure a system is highly available include:

- Load balancers and CDN to less load on servers
- Investing in redundancy by leveraging **database replication, regions, and availability zones**
- Protecting the system from failures and atypical behaviours by utilizing **circuit breakers, rate limiting, load shedding, and retries**


To make sure the system is durable, we should be able to migrate service to a new machine without losing vital data. We can achieve this via
* Backups
* Storing data across multiple disks rather than on a single disk*
* Quickly replicating data to other servers in related systems for extra redundancy, so if your entire system goes down, you'll still have copies of your data in other systems.


Deciding between strong consistency and eventual consistency
Ask yourself, would it be fine if data in my system was occasionally wrong for a split second or so?

ACID talks about transaction guarantees without context of database.
CAP is about trade-offs in distributed systems, mostly around nodes always being the latest values and system always being available.


E.g. A chat app

User A sends a message to our servers, we have it hit on the load balancer first to avoid make sure servers aren't overloaded.

When a server receives a message, they will store data in the database, which is also behind a load balancer, as we don't want to overload the database.

User B can access messages by reading from servers through load balancer.

![[aol-desktop-diagram.webp]]

How does User B know it's time to receive data?

###### Short Polling, Long Polling and WebSockets
Are we there yet?

Short Polling
Wait for a set interval and keep asking a server until a response is received. Eg, weather widget updates every x minute.

Long Polling
Server holds the line until it has something to share, avoiding constant check-ins. Used when real time updates matter, and we want to minimise unnecessary requests.

It's however resource intensive.

WebSockets
With WebSockets, we can have arbitrary lengths of time pass without needing to worry about connection timeout like long polling. It enables real time communication between clients and servers without the need for continuous polling.

###### What is a CDN? How is it different to load balancers? Which sits first?
CDN - geographically distributed group of servers that caches static content close to end users. They do not host content. CDN like CloudFront sits in front of everything, if its cache misses  then you go to load balancer which will direct your request to the origin server, else served from cache.

The TTL (Time to Live) value determines how long content should be stored in a  cache before it's considered stale and needs to be fetched again from origin server.
CDN used to improve website performance for a global audience.
While load balancers are used to handle high traffic website.


###### Db caching
For anything dynamic, better having multiple servers behind a load balancer. Note if all those servers interact with the same db, there will still be a bottleneck.

In this case, we can apply caching (using a cache like redis to reduce number of times we need to query from the db) or db sharding. 

E.g. In the case of insta, does it make sense to SELECT COUNT* how many people liked a photo? No, you just increment counter by 1 for each like. So when you like a post, it increments count in cache and then does a fire and forget to request db to record the mutation.




##### Design Ticketmaster

System should be have consistency under concurrency (users fighting for 1 seat in a concert). Note very read intensive.

We always start by asking about requirements.

Functional Requirements
User can book tickets for a specific show.
User can book more than 1 ticket at same time.
User can book specific seats, or request N seats together.
Users are given x time after selecting seats to pay for them.
During this time, seats are reserved, other users should see these seats as unavailable.
The seats will return to the pool if user aborts the transaction or time expires.
Shows may offer zones where N tickets can be sold, e.g. a dance floor.

Non-functional Requirements
Consistency - can't sell same seat twice, return seats to pool quickly.

Responsiveness - Seat map should reflect actual occupancy at all times.

Availability - Not lose data on tickets sold. Eventual consistency, if things break then user will see data that's slightly stale, but they won't notice system is down.

In the case of Ticketmaster, we aim for our system to be horizontally scalable. E.g. having separate replicated DB and servers to handle the more important shows.
In terms of CAP, this basically eliminates P part.

E.g. Have replicas for our databases. If one db goes down, we just spin up the same service from a backup db shard. E.g., leader-follower replication. Leader goes down, then a follower is promoted to the leader.

![[markov-diagram.webp]]



Another example of designing the same Ticketmaster system

![[1 _Z-hvLz3d15IgOOCwn_zkA.webp]]


















