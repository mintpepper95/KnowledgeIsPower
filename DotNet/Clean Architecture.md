Based on ASP.NET Core 3.0 - Jason Taylor - NDC Sydney 2019

Domain Driven Design

![[1_YVxaXqiIYskqPauNKZ2OSA.png|600]]


![[domain-driven-design-microservice 1.png]]


#### Domain layer

![[Pasted image 20240711185620.png]]

In your entity classes (which lives in domain layer), a good practice is make sure you initialise all collections `ICollection<OrderDetails>`, and set the setter to private. This way you can avoid null check or assigning it to null. 

Avoid using data annotations for entities in the domain layer. 

Think if something is really a primitive type or something more complex, for example postcode with certain rules.

#### Application layer
Web Api Endpoints
Interfaces
Models
Logic
Command/Queries
Validators
Exceptions


CQRS - separate reads (queries) from writes (commands), eg. MediatR for simplicity and easy to maintain.

By using MediatR, application layer just becomes a series of request/response.

For example.

![[Pasted image 20240711191433.png]]
![[Pasted image 20240711191346.png]]

If we look at a particular command/query handler, we will see all the dependencies are on interfaces, making things flexible.

Use fluid validation instead of data annotations.



