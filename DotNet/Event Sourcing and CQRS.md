#### Event Sourcing
Storing data as immutable events, instead of readily available data. We don't update and delete data. Best for modelling something that changes overtime. By storing changes as individual events we can observe changes that we can't see with traditional storage.

We store the events into a table in a db. When we need to query for the current state, we run a query that returns and processes the events to get the current state.

Event sourcing needs to start from a certain snapshot and then work through all the events to derive the current state.

Advantages:
* Immutable, they don't change and simply accumulate, allows us to look back and see what happened
* Replayability, eg. if there is a bug on interest calculation, we can simply rewind back to a point before bug was surfaced and replay the old events
Cons
* Eventual consistency



#### CQRS - for implementing event sourcing

Because it separates read and write, you can scale each part independently depending on needs. You can have separate models for read & write.

However it introduces complexity. So use only when necessary.



##### Reading event sourced data
1. App queries data.
2. Events are returned from db.
3. Returned events are summarized into current state.

Potential problem - If we have a lots of events, summarizing will be slow.
A solution to do this problem, is do the summarizing when data is written, not when it's read - CQRS

CQRS has two sides, a write side and a read side.


![[Pasted image 20240319221850.png ]]

We segregate reading from writing, we get much higher performance, as write and read are de-coupled. Trade off is the system becomes eventually consistent. As not all services will read the events at the same time.

A read might not be possible immediately after a write.

Eg. We write to Kafka Topic, creating a log of events. This is much faster compared to writing to db. The events in the Topic are pushed to the view on the read side where they can be queried. The summarization happens before data is inserted into the view.
Here the reduce/summarization is done at write time, not read time.



Event storage
Event sourcing is harder to implement.
Compared to a traditional system with only app and db.

With event sourcing, there is at least one more part.
Eg. A kafka Topic that separates the write side from the read side.

As app evolves overtime, the data model may change. This leads to events of different versions of the schema stored inside the Topic. This means we need to introduce more code to understand them.


Eg. A flight booking system is very read heavy. Consistency is only required at booking time.