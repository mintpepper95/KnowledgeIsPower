#### N-way set Associative Cache

##### 3 fixed sized cache structures

##### Direct mapped cache
Every key is mapped to a single location.
This can cause conflict misses ( where two or more keys are mapped to the same location), causing frequent replacements.
##### Fully associative
In a fully associative cache, there's no mapping between keys and locations (basically  not forcing data to be stored into particular blocks)

##### Set associative
S number of N-sized sets (N-way). 
In this case, it's 5-way sets. Total capacity is 4 * 5 = 20.
Every key is mapped to a specific set ( by a hash fn ), and stored in any location within that set. 

Conflict misses can occur with set associative caches when a new key is added to a full set. In such cases, a replacement algorithm decides which existing key to evict from the cache for the new key.
![[Pasted image 20240408135339.png]]

##### Replacement algorithms

We can use a doubly-linked list to represent the access order.
Eg. Most recently used item can be added to the front ( think why ) in LRU.

We can use cache to store the actual Node, where each node contains a key and a value. Assume we allow for keys and values of arbitrary types. For a given cache, all keys must be same type, all values must be same type.

Introduce an optional custom param for alternative replacement algo. (hint abstract classes)


```
[HEAD, 1, 2, 5, 7] -> we accessed 5, [HEAD, 5, 1, 2, 7]
```


###### LRU ( Least Recently used )


###### MRU ( Most Recently used )


