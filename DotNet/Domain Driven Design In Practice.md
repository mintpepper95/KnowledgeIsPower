[[#When to use DDD, explain what ubiquitous language and bounded context are]]
[[#How do Entities and Value Objects differ?]]


---
#### When to use DDD, explain what ubiquitous language and bounded context are
DDD is useful if the project you are working on has a lot of complex business rules.
DDD won't help you if you work with things like Twitter because the challenge comes from performance with big data.

DDD forms an onion structure, where inner layer should remain isolated and does not depend on outer layer.

This means Entities, Value Objects, Domain Events and Aggregates should only know about each other and not anything in the outer layer, they keep most of the business logics. In practice, this means they should NOT contain any logics about how they are created/persisted etc.

Similarly, repositories and factories and domain services should know about each other and the inner layer, but should not be aware of application services.
![[Pasted image 20240712160414.png|400]]

---
### Starting the first bounded context

#### Starting out the snack machine
We want to model a snack machine
- insert money into the machine
- return money back
- buy a snack

So we can start with a snack machine class, with it holding a bunch of properties, MoneyInTenCents, MoneyInTwentyCents .... MoneyInOneHundredDollarsNote etc

We also need to distinguish the money inside the snack machine from previous transactions and also the money inserted for the current transaction. So we need another set of above properties for in transaction money.

We can simplify our logic by extracting money related operations into a separate Money class. Now instead of a bunch of properties, we just have two Money properties, MoneyInsideMachine and MoneyCurrentTransaction. This greatly simplifies the code in snack machine class.

#### How do Entities and Value Objects differ?
At this point, we have two classes, Snack Machine and Money. Snack Machine is an entity, where Money is a value object.

How do they differ?
We have 3 types of equality.
* Identifier - equal if same id
* Reference equality - equal if same object
* Structural equality - equal if same structure, eg. amount

For entities, they are considered same if same id and same reference.
For value objects, they typically don't have id fields. They are considered same if same structural equality, in this case, if money amount is equal.

Imagining have two 1 dollar note, in our problem, they would be treated the same.
But we do care about distinguishing between 2 snack machines, even if they have same money inside and same inventory.

Value objects should be immutable, we should construct a new instance based on existing object.

Also in terms of life span, value objects can't live by their own. They should belong to one or several entities. In our case, money doesn't make sense without snack machine.

#### Recognising a value object
Whether something is a value object will depend on the domain.
If we need an app to track all money in circulation, then we would need to see two $5 notes differently, even if they have the same value, they would be entities.

If we can safely replace an instance with another one that has the same attributes, then it's a value object. Think of integers, all 5s are the same, there's no difference between a 5 you created in another part of the system with a 5 you created just now.

Ideally we want to put most business logic into value objects, and have entities as wrappers.

For value object, we usually want to define some basic abstract class called `ValueObject`, with some abstract/virtual methods perhaps we can implement in derived classes to check if two value objects are the same.

Value Object properties should be immutable, so private setter or just remove setter (readonly property), with parameterised constructor for setting properties initially.

However for value objects with a lot of parameters, it maybe somewhat annoying to pass all parameters when creating them when we want some default value object, we can have a static readonly field that returns a new instance of empty/initial Value Object if needed. We can also have other fields, like for One Dollar, One Cent etc.
`public static readonly Money DefaultMoney = new Money(0,0,0,0,0,0)`

Quick word on abstract vs virtual
Virtual methods can have default implementation ( you can override in derived classes ), abstract does not ( you have to override in derived classes ).


#### Value Object vs .NET Value Types
.NET Value Type serve as base class for int, byte, long and structs etc.

Strictly speaking, no relation between these two as one is a concept, the other is an implementation,

However there are some similarities, both immutable and have structural equality.
Note that structs in .NET does not support inheritance of classes, so may lead to dupe code. So we should use classes to implement Value Object.

Ideally we should have tests around value objects if needed.


### Introducing UI and Persistence Layers
UI is just adding an interface.

#### Adding persistence
![[Pasted image 20240712173743.png|400]]

How do we persist snack machine and money inside it?
While above diagram might look okay, it's not.
Money as a value object should not have an ID and that violates definition of Value Object.
Also this allows money object to live by its own. Cause we will be able to delete snack machine while Money still exist.

An ideal approach would look like below, just inline them into parent.
![[Pasted image 20240712174048.png|300]]

We would need an ORM to link the table with the actual entity.

### Extending bounded context with aggregates
We need to allow for buying different types of snacks.
Return extra money user has inserted.
Check if there's enough change in the snack machine.

This is what our domain model will look like
![[Pasted image 20240712175647.png]]
Slot and Snack are entities.

#### Aggregates
In our domain model, SnackMachine and Snack are Aggregates.

Aggregates basically gather related entities under a single Aggregate.
In our case, root entity is Snack Machine.
![[Pasted image 20240712180421.png|300]]
Aggregates can have invariants, eg. in the case snack machine, it could be that the weight of the snacks can't be over 10kg.

Entities should be high cohesive, if an entity makes sense without others, then it should probably be its own root aggregate.

Snack machine + slot => aggregate 1
Snack => aggregate 2

You want to make sure one root entity is not related to too many other entities, if so may want to consider extracting that entity out of the aggregate.

Classes outside that aggregate can only reference the root entity.
The idea is we want to help preserve the invariants in an aggregate by restricting access.

Eg. If we want to retrieve Slot, we would have to go to Snack Machine first.
![[Pasted image 20240712180455.png|400]]

![[Pasted image 20240712180715.png|400]]

We also want to save the aggregate in a transactional manner, eg. Saving both Snack Machine and slot, should not be possible to update slot but not snack machine.

While entity can belong to a single aggregate, value object can belong to multiple aggregates, remember entities are just wrappers to value objects.


![[Pasted image 20240712181634.png|400]]
We can of course omit the `AggregateRoot` base if it's not needed in your domain.

![[Pasted image 20240712181928.png]]
Note here we mark Slots as protected as they should ideally not be directly accessible. 
We can then have methods inside that class that extract info from Slots.

 Note for our Slot class, we can extract Snack, Quantity and Price into a Value Object since these 3 are mostly used together
Before
![[Pasted image 20240713001653.png]]

After
![[Pasted image 20240713001810.png]]

Our new Value Object SnackPile
Note when we update - subtract, we create a new Value Object rather than update

Quantity and Price can't be negative, Price can't be less than 1 cent, since we are using a decimal here.
![[Pasted image 20240713002230.png]]


#### Additional requirements
Implementing missing requirement
* inserted money is sufficient for buying a snack
* Snack pile should not be empty when buying
* Return the change - upon further analysis, this rule is more complex, in this case we want snack machine to maintain smaller denomination money as much as possible. Meaning that if a snack is $1, and user inserts, 4 * 25 cents AND $1, it should return $1. IE. It should return money with highest denomination
* We shouldn't allow purchase if we don't have enough change inside machine to return

To accommodate the new requirement, we just need to track how much Money user inserted, we don't care about the actual type, because we will return money with highest denomination.

![[Pasted image 20240713003324.png]]

### Repositories
Domain model
![[Pasted image 20240713004031.png]]

Database
![[Pasted image 20240713004139.png|500]]

We use repositories to persisting our entities.
There should be a repository per each aggregate.
In our case, there should be two of them because we have two aggregates.
SnackMachineRepository - so when we call the repo, it should include the slots as well
SnackRepository

#### Getting a sub entity
So if we want a sub entity, like for example a particular Slot. we can implement a method in the SnackMachineRepository that returns the *snack machine holding that Slot*. Persistence of sub entities should also be behind the scene.

### The second bounded context

New Task - ATM machine
This model allows user to withdraw cash with their bank cards

* ATM should dispense cash
* Charge from user bank card + fee
* Track money charged

Bounded contexts means separating the models into smaller pieces.
They define the ubiquitous language, meaning the language only has to be consistent inside the same bounded context.

Bounded context span across the entire onion architecture, meaning that we would want to define another project/microservice for another bounded context.

We use a `context map` to map the connections between different bounded contexts.

#### Bounded contexts vs subdomains
Subdomain is a sub domain of the problem. It's usually defined not by devs, but rather domain experts.
Bounded context is a sub context of the solution.
Ideally they should be one to one.

#### Context map

![[Pasted image 20240713011502.png|500]]
ATM and Snack machine shares Money.

Code for business logics and utility shouldn't be reused in most case.

If we do need to use it, eg. Money, we can extract to a shared kernel.

The root aggregate, ATM, would look like this.
![[Pasted image 20240713012814.png]]

ATM table in the db would look like this

![[Pasted image 20240713012940.png|400]]

### Domain events

Let's say now we want a new requirement, to track user payments and move money from snack machine to ATM.

We need a new subdomain/bounded context. Let's call it management.

Let's say the money charged from ATM goes to office bank account. And when we transfer money from snack machine to ATM, we don't do it directly, but move it to office bank account first and then send it to ATM.

Our new updated context map

![[Pasted image 20240713014410.png|500]]

Let's introduce an `HeadOffice` entity.

Domain event - Describe an event for our domain. They are used to de-couple bounded contexts.

In this case, domain events will help ATM decouple from Management bounded context. Because sending money from ATM to HeadOffice should not be the responsibility of the ATM as it violates single responsibility.


#### Handling domain events the classical way

We define an `IDomainEvent` interface. 

We define an `IHandler` interface that derives from a particular domain event.

![[Pasted image 20240713015046.png]]

Eg. Balance changed event.
We need both the event and a handler.

![[Pasted image 20240713015123.png|500]]

Handler
![[Pasted image 20240713015206.png|600]]

To raise the events, we introduce a static `DomainEvents` class.
![[Pasted image 20240713015358.png]]
![[Pasted image 20240713015412.png]]
This approach violates two principles
* Layers in onion architecture should only know themselves and lower layer ones. Here ATM entity works with domain events that belongs at a higher level
* Doesn't fit into Unit Of Work. We raise a domain event, and validates and saves to datatbase. But what if validation fails, domain event would be processed.


#### A better approach to handling events
Entities should be responsible for creating an event only, while delegating the raising to infrastructure.

So our entities that inherits this AggregateRoot is no longer responsible for dispatching it, only creating it.

### Further enhancements

Factories - class for creating domain entities

Domain services - containers for domain logic, process logic that doesn't belong to entities or value objects. 



















