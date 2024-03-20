
Code first
* You don't bother with db, just create the entities, and ef handles everything with EF migrations.
* Database version control, since all changes are tracked
* The con is that everything is done through code. Modifications to db won't reflect on entities. So if you want to update a db you need to change the entity classes and then run update database. Hard to write database objects like Stored Procedures and triggers.




Database first
* For supporting existing db ( you can easily generate code that mirrors database structure, don't need to spend time write c# classes that represent the tables )
* Manually setup tables and columns in the db using tools like mssql. And then code the entities to match these columns.
* If you want to change schema, you need to update EF mappings manually.