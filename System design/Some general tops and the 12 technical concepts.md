

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

#### APIs
Set of rules or contracts through two entities communicate. Think Web APIs, or Canva WebX APIs.

When we design API, we need to explore the architectures, review their strengths and explain why one is more suitable.

##### REST
HTTP Verbs

Strength - most commonly used, structured way of getting and modifying info from your db.

Weakness - needs to write request for each type of entity, in contrast to things like GraphQL where no separate inquires are required to grab all the data. Also not as space efficient as RPC.


##### Remote Procedure Call

Calling a procedure/command in a remote machine. API is more thought oof as an action or command.

![[Pasted image 20240325224257.png|400]]

Strength - More space efficient than REST.

Weakness - Can only be used for internal communication.


##### GraphQL

Strength - Works well for customer facing web and mobile, frontend engineers can create requests without requiring backend to build more routes.

Weakness - Initially upfront dev work to use this. Also less friendly for external users compared with REST. Not suitable for use cases where certain data needs to be aggregated on the backend.



##### Db (SQL vs NoSQL)

SQL is a relational db composed of tables.
Very specific and powerful query.

SQL has stronger ACID guarantees out of the box.


Slower for writing than NoSQL db.

NoSQL typically have slower reads.

Does not work well for mixed schema data.


NoSQL is a nested key value store.

NoSQL do not have fixed schema.




##### Scaling

CAP theorem


Web auth and basic security



##### Load balancers



##### Caching


##### Message queues


##### Indexing


##### Failovers


##### Replication


##### Consistent hashing













