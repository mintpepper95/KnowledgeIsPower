
[[[#DDD Theory]]]
[[#Practical example in ASP.NET]]


#### DDD Theory

For designing complex microservices ( independently deployable services ) with significant business rules. Simpler responsibilities, like CRUD can be managed with simpler approaches.

We need to first identify the subdomains. ideally, each subdomain is mapped to a single bounded context. A bounded context is a logical boundary in which the terms are consistent.

**Bounded Context** - Eg. For e-commerce site, we have order management, customer management, stock management, delivery management, payment management, product management etc...

A bounded context can include many aggregate, or a single aggregate.

**Ubiquitous language** - used in a boundary context, meaning all microservices etc uses the same language. We use this language to build a model of our domain. Example, host in the listing domain of airbnb, refers to a person renting their homes.

**Value object** - an immutable object that represents a specific value. E.g. price, or rgb value, they are considered equal if they have the same value, there's no identity. They also help express business rules.

**Entity** - they may have the same values, but are regarded as different instances. They are only equal if they have the same identity. They are mutable.

**Domain events** - events with data that notify other services when something happens. Eg. buying a book, user is logged on etc.

**Aggregate** - A cluster that contains value objects, entities and domain events. Eg. Customer place orders to buy books, then customer, order and book can be treated as an aggregate. Aggregates must be ref by a main entity, which is the root entity.

Aggregate live in the write model.




Aggregate root - To ensure aggregate is always in a valid state. It's also what's visible to the outside. 


We think about the aggregate as a whole. All calls from outside will ref the aggregate root, they can't ref the other entities within this aggregate.

Aggregate define a consistency boundary where states of all entities must be valid according to the business rules.

To ensure state changes are consistent, we must save all changes to an aggregate in 1 atomic data transaction. For reading, we load the whole thing from the repository. So our aggregate remains in a valid state.
![[Pasted image 20240320003601.png|680]]

It's responsible for aggregates are in valid states. Eg. Not more guests booked than homes available.

Aggregate is also transactional boundary, meaning changes to it are either committed or rolled back as a whole in case any rules/operations fail.

In the context of event sourcing, each operation made on the aggregate should result in a new event.

We can think of an aggregate as a cluster of associated objects, it's a single unit for the purpose of data changes. Each aggregate has a root and a boundary. Root is a single, specific entity contained in the aggregate. It's the only member of the aggregate that outside objects are allowed to interact with.

Eg. Invoice is a domain object, it can be made up of several objects, amounts to be paid, vendor name, due date. Invoice joins them together. Invoice is the aggregate. The root aggregate is the unique invoice number.



Read model
When events are added to the event store/db as result of write model processing, corresponding projections are triggered to create/update the read model. 

Due to being derived from events, they can deleted/recreated without losing info. 


![[ES-CQRS-Sketch-4.svg]]











#### Practical example in ASP.NET
In this case, our microservice is divided into 3 layers.
Api - web api project, here we have the controllers and system configurations like DI.
Domain - class library, for business logics. DTOs, Interfaces and services that implements interfaces. Enums, constants etc as well.
Infra - class library, where we access the database. Eg. Db Context, Entities ( where each properties maps to a column ), repository interfaces and implementations.

![[domain-driven-design-microservice.png]]







CQRS ( Command Query Responsibility Pattern) -
Separation of command and query responsibilities.

Have separate interfaces that reads data to that updates data.
Data you read and write are stored in different database tools.

Command - Only way to change something in the system. Commands are sent to CommandHandler from CommandDispatcher. Command handler is typically responsible for one aggregate. They can also emit domain events instead of directly modifying other aggregates.

In Event Sourcing, each operation made on the aggregate should result with the new **event**. For example, issuing the invoice should result in the InvoiceIssued event. Once the event has happened, it is recorded in the event store.


Query - Only way to read data from system. Querys are sent to QueryHandler from QueryDispatcher.


Eventual consistence
The models maybe inconsistent for a while due to result of updating. But this situation will be resolved and system will eventually be consistent.

With CQRS, updating/writing are transmitted to services that writes the model. Once requests are processed, the changes are made to the reading model.

When user queries data, the services answer requests with the reading model.

CQRS is updating reading model with async processes after writing model is registered. Which is what eventual consistency is about.

There are no transactional dependency in eventual consistent systems, therefore read and write don't wait for each other, this is performance.






Event Sourcing
Accumulating events that took place.
With event sourcing, each change that affects entity is recorded.
When query is sent from the client, the system combines existing event info to provide the latest data.

Generating the data will be more slower due to this process compared to where data is readily available.

![[1 CAfs0qB6Ap4P10UqWsD7rw.webp]]


We can solve Event sourcing problem with CQRS.

![[1 Y0Kml0LWJvp4Tcm83DyCfw.webp]]

Client starts the operation to change to state of entity on the system. Event info is generated as result of this operation, and it's stored in the system as a model for writing. Event sourcing is for writing.

Our process is triggered and reading model is updated based on our writing model.


