We need the following nuget packages to sue DbContext class
microsoft.entityframeworkcore 

choosing a database
We use entityframeworkcore.sqlite


This allows us to add migration, drop databse etc
microsoft.entityframeworkcore.tools

migration, we can use it for initial migration, from no db to creating a db
`add-migration CityInfoDBInitialMigration`

This will generate a snapshot ( of current context model ) and our migration

`update-database` to apply migration

To remove a bad migration that hasn't applied yet, do
`Remove-Migration`

#### Seeding the databse - provide some sample data


#### SQL injection
![[Pasted image 20240711021411.png]]



#### Automapper
automapper
