Also known as pub/sub.

At least 3 components
* Producer - emits events to a broker
* Broker - Redirected the emitted events to the consumer
* Consumer - Receives the events and execute.

Advantages
 * decouple the different components
 * dependency inversion 
 * scalability
 * replacibility - you can easily replace a service, just move all the old service events data into the new service and make new service subscribe to those events

Eg. Service 1 wants to communicate to Service 2. For direct communication Service 1 must know about Service 2 (coupling). In event driven architecture, Service 1 would just send an event. Service 2 only needs to know about the event for process (dependency inversion). Services don't need to know about each other, they just produce and consume events.

Events are immutable. An event maybe consumed by many different services. Events can be persisted.

Availability - There is also no single point of failure. For direct communications, both Service 1 and 2 needs to be active. In event driven systems, the events are stored in a broker until Service 2 reads it.

###### Cons
The introduction of a broker may degrades performance.
Eventual consistency - there is a delay between sending it and processing it. Especially when you have different services consumes the same event, they can be out of sync.

If response depending on time, then replaying events in a service may get us different responses. So we need to store timestamps.

Also no granular control, in direct communication, we can specify how much time the request takes and the exact services we send it to. But with event bus, we don't exactly know when an event will be processed by the different services.

Difficult to reason about flow of events.
Eg. We can see service 1 publishes an event to event bus easily. But then not easy, we need to find out the subscribers of that event.



###### When to use event driven architecture
When we don't want tight coupling,
When scalability is more important than performance
When we need this info in multiple systems. So all the services that need the data don't have to call the same database. 

Eg. user on a website clicks a buy button. System triggers an purchased event. This event can be processed by different systems, eg. Inventory serivce/Recommendation service. So we don't expect a response from the UI, we are just sending the events and any system interested in it can subscribe to it.

Alternatively we may want to confirm the payment before shipping the product. Can still do with event driven system. The user clicks buy button. Emits an event to confirm payment. Payment service will read it and process it and send response back to the client.



Each services store the incoming events data into its own database.
