ORM - object relational mapper, maps the concept of OOP to relational db

C# doesn't understand SQLs, sees it as string
ORM translates LINQ ( C# understands ) to SQL.
ORM translate tabular data to relational objects.
Can send raw sql, which  gives you full power of sql

EF cheat sheet -  https://github.com/rstropek/htl-leo-csharp-4/blob/master/slides/ef-aspnet-cheat-sheet.md



###### Migration overview
When a db model change is introduced ( eg. New entities/ properties are added/removed ), developer use EF core to add a migration describing the change to necessary to keep data in sync with with application data model while keeping existing data.
![[Pasted image 20240320005650.png]]

EF core compares current model against a snapshot of old model to determine differences, and generate migration source files.

Once a migration is generated, it can be applied to a db. EF core records all applied migrations in a special history table.

Migration is usually a two step process
1. Create the migration script
2. Applying migration




##### DbContext
A class that derives from `DbContext` is called a context class.
It's a bridge between your domain class and db.


![[Pasted image 20240320010226.png]]
It's the primary class for interacting with the db.
Querying: Converts LINQ to SQL queries and sends to db
Change tracking: tracking changes on entities
Persisting Data: Insert/Update/Delete
Caching: Stores entities retrieved during life time of context class

##### Dbset
![[dbset-fg1.png]]

A `DbSet` class represent an entity set that can be used for crud.
A context class must include DbSet type properties for entities which map to db tables/views.