There are different kinds of two pointers.
* slow and fast pointer, usually for linked list, see [[]]



* opposite direction two pointer, where the two pointers start off at the two ends
[[167. Two Sum II - Input Array Is Sorted]]



* sliding window, which can have flexible size or fixed size

A typical sliding window looks like below

```python
start = 0

# keep expanding right
for end in range(len(arr)):
	# update whatever we are looking for

	# after the update, while condition is broken
	while condition broken:
		start += 1
		# maybe we track the optimal result so far here

	# or we could track the optimal result so far here

return ans
```


[[713. Subarray Product Less Than K]]
[[2958. Length of Longest Subarray With at Most K Frequency]]
