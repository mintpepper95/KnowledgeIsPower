Explain where each commonly used

Prefix Sum
when you need to query something (e.g. sum, product) over a range of continuous items in an array. We can quickly compute the values over a range [i, j] from [0, j] - [0, i - 1]

Two Pointer
move them towards each other or away from each other

Sliding Window
E.g. max subarray product of subarray size k - brute force 
We avoid repeated calculations over a subarray by removing old element, adding in new element, and update our calculation. Instead of calculating over the entire subarray of k again. This reduces time complexity of O(n * k) to O(n)

Fast & Slow Pointer
For Linked lists and array for finding cycles - Core idea is to move two pointers at different speeds. Once they both get into the linked list. Since fast moves double speed compared to slow. E.g., slow moves 1 step, fast moves 2 steps. Slow moves 2 steps, fast moves 4 steps. So every time fast moves, it will be closer to slow ( since both in a cycle ).

Key insight - fast gains one step

Extension - find start of a cycle - move slow pointer to start, and both pointer move one step at a time, when they meet again, it's the start of a cycle


math: slow = L (len before cycle) + M ( steps before cycle )
      fast = 2 * (L + M) = L + K * C + M  (K is number of loops, C is cycle length)
      L + M = K * C
      L = K * C - M
    where K * C - M is meeting to

E.g. [287. Find the Duplicate Number]


Can also be used to find middle node, since fast moves twice as fast.


Linked List In-Place Reversal
Dummy node is useful for linked listr
Dummy value is useful for dp

Monotonic Stack 
- find next greater or next smaller element
- think of popped element, e.g. for next smaller element, we want to pop off the top of stack if it's > next element, until stack_element < next element


Top 'k' Elements
K largest - min heap of size k, think why, cause min element in min heap serves as a lower bound, if larger element comes in, the min element will be updated , if new element smaller than min element than it won't be added


Quick Select - O(n), for finding the kth smallest or largest element
pick a pivot, partition array around pivot     [....3....]

Overlapping Intervals
E.g. if we need to merge all overlapping intervals
sort them by 1st element, iterate through and merge if overlaps

Modified Binary Search

Depth-First Search(DFS)
E.g. finding topological order in directed acyclic graph
DFS to find all the nodes, u -> v, u needs to happen before v
So basically DFS with post order push to the stack, as the recursive dfs calls will handle all the children, then we add current to stack.

So reversing will be topo sorted.


Breadth-First Search(BFS)
E.g. finding shortest transformation sequence from one word to another
eg. hit, hot, dot, dog

generic state
 -ot -> hot, dot, lot
 h-t -> hot
 ho- -> hog

for a word, generate all its generic patterns, and look them up to find neighbours
```python
from collections import defaultdict, deque

pattern_dict = defaultdict(list)
for word in wordList:
    for i in range(len(word)):
        pattern = word[:i] + '*' + word[i+1:]
        pattern_dict[pattern].append(word)
```

E.g. if you want to delete child before the parent, use post-order traversal

Matrix Traversal
Most matrix traversal can be seen as graph problems. Usually a 2d array.

Backtracking
Exploring all potential solution
Generate all valid parenthesis of a given length - basically brute force using backtacking

Dynamic Programming
Imagine solution already exist for subproblems