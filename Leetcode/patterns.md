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


Can also be used to find middle node, since fast moves twice as fast.



Linked List In-Place Reversal
Monotonic Stack 
Top 'k' Elements
Quick Select 
Overlapping Intervals
Modified Binary Search
Depth-First Search(DFS)
Breadth-First Search(BFS)
Matrix Traversal
Backtracking
Dynamic Programming