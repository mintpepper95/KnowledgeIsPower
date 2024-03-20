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


![[Pasted image 20240319235232.png]]
This is denormalized data, we combined multiple things into one table, so that accessing is fast.
Data integrity is not maintained. Common to have redundant data. Excess data, storage is less optimal.




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










