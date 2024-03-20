[[#Denormalization]]
[[#DB indexing]]
[[#Sharding]]




#### Denormalization
Normalized - For storing non-redundant and consistent data
Denormalized - Combining data so querying is fast

![[Pasted image 20240319235202.png]]
This is normalized data.
Data integrity is maintained and little redundant data.
Many tables.

**Normalization** is the process of removing redundant data from the database by splitting the table in a well-defined manner in order to maintain data integrity. This process saves much of the storage space.


![[Pasted image 20240319235232.png]]
This is denormalized data, we combined multiple things into one table, so that accessing is fast.
Data integrity is not maintained. Common to have redundant data. Excess data, storage is less optimal.

**De-normalization** is the process of adding up redundant data on the table in order to speed up the complex queries and thus achieve better performance.

#### Different types of normalizations
- **First Normal Form (1NF):** A relation is said to be in 1NF only when all the entities of the table contain unique or atomic values.
- **Second Normal Form (2NF):** A relation is said to be in 2NF only if it is in 1NF and all the non-key attribute of the table is fully dependent on the primary key.
- **Third Normal Form (3NF):** A relation is said to be in 3NF only if it is in 2NF and every non-key attribute of the table is not transitively dependent on the primary key.

#### PK and Composite Key and Unique Key
**Primary Key** is that column of the table whose every row data is uniquely identified. Every row in the table must have a primary key and no two rows can have the same primary key. Primary key value can never be null nor can it be modified or updated.

**Composite Key** is a form of the candidate key where a set of columns will uniquely identify every row in the table.

Unique Keys are like PKs but can have NULL values.

A Stored procedure is a collection of pre-compiled SQL Queries. Meaning once we compile it next time and can just reuse.

#### Function vs Stored Procedure
Functions have outputs and optionally inputs. 
Use function if you want to compute and return a value/table for other SQL statements.

While SP are more like batch script.


#### DB indexing
![[Pasted image 20240320000007.png]]
While we see data as rows. Data are stored inside blocks in the db. Each block has a fixed space. A block is the smallest unit of data used by a database. This data is persisted in disk.

When we do a query like a select, the db has to load all records in memory. Now we have all the records. It will traverse every single record any return back any record that meets our select criteria. O(n) This is known as a full table scan.

With an index, we set what col we most use to query our data as index ( As updating/writing a table with indexes takes more time than one without). Eg. For name, it would not be a good idea to use it as PK due to same name, so we can create our own indexes. So ideal for most common cols we use for searching and we don't update the tables too much.

When we create an index on a column, we are basically giving it some location coordinates.

Eg. 
B is the block. I is the index number within that block.
Note real indexes look more like b-tree, this is just for illustration.
Note this is sorted in alphabetical order.
![[Pasted image 20240320000952.png]]

Things that are sorted are fast to query.


#### Sharding
Horizontal scaling technique.
Used in distributed databases where a single db can't handle the volume of data/requests. We basically divide a large dataset into smaller subsets called shards, and store each shard on a separate db server.










