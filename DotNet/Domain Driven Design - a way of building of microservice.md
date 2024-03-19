
For designing complex microservices with significant business rules. Simpler responsibilities, like CRUD can be managed with simpler approaches.


We need to first identify the subdomains. ideally, each subdomain is mapped to a single bounded context. A bounded context is a logical boundary in which the terms are consistent.

Ubiquitous language - used in a boundary context, meaning all the microservices etc uses the same language

Example, host in the listing domain of airbnb, refers to a person renting their homes.


Aggregate- eg. Reservation Aggregate in Airbnb might be Guest entity. Host entity, and price value objects ( the pricing ) The Reservation Aggregate is at the root so it's the root aggregate. It's responsible for aggregates are in valid states. Eg. Not more guests booked than homes available.

Aggregate is also transactional boundary, meaning changes to it are either commited or rolled back as a whole.







Bounded Context - Eg. For e-commerce site, we have order management, customer management, stock management, delivery management, payment management, product management etc...


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

Command - Only way to change something in the system. Commands are sent to CommandHandler from CommandDispatcher.

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


