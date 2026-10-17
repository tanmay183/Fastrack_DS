# Topic: Heap (Priority Queue) in Python

**All concepts + 15 standard problems from easy to hard, every line of code explained in comments, with output**

> **Goal of this note:** after this, whenever a problem says *"k-th largest"*, *"top k"*, *"always pick the smallest/largest next"*, *"merge sorted lists"*, *"running median"* or *"shortest path with weights"*, you should immediately think **heap** and be able to write the code without hesitation.

How to use this note: first read Sections 1 to 5 (concepts, about 40 minutes), then solve the 15 problems in Section 6 in order. Try each problem yourself for 10 minutes **before** reading the solution.

---

## Table of Contents

1. [What is a heap?](#1-what-is-a-heap)
2. [How a heap is stored (array representation)](#2-how-a-heap-is-stored-array-representation)
3. [Heap operations and how they work inside](#3-heap-operations-and-how-they-work-inside)
4. [Python's `heapq` module](#4-pythons-heapq-module)
5. [Heap from scratch (interview favourite)](#5-heap-from-scratch-interview-favourite)
6. [Patterns: when to use a heap](#6-patterns-when-to-use-a-heap)
7. [The 15 problems](#7-the-15-problems)
8. [Mistakes, cheat sheet, and extra practice](#8-mistakes-cheat-sheet-and-extra-practice)

---

## 1. What is a heap?

### 1.1 The idea in plain words

Think of a **hospital emergency room**. Patients do not get treated in the order they arrive. The **most urgent patient is always treated first**, no matter when they came in. New patients keep arriving, and the doctor always asks: *"Who is the most urgent right now?"*

That structure is called a **priority queue**, and the standard efficient way to build it is a **heap**.

> A heap gives you the **smallest** (or **largest**) element **instantly**, and lets you insert and remove elements **quickly** (O(log n)), without keeping everything fully sorted.

### 1.2 The two rules of a heap

1. **Shape rule:** it is a **complete binary tree**. Every level is full, except possibly the last, which is filled from left to right (no gaps).
2. **Order rule (heap property):**
   - **Min-heap:** every parent is **less than or equal to** its children. The root is the **minimum**.
   - **Max-heap:** every parent is **greater than or equal to** its children. The root is the **maximum**.

```
Min-heap                       Max-heap

        1                            9
      /   \                        /   \
     3     2                      7     8
    / \   / \                    / \   / \
   7   4 8   5                  3   4 5   2
```

### 1.3 Important: a heap is NOT a sorted list

Only **parent versus child** is ordered. Siblings have no order (in the min-heap above, `3` and `2` are siblings, and `3` comes first even though it is bigger). The bottom level reads `7, 4, 8, 5`, which is not sorted either. All we promise is: **the root is the smallest**.

This "weaker than sorted" promise is exactly why a heap is **cheaper** to maintain than a sorted list.

| Structure | Get min | Insert | Remove min |
|---|---|---|---|
| Unsorted list | O(n) | O(1) | O(n) |
| Sorted list | O(1) | O(n) | O(1) at the front, O(n) with `pop(0)` |
| **Heap** | **O(1)** | **O(log n)** | **O(log n)** |

---

## 2. How a heap is stored (array representation)

Because the tree is **complete** (no gaps), we do not need pointers. We store it in a plain **array**, level by level:

```
Tree:                           Array:
        1                       index:  0  1  2  3  4  5  6
      /   \                     value: [1, 3, 2, 7, 4, 8, 5]
     3     2
    / \   / \
   7   4 8   5
```

For a node at index `i` (0-based):

| Relation | Formula |
|---|---|
| Parent | `(i - 1) // 2` |
| Left child | `2 * i + 1` |
| Right child | `2 * i + 2` |

```python
heap = [1, 3, 2, 7, 4, 8, 5]            # the array form of the tree above
i = 1                                    # look at the node at index 1 (value 3)
print("parent of", i, "=", (i - 1) // 2)     # parent is at index 0 (value 1)
print("left child of", i, "=", 2 * i + 1)    # left child is at index 3 (value 7)
print("right child of", i, "=", 2 * i + 2)   # right child is at index 4 (value 4)
```

**Output:**
```text
parent of 1 = 0
left child of 1 = 3
right child of 1 = 4
```

**Why this matters:** parent and child positions are found with simple arithmetic, so the heap uses only O(n) space and is very fast in practice.

> If your array is **1-indexed**, the formulas become parent = `i // 2`, left = `2i`, right = `2i + 1`. Python lists are 0-indexed, so use the first set.

---

## 3. Heap operations and how they work inside

### 3.1 Peek (see the minimum): O(1)

The minimum is always at **index 0**. Just read `heap[0]`.

### 3.2 Insert / push (sift up): O(log n)

1. Add the new value at the **end** of the array (keeps the tree complete).
2. While it is **smaller than its parent**, swap them (move it **up**).

**Example:** insert `0` into `[1, 3, 2, 7, 4, 8, 5]`.

```
Step 0: append 0 at index 7        [1, 3, 2, 7, 4, 8, 5, 0]   parent of 7 is index 3 (value 7)
Step 1: 0 < 7, swap                [1, 3, 2, 0, 4, 8, 5, 7]   now at index 3, parent is index 1 (value 3)
Step 2: 0 < 3, swap                [1, 0, 2, 3, 4, 8, 5, 7]   now at index 1, parent is index 0 (value 1)
Step 3: 0 < 1, swap                [0, 1, 2, 3, 4, 8, 5, 7]   now at index 0, no parent. Done
```

The new value climbs at most the **height** of the tree, which is log n levels.

### 3.3 Remove minimum / pop (sift down): O(log n)

1. Save the root (that is the answer).
2. Move the **last** element to the root (keeps the tree complete).
3. While it is **bigger than a child**, swap it with the **smaller child** (move it **down**).

**Example:** pop from `[0, 1, 2, 3, 4, 8, 5, 7]`.

```
Step 0: answer = 0. Move last (7) to the root       [7, 1, 2, 3, 4, 8, 5]
Step 1: children of index 0 are 1 and 2. Smaller is 1. 7 > 1, swap
                                                    [1, 7, 2, 3, 4, 8, 5]
Step 2: children of index 1 are 3 and 4. Smaller is 3. 7 > 3, swap
                                                    [1, 3, 2, 7, 4, 8, 5]
Step 3: index 3 has no children. Done
```

Always swap with the **smaller** child. If you swapped with the bigger one, the bigger child would end up above the smaller one and break the heap rule.

### 3.4 Heapify (build a heap from an array): O(n)

Turning an existing list into a heap does **not** need n separate pushes (that would be O(n log n)). Instead:

- Start from the **last parent** (index `n // 2 - 1`) and go back to index 0.
- At each index, do a **sift down**.

```
Why O(n)?  Half of all nodes are leaves and need 0 work.
A quarter of the nodes move at most 1 step, an eighth at most 2 steps, and so on.
Total work = n/4*1 + n/8*2 + n/16*3 + ...  which adds up to about n, not n log n.
```

### 3.5 Complexity table

| Operation | Time | Notes |
|---|---|---|
| Peek min/max | O(1) | `heap[0]` |
| Push | O(log n) | sift up |
| Pop | O(log n) | sift down |
| Heapify (build from list) | **O(n)** | faster than n pushes |
| Search for a value | O(n) | the heap cannot help here |
| Delete an arbitrary value | O(n) | find it first (see lazy deletion, Section 6) |
| Heap sort | O(n log n) | n pops |
| Space | O(n) | the array |

---

## 4. Python's `heapq` module

Python has no separate heap class. The `heapq` module works on an **ordinary list**, and it is a **min-heap only**.

### 4.1 The core functions

```python
import heapq                              # import the heap library

h = []                                    # an empty list will act as our heap
for x in [5, 3, 8, 1, 2]:                 # go through the numbers one by one
    heapq.heappush(h, x)                  # push x into the heap (keeps heap order)
print(h)                                  # the raw list is in HEAP order, not sorted
print(h[0])                               # peek: the smallest is always at index 0
print(heapq.heappop(h))                   # remove and return the smallest
print(h)                                  # heap order is repaired after the pop
print(sorted(h))                          # sorted() gives the true sorted view
```

**Output:**
```text
[1, 2, 8, 5, 3]
1
1
[2, 3, 8, 5]
[2, 3, 5, 8]
```

**Explanation:** `[1, 2, 8, 5, 3]` is a valid min-heap but not sorted. **Never read `h[1]` expecting the second smallest.** The only guaranteed position is `h[0]`.

```python
import heapq                              # import the heap library

a = [5, 3, 8, 1, 2]                       # a normal unordered list
heapq.heapify(a)                          # convert in place to a heap in O(n), returns None
print(a)                                  # now it follows the min-heap rule

b = [1, 3, 2, 7, 4, 8, 5]                 # the heap from Section 2
heapq.heappush(b, 0)                      # push 0 (this is the sift-up example from 3.2)
print(b)                                  # matches the hand trace: [0, 1, 2, 3, 4, 8, 5, 7]
print(heapq.heappop(b))                   # pop: returns 0 (the sift-down example from 3.3)
print(b)                                  # matches the hand trace: [1, 3, 2, 7, 4, 8, 5]
```

**Output:**
```text
[1, 2, 8, 3, 5]
[0, 1, 2, 3, 4, 8, 5, 7]
0
[1, 3, 2, 7, 4, 8, 5]
```

### 4.2 Two combined functions (faster than push followed by pop)

```python
import heapq                              # import the heap library

h = [2, 5, 9]                             # a list that is already a valid min-heap
heapq.heapify(h)                          # make sure it is a heap
print(heapq.heappushpop(h, 1))            # push 1 THEN pop smallest: 1 is smallest, returns 1 at once
print(heapq.heappushpop(h, 4))            # push 4 THEN pop smallest: pops 2
print(h)                                  # heap now holds the remaining values
print(heapq.heapreplace(h, 7))            # pop smallest (4) FIRST, then push 7
print(h)                                  # 4 is gone, 7 added
```

**Output:**
```text
1
2
[4, 5, 9]
4
[5, 7, 9]
```

| Function | Order of work | Use when |
|---|---|---|
| `heappushpop(h, x)` | push x, then pop | keeping a heap at a fixed size k (the new item may be rejected immediately) |
| `heapreplace(h, x)` | pop, then push x | replacing the root (meeting rooms, k-way merge) |

### 4.3 `nlargest`, `nsmallest`, `merge`

```python
import heapq                                          # import the heap library

nums = [4, 1, 7, 3, 9, 2]                             # sample numbers
print(heapq.nlargest(3, nums))                        # 3 largest, in descending order
print(heapq.nsmallest(2, nums))                       # 2 smallest, in ascending order
words = ["kiwi", "banana", "fig", "cherry"]           # sample words
print(heapq.nlargest(2, words, key=len))              # 2 longest words, using key like sort does
print(list(heapq.merge([1, 4, 7], [2, 5, 8], [3, 6, 9])))   # merge sorted inputs lazily
```

**Output:**
```text
[9, 7, 4]
[1, 2]
['banana', 'cherry']
[1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**Time:** `nlargest(k, arr)` is O(n log k). `merge` of m total items from k lists is O(m log k).

### 4.4 Getting a MAX-heap: negate the values

`heapq` only provides a min-heap. For a max-heap, **store negative values** and negate again when you read them.

```python
import heapq                                  # import the heap library

mh = []                                       # list that will act as a max-heap
for x in [5, 3, 8, 1]:                        # go through the numbers
    heapq.heappush(mh, -x)                    # store the NEGATIVE: biggest number becomes smallest negative
print(-heapq.heappop(mh))                     # pop gives -8, negate back to 8 (the maximum)
print(-mh[0])                                 # peek: next maximum is 5
```

**Output:**
```text
8
5
```

### 4.5 Heaps of tuples (priority + data)

Python compares tuples **element by element**, so `(priority, item)` is the standard way to attach data to a priority.

```python
import heapq                                          # import the heap library

tasks = []                                            # heap of (priority, task name)
heapq.heappush(tasks, (2, "write code"))              # priority 2
heapq.heappush(tasks, (1, "fix bug"))                 # priority 1 (most urgent)
heapq.heappush(tasks, (3, "relax"))                   # priority 3
while tasks:                                          # keep going until the heap is empty
    p, t = heapq.heappop(tasks)                       # always the lowest priority number first
    print(p, t)                                       # show the order tasks come out
```

**Output:**
```text
1 fix bug
2 write code
3 relax
```

### 4.6 The tie-break trap (very common bug)

If two tuples have the **same first value**, Python compares the **second** value. If that value cannot be compared, you get a `TypeError`.

```python
import heapq                                              # import the heap library

try:                                                      # catch the error so the script continues
    h2 = []                                               # empty heap
    heapq.heappush(h2, (1, {"a": 1}))                     # first item: priority 1 with a dict
    heapq.heappush(h2, (1, {"b": 2}))                     # same priority, so Python must compare the dicts
except TypeError as e:                                    # dicts cannot be compared with <
    print("Error:", e)                                    # show the error message

cnt = 0                                                   # unique counter used as tie-breaker
h3 = []                                                   # fresh heap
for pr, nm in [(1, {"a": 1}), (1, {"b": 2})]:             # same two items again
    heapq.heappush(h3, (pr, cnt, nm))                     # (priority, counter, data): counter decides ties
    cnt += 1                                              # next item gets a bigger counter
print(heapq.heappop(h3))                                  # first inserted item comes out first
```

**Output:**
```text
Error: '<' not supported between instances of 'dict' and 'dict'
(1, 0, {'a': 1})
```

**Rule:** when the data part may not be comparable (dicts, linked-list nodes, custom objects), put a **unique index or counter** between the priority and the data. This is also how you get **first-in-first-out** order among equal priorities.

### 4.7 Heap of custom objects (`__lt__`)

```python
import heapq                                      # import the heap library

class Job:                                        # a simple class to store in the heap
    def __init__(self, name, prio):               # constructor receives name and priority
        self.name = name                          # store the name
        self.prio = prio                          # store the priority
    def __lt__(self, other):                      # heapq uses "<" to compare two Job objects
        return self.prio < other.prio             # lower priority number = comes out first

jobs = []                                         # heap of Job objects
for n, p in [("c", 3), ("a", 1), ("b", 2)]:       # three jobs with different priorities
    heapq.heappush(jobs, Job(n, p))               # push each Job (heap uses __lt__)
print([heapq.heappop(jobs).name for _ in range(3)])   # pop three times and collect the names
```

**Output:**
```text
['a', 'b', 'c']
```

### 4.8 Heap sort in two lines

```python
import heapq                                      # import the heap library

def heap_sort(arr):                               # returns a sorted copy of arr
    h = list(arr)                                 # copy so the input is not changed
    heapq.heapify(h)                              # build the heap in O(n)
    return [heapq.heappop(h) for _ in range(len(h))]   # n pops, each O(log n), come out in sorted order

print(heap_sort([9, 4, 7, 1, 8, 2]))              # test it
```

**Output:**
```text
[1, 2, 4, 7, 8, 9]
```

**Time O(n log n), space O(n)** for this version. The classic in-place heap sort uses O(1) extra space.

### 4.9 `heapq` quick reference

| Task | Code |
|---|---|
| Create empty heap | `h = []` |
| Build from list | `heapq.heapify(h)` |
| Push | `heapq.heappush(h, x)` |
| Pop smallest | `heapq.heappop(h)` |
| Peek smallest | `h[0]` |
| Size | `len(h)` |
| Max-heap | push `-x`, read `-h[0]` |
| Push + pop in one step | `heapq.heappushpop(h, x)` |
| Pop + push in one step | `heapq.heapreplace(h, x)` |
| K largest / smallest | `heapq.nlargest(k, h)` / `heapq.nsmallest(k, h)` |

---

## 5. Heap from scratch (interview favourite)

Interviewers sometimes say *"implement a min-heap without using `heapq`"*. This is the whole thing. Read it slowly once, because it is exactly the sift-up and sift-down from Section 3.

```python
class MinHeap:                                         # our own min-heap class
    def __init__(self):                                # constructor
        self.data = []                                 # the heap lives in a plain list

    def __len__(self):                                 # lets us call len(heap)
        return len(self.data)                          # number of elements stored

    def peek(self):                                    # look at the minimum without removing it
        return self.data[0]                            # the minimum is always at index 0

    def push(self, val):                               # insert a value
        self.data.append(val)                          # step 1: add at the end (keeps tree complete)
        self._sift_up(len(self.data) - 1)              # step 2: fix the order by moving it up

    def pop(self):                                     # remove and return the minimum
        data = self.data                               # short name for readability
        top = data[0]                                  # step 1: remember the root (our answer)
        last = data.pop()                              # step 2: remove the last element from the list
        if data:                                       # if the heap is not empty after removing it
            data[0] = last                             # step 3: put that last element at the root
            self._sift_down(0)                         # step 4: move it down to its correct place
        return top                                     # return the old minimum

    def _sift_up(self, i):                             # move the element at index i upward
        data = self.data                               # short name
        while i > 0:                                   # stop when we reach the root
            parent = (i - 1) // 2                      # index of the parent
            if data[i] < data[parent]:                 # child is smaller than parent: wrong order
                data[i], data[parent] = data[parent], data[i]   # swap child and parent
                i = parent                             # continue from the parent's position
            else:                                      # order is already correct
                break                                  # nothing more to fix

    def _sift_down(self, i):                           # move the element at index i downward
        data = self.data                               # short name
        n = len(data)                                  # current heap size
        while True:                                    # loop until the element settles
            smallest = i                               # assume the current node is the smallest
            left, right = 2 * i + 1, 2 * i + 2         # indexes of its two children
            if left < n and data[left] < data[smallest]:      # left child exists and is smaller
                smallest = left                        # remember the left child
            if right < n and data[right] < data[smallest]:    # right child exists and is even smaller
                smallest = right                       # remember the right child
            if smallest == i:                          # node is not bigger than its children
                break                                  # it is in the right place, stop
            data[i], data[smallest] = data[smallest], data[i]   # swap with the smaller child
            i = smallest                               # continue from the child's position

    def heapify(self, arr):                            # build a heap from a list in O(n)
        self.data = list(arr)                          # copy the list
        for i in range(len(self.data) // 2 - 1, -1, -1):   # from the last parent back to the root
            self._sift_down(i)                         # fix each subtree from the bottom up

h = MinHeap()                                          # create an empty heap
for x in [5, 3, 8, 1, 2]:                              # insert these values one by one
    h.push(x)                                          # each push is O(log n)
print(h.data)                                          # internal array (same layout as heapq gave)
print(h.pop())                                         # smallest value comes out
print(h.data)                                          # heap after the pop
print([h.pop() for _ in range(len(h))])               # pop the rest: values come out sorted

g = MinHeap()                                          # another heap
g.heapify([5, 3, 8, 1, 2])                             # build it in O(n) instead of 5 pushes
print(g.data, g.peek())                                # array layout and the minimum
```

**Output:**
```text
[1, 2, 8, 5, 3]
1
[2, 3, 8, 5]
[2, 3, 5, 8]
[1, 2, 8, 3, 5] 1
```

**To make a max-heap:** flip the two comparisons (`<` becomes `>`) in `_sift_up` and `_sift_down`, or store negated values.

---

## 6. Patterns: when to use a heap

### 6.1 Clue words in the problem

| If the problem says... | Think... |
|---|---|
| "k-th largest / smallest" | heap of size k |
| "top k frequent / closest / largest" | heap of size k |
| "always process the smallest (or largest) next" | priority queue |
| "merge k sorted lists/arrays" | heap of k heads |
| "running median" / "median of a stream" | two heaps |
| "minimum cost to combine items" | greedy with min-heap |
| "schedule tasks / meeting rooms" | heap on end times / counts |
| "shortest path with positive weights" | Dijkstra (heap) |
| "next best choice changes as data arrives" | heap |

### 6.2 The Top-K rule (memorize this)

| You want | Keep a heap of size k that is a... | Why |
|---|---|---|
| **k largest** | **min-heap** | the root is the smallest of the k largest, so it is the one to kick out |
| **k smallest** | **max-heap** | the root is the largest of the k smallest, so it is the one to kick out |

It feels backwards at first, but think of it as a **club with k seats**. When a new person arrives, the *weakest* current member (the root) leaves if the newcomer is stronger.

**Cost:** O(n log k) time, O(k) space. When k is much smaller than n, this beats sorting (O(n log n)).

### 6.3 The two-heap pattern

Keep the **smaller half** of the numbers in a **max-heap** and the **larger half** in a **min-heap**. The two roots are the middle elements, so the **median** is always available in O(1).

```
small half (max-heap)     large half (min-heap)
   [1, 2, 3]       |        [4, 5, 6]
        top = 3    |   top = 4         median = (3 + 4) / 2
```

### 6.4 Lazy deletion

A heap cannot remove an arbitrary item quickly. The trick: **do not remove it now**. Mark it as stale (by index, by a `removed` set, or by comparing with the latest known distance). When the stale item reaches the **top**, just **skip it**. You will see this in Dijkstra (Problem 12) and sliding window maximum (Problem 15).

### 6.5 Heap or sort?

| Situation | Better choice |
|---|---|
| Need everything in order, once | `sort()` |
| Need only the top k from a huge or streaming input | heap |
| Data keeps changing, and you repeatedly need the min/max | heap |
| Need the k-th element only, once, on a fixed array | heap, or quickselect (O(n) average) |

---

## 7. The 15 problems

| # | Problem | Level | Heap idea |
|---|---|---|---|
| 1 | Last Stone Weight | Easy | max-heap, smash top two |
| 2 | Kth Largest Element in a Stream | Easy | min-heap of size k |
| 3 | Minimum Cost to Connect Ropes | Easy | greedy, always join two smallest |
| 4 | Kth Largest Element in an Array | Medium | min-heap of size k |
| 5 | Top K Frequent Elements | Medium | count first, then heap of size k |
| 6 | K Closest Points to Origin | Medium | max-heap of size k |
| 7 | Meeting Rooms II | Medium | min-heap of end times |
| 8 | Reorganize String | Medium | max-heap by frequency |
| 9 | Kth Smallest Element in a Sorted Matrix | Medium | heap walks the matrix |
| 10 | Find K Pairs with Smallest Sums | Medium | heap expands pairs |
| 11 | Task Scheduler | Medium | max-heap plus cooldown queue |
| 12 | Network Delay Time (Dijkstra) | Medium/Hard | min-heap on distance, lazy deletion |
| 13 | Merge K Sorted Linked Lists | Hard | min-heap of k heads |
| 14 | Find Median from Data Stream | Hard | two heaps |
| 15 | Sliding Window Maximum | Hard | max-heap with lazy deletion |

For each problem you get: the question, the idea, the code **with every line explained in a comment**, the output, and the complexity.

---

### Problem 1: Last Stone Weight (Easy)

**Question:** You have stones with given weights. Each turn, take the **two heaviest** stones and smash them together. If their weights are equal, both are destroyed. Otherwise, the lighter one is destroyed and the heavier one shrinks to `heavy - light`. Return the weight of the last stone (or `0` if none is left).

**Example:** `[2, 7, 4, 1, 8, 1]` gives `1`.

**Idea:** We repeatedly need the **largest** element, and the set keeps changing. That is a **max-heap**. Python has only a min-heap, so we store **negative** weights.

```python
import heapq                                               # import the heap library

def last_stone_weight(stones):                             # takes a list of stone weights
    heap = [-s for s in stones]                            # negate every weight: min-heap now acts as max-heap
    heapq.heapify(heap)                                    # build the heap in O(n)
    while len(heap) > 1:                                   # smash while at least two stones remain
        first = -heapq.heappop(heap)                       # heaviest stone (negate back to positive)
        second = -heapq.heappop(heap)                      # second heaviest stone
        if first != second:                                # different weights: one stone survives
            heapq.heappush(heap, -(first - second))        # push the leftover weight (negated) back
    return -heap[0] if heap else 0                         # last stone's weight, or 0 if none left

print(last_stone_weight([2, 7, 4, 1, 8, 1]))               # the sample from the question
print(last_stone_weight([1]))                              # a single stone is the answer itself
print(last_stone_weight([3, 3]))                           # equal stones destroy each other
```

**Output:**
```text
1
1
0
```

**Dry run for `[2, 7, 4, 1, 8, 1]`:** (8, 7) become 1, stones: 2, 4, 1, 1, 1. Then (4, 2) become 2, stones: 2, 1, 1, 1. Then (2, 1) become 1, stones: 1, 1, 1. Then (1, 1) vanish, stones: 1. Answer `1`.

**Complexity:** Time O(n log n), Space O(n).

---

### Problem 2: Kth Largest Element in a Stream (Easy)

**Question:** Design a class that receives numbers one at a time. After each `add(val)`, return the **k-th largest** number seen so far.

**Idea:** Keep a **min-heap of exactly k elements** holding the k largest numbers. Its root (the smallest of those k) is the k-th largest overall.

```python
import heapq                                               # import the heap library

class KthLargest:                                          # class that tracks the kth largest number
    def __init__(self, k, nums):                           # constructor gets k and the starting numbers
        self.k = k                                         # remember k
        self.heap = []                                     # min-heap that will hold only the k largest numbers
        for n in nums:                                     # go through the initial numbers
            self.add(n)                                    # reuse add() so the size logic lives in one place

    def add(self, val):                                    # add a new number, return current kth largest
        heapq.heappush(self.heap, val)                     # push the new value into the heap
        if len(self.heap) > self.k:                        # heap has more than k items now
            heapq.heappop(self.heap)                       # remove the smallest, so only the k largest stay
        return self.heap[0]                                # root = smallest of the k largest = kth largest

kl = KthLargest(3, [4, 5, 8, 2])                           # k = 3, starting numbers given
print(kl.add(3))                                           # heap holds [4, 5, 8]: answer 4
print(kl.add(5))                                           # 3rd largest of 2,3,4,5,5,8 is 5
print(kl.add(10))                                          # 3rd largest of ...,8,10 is 5
print(kl.add(9))                                           # 3rd largest is now 8
print(kl.add(4))                                           # 4 is too small to matter: answer stays 8
```

**Output:**
```text
4
5
5
8
8
```

**Complexity:** `add` is O(log k) time. Space O(k). (Sorting every time would cost O(n log n) per query.)

---

### Problem 3: Minimum Cost to Connect Ropes (Easy)

**Question:** You have ropes of different lengths. Joining two ropes costs the **sum of their lengths**. Connect all ropes into one with the **minimum total cost**.

**Example:** `[4, 3, 2, 6]` gives `29`.

**Idea (greedy):** Every rope's length gets paid again each time it is part of a join, so **long ropes should be joined late**. Always join the **two shortest** ropes, and put the result back. This is exactly Huffman coding.

```python
import heapq                                               # import the heap library

def min_cost_to_connect_ropes(ropes):                      # takes a list of rope lengths
    heap = list(ropes)                                     # copy so the caller's list is not changed
    heapq.heapify(heap)                                    # make it a min-heap in O(n)
    total_cost = 0                                         # running total of all join costs
    while len(heap) > 1:                                   # until only one rope is left
        a = heapq.heappop(heap)                            # shortest rope
        b = heapq.heappop(heap)                            # second shortest rope
        cost = a + b                                       # cost of joining these two
        total_cost += cost                                 # add it to the answer
        heapq.heappush(heap, cost)                         # the joined rope goes back into the pool
    return total_cost                                      # total cost to make a single rope

print(min_cost_to_connect_ropes([4, 3, 2, 6]))             # sample from the question
print(min_cost_to_connect_ropes([1, 2, 3, 4, 5]))          # another test
print(min_cost_to_connect_ropes([5]))                      # one rope: nothing to join
```

**Output:**
```text
29
33
0
```

**Dry run for `[4, 3, 2, 6]`:** join 2+3 = 5 (cost 5), ropes: 4, 5, 6. Join 4+5 = 9 (cost 14), ropes: 6, 9. Join 6+9 = 15 (cost 29).

**Complexity:** Time O(n log n), Space O(n).

---

### Problem 4: Kth Largest Element in an Array (Medium)

**Question:** Given an unsorted array, return the **k-th largest** element (in sorted order, not the k-th distinct).

**Example:** `[3, 2, 1, 5, 6, 4]`, `k = 2` gives `5`.

**Idea:** Same as Problem 2, on a fixed array. A **min-heap of size k** keeps the k biggest numbers, and its root is the answer. Sorting works too (O(n log n)), but the heap is O(n log k).

```python
import heapq                                               # import the heap library

def find_kth_largest(nums, k):                             # nums: list of numbers, k: which largest we want
    heap = []                                              # min-heap that will keep only the k largest numbers
    for x in nums:                                         # look at every number once
        heapq.heappush(heap, x)                            # push it into the heap
        if len(heap) > k:                                  # more than k numbers inside
            heapq.heappop(heap)                            # drop the smallest of them
    return heap[0]                                         # smallest of the k largest = kth largest

def find_kth_largest_short(nums, k):                       # the same answer using the library helper
    return heapq.nlargest(k, nums)[-1]                     # k largest in descending order, last one is the kth

print(find_kth_largest([3, 2, 1, 5, 6, 4], 2))             # sample from the question
print(find_kth_largest([3, 2, 3, 1, 2, 4, 5, 5, 6], 4))    # duplicates count separately
print(find_kth_largest_short([3, 2, 1, 5, 6, 4], 2))       # same as the first line
```

**Output:**
```text
5
4
5
```

**Dry run for `[3, 2, 1, 5, 6, 4]`, k = 2:** heap grows to `[2, 3]` after 3 and 2. Push 1 gives `[1, 2, 3]`, pop 1 leaves `[2, 3]`. Push 5, pop 2 leaves `[3, 5]`. Push 6, pop 3 leaves `[5, 6]`. Push 4, pop 4 leaves `[5, 6]`. Root is `5`.

**Complexity:** Time O(n log k), Space O(k). (Quickselect gives O(n) average. Mention it in interviews.)

---

### Problem 5: Top K Frequent Elements (Medium)

**Question:** Return the `k` most frequent elements of an array (any order).

**Example:** `[1, 1, 1, 2, 2, 3]`, `k = 2` gives `[1, 2]`.

**Idea:** Step 1: **count** frequencies with a dictionary. Step 2: keep a **min-heap of size k** of `(count, value)` pairs. The pair with the smallest count gets kicked out when the heap is too big.

```python
import heapq                                               # import the heap library
from collections import Counter                            # Counter counts how often each value appears

def top_k_frequent(nums, k):                               # nums: list of numbers, k: how many to return
    freq = Counter(nums)                                   # dictionary: value -> how many times it appears
    heap = []                                              # min-heap of (count, value), size at most k
    for num, cnt in freq.items():                          # go through each distinct value
        heapq.heappush(heap, (cnt, num))                   # push the pair; the heap orders by count first
        if len(heap) > k:                                  # more than k pairs inside
            heapq.heappop(heap)                            # remove the pair with the smallest count
    return [num for cnt, num in heap]                      # keep only the values (order is arbitrary)

print(sorted(top_k_frequent([1, 1, 1, 2, 2, 3], 2)))       # sorted() only to make the printed order fixed
print(top_k_frequent([1], 1))                              # single element
print(sorted(top_k_frequent([4, 4, 4, 5, 5, 6, 6, 6, 6, 7], 2)))   # 6 appears 4 times, 4 appears 3 times
```

**Output:**
```text
[1, 2]
[1]
[4, 6]
```

**Complexity:** Time O(n + d log k) where d is the number of distinct values, Space O(d). (Bucket sort can do O(n).)

---

### Problem 6: K Closest Points to Origin (Medium)

**Question:** Given points `[x, y]`, return the `k` points closest to `(0, 0)`.

**Example:** `[[3, 3], [5, -1], [-2, 4]]`, `k = 2` gives `[[3, 3], [-2, 4]]`.

**Idea:** We want the **k smallest** distances, so we keep a **max-heap of size k** (negate the distance). The farthest point among the k sits at the root and is removed when a closer point arrives. Use **squared distance** (`x*x + y*y`): no square root is needed because the order is the same.

```python
import heapq                                               # import the heap library

def k_closest(points, k):                                  # points: list of [x, y], k: how many to return
    heap = []                                              # max-heap (via negative distance) of size at most k
    for x, y in points:                                    # look at every point
        dist = x * x + y * y                               # squared distance from the origin
        heapq.heappush(heap, (-dist, x, y))                # negate: the farthest point gets the smallest key
        if len(heap) > k:                                  # more than k points inside
            heapq.heappop(heap)                            # remove the farthest point
    return [[x, y] for _, x, y in heap]                    # drop the distances, keep the coordinates

print(sorted(k_closest([[1, 3], [-2, 2]], 1)))             # distances 10 and 8, so [-2, 2]
print(sorted(k_closest([[3, 3], [5, -1], [-2, 4]], 2)))    # distances 18, 26, 20
```

**Output:**
```text
[[-2, 2]]
[[-2, 4], [3, 3]]
```

**Complexity:** Time O(n log k), Space O(k).

---

### Problem 7: Meeting Rooms II (Medium)

**Question:** Given meeting time intervals `[start, end]`, find the **minimum number of rooms** needed so that no two meetings overlap in the same room.

**Example:** `[[0, 30], [5, 10], [15, 20]]` gives `2`.

**Idea:** Sort meetings by **start time**. Keep a **min-heap of end times**, one for each room in use. For a new meeting: if the **earliest-ending** room is already free (`end <= start`), reuse it. Otherwise open a new room. The heap size at the end is the number of rooms.

```python
import heapq                                               # import the heap library

def min_meeting_rooms(intervals):                          # intervals: list of [start, end]
    if not intervals:                                      # no meetings at all
        return 0                                           # no rooms needed
    intervals.sort(key=lambda x: x[0])                     # sort meetings by start time (changes the input list)
    rooms = []                                             # min-heap of end times of meetings in progress
    for start, end in intervals:                           # process meetings in order of start time
        if rooms and rooms[0] <= start:                    # earliest-finishing meeting is over before this one starts
            heapq.heapreplace(rooms, end)                  # reuse that room: pop its old end time, push the new end
        else:                                              # every room is still busy
            heapq.heappush(rooms, end)                     # open a new room for this meeting
    return len(rooms)                                      # number of rooms we ever had to use

print(min_meeting_rooms([[0, 30], [5, 10], [15, 20]]))     # sample from the question
print(min_meeting_rooms([[7, 10], [2, 4]]))                # no overlap, one room is enough
print(min_meeting_rooms([[1, 5], [2, 6], [3, 7], [4, 8]]))   # all four overlap
```

**Output:**
```text
2
1
4
```

**Dry run for the sample:** sorted order is `[0,30]`, `[5,10]`, `[15,20]`. Meeting `[0,30]`: rooms `[30]`. Meeting `[5,10]`: earliest end 30 > 5, so a new room: `[10, 30]`. Meeting `[15,20]`: earliest end 10 <= 15, so reuse: `[20, 30]`. Two rooms.

**Complexity:** Time O(n log n), Space O(n).

---

### Problem 8: Reorganize String (Medium)

**Question:** Rearrange a string so that **no two adjacent characters are the same**. Return any valid result, or `""` if impossible.

**Example:** `"aab"` gives `"aba"`, and `"aaab"` gives `""`.

**Idea (greedy):** At each step, place the **most frequent remaining character** (max-heap by count). But we must not use the same character twice in a row, so we **hold back** the character we just used for one step, and push it back afterwards.

```python
import heapq                                               # import the heap library
from collections import Counter                            # to count character frequencies

def reorganize_string(s):                                  # s: the input string
    freq = Counter(s)                                      # character -> how many times it appears
    heap = [(-cnt, ch) for ch, cnt in freq.items()]        # negative counts make a max-heap by frequency
    heapq.heapify(heap)                                    # build the heap
    result = []                                            # characters of the answer, in order
    prev_cnt, prev_ch = 0, ""                              # the character used in the previous step (held back)
    while heap:                                            # while there are characters left to place
        cnt, ch = heapq.heappop(heap)                      # most frequent available character
        result.append(ch)                                  # place it in the answer
        cnt += 1                                           # we used one copy (cnt is negative, so +1 moves toward 0)
        if prev_cnt < 0:                                   # the previous character still has copies left
            heapq.heappush(heap, (prev_cnt, prev_ch))      # it is safe to use again from the next step
        prev_cnt, prev_ch = cnt, ch                        # hold this character back for one step
    return "".join(result) if len(result) == len(s) else ""   # fewer characters placed means it was impossible

print(reorganize_string("aab"))                            # possible: aba
print(reorganize_string("aaab"))                           # impossible: empty string is printed as a blank line
print(reorganize_string("aabb"))                           # possible: abab
print(reorganize_string("vvvlo"))                          # possible: vlvov
```

**Output:**
```text
aba

abab
vlvov
```

**Why it works:** if the most frequent character appears more than `(n + 1) // 2` times, no arrangement exists, and the loop ends early with fewer characters than needed.

**Complexity:** Time O(n log d) where d is the number of distinct characters (at most 26), Space O(d).

---

### Problem 9: Kth Smallest Element in a Sorted Matrix (Medium)

**Question:** A matrix has every **row sorted** and every **column sorted** (ascending). Find the **k-th smallest** element.

**Example:** `[[1, 5, 9], [10, 11, 13], [12, 13, 15]]`, `k = 8` gives `13`.

**Idea:** The smallest element is the top-left corner. After taking an element, the next candidate in that row is the one to its right. This is exactly **merging k sorted rows**: put the first element of each row in a min-heap, and pop `k` times. Each time you pop `(value, row, col)`, push the next element from the same row.

```python
import heapq                                               # import the heap library

def kth_smallest(matrix, k):                               # matrix: sorted rows and columns, k: rank wanted
    rows, cols = len(matrix), len(matrix[0])               # matrix dimensions
    heap = []                                              # min-heap of (value, row, col)
    for r in range(min(rows, k)):                          # only the first k rows can contain the answer
        heapq.heappush(heap, (matrix[r][0], r, 0))         # push the first element of each such row
    count = 0                                              # how many elements we have popped so far
    while heap:                                            # keep going while candidates exist
        val, r, c = heapq.heappop(heap)                    # the smallest unvisited value
        count += 1                                         # that was the count-th smallest overall
        if count == k:                                     # we reached the rank we need
            return val                                     # this value is the answer
        if c + 1 < cols:                                   # there is another element to the right in this row
            heapq.heappush(heap, (matrix[r][c + 1], r, c + 1))   # push it as the next candidate of this row

matrix = [[1, 5, 9], [10, 11, 13], [12, 13, 15]]           # the sample matrix
print(kth_smallest(matrix, 8))                             # sorted order is 1,5,9,10,11,12,13,13,15
print(kth_smallest(matrix, 1))                             # the smallest is the top-left value
print(kth_smallest([[-5]], 1))                             # a 1 x 1 matrix
```

**Output:**
```text
13
1
-5
```

**Complexity:** Time O(k log min(rows, k)), Space O(min(rows, k)).

---

### Problem 10: Find K Pairs with Smallest Sums (Medium)

**Question:** Two **sorted** arrays `nums1` and `nums2` are given. Pick one number from each to form a pair `(u, v)`. Return the `k` pairs with the **smallest sums**.

**Example:** `nums1 = [1, 7, 11]`, `nums2 = [2, 4, 6]`, `k = 3` gives `[[1, 2], [1, 4], [1, 6]]`.

**Idea:** Think of a grid where cell `(i, j)` is the pair `nums1[i] + nums2[j]`. Rows and columns are sorted, so this is like Problem 9. Start with `(i, 0)` for each `i`, pop the smallest, then push its right neighbour `(i, j + 1)`.

```python
import heapq                                               # import the heap library

def k_smallest_pairs(nums1, nums2, k):                     # two sorted lists and the number of pairs wanted
    if not nums1 or not nums2:                             # nothing to pair
        return []                                          # so the answer is empty
    heap = []                                              # min-heap of (sum, i, j)
    for i in range(min(k, len(nums1))):                    # only the first k elements of nums1 can matter
        heapq.heappush(heap, (nums1[i] + nums2[0], i, 0))  # pair nums1[i] with the smallest of nums2
    result = []                                            # the pairs we will return
    while heap and len(result) < k:                        # until we have k pairs or run out of candidates
        total, i, j = heapq.heappop(heap)                  # the pair with the smallest sum right now
        result.append([nums1[i], nums2[j]])                # record that pair
        if j + 1 < len(nums2):                             # nums1[i] has a next partner in nums2
            heapq.heappush(heap, (nums1[i] + nums2[j + 1], i, j + 1))   # push that next pair
    return result                                          # the k smallest-sum pairs

print(k_smallest_pairs([1, 7, 11], [2, 4, 6], 3))          # sample from the question
print(k_smallest_pairs([1, 1, 2], [1, 2, 3], 2))           # duplicate values give equal pairs
print(k_smallest_pairs([1, 2], [3], 3))                    # fewer than k pairs exist, so we get all of them
```

**Output:**
```text
[[1, 2], [1, 4], [1, 6]]
[[1, 1], [1, 1]]
[[1, 3], [2, 3]]
```

**Complexity:** Time O(k log k), Space O(k).

---

### Problem 11: Task Scheduler (Medium)

**Question:** Each task is a letter. One task runs per time unit (or the CPU idles). The **same** task must be separated by at least `n` time units. Return the **minimum time** to finish all tasks.

**Example:** `["A","A","A","B","B","B"]`, `n = 2` gives `8`: `A B idle A B idle A B`.

**Idea:** Greedy: at every time unit, run the task with the **most remaining copies** (max-heap of counts). After running it, it must **cool down** for `n` units, so keep it in a waiting queue together with the time it becomes available again. When that time arrives, put it back into the heap.

```python
import heapq                                               # import the heap library
from collections import Counter, deque                     # Counter for counts, deque for the cooldown queue

def least_interval(tasks, n):                              # tasks: list of letters, n: cooldown gap
    freq = Counter(tasks)                                  # task -> how many times it must run
    max_heap = [-cnt for cnt in freq.values()]             # negative counts make a max-heap
    heapq.heapify(max_heap)                                # build the heap
    time = 0                                               # current time unit
    cooldown = deque()                                     # queue of (time it is ready again, remaining negative count)
    while max_heap or cooldown:                            # until nothing is ready and nothing is waiting
        time += 1                                          # one more time unit passes
        if max_heap:                                       # some task is ready to run
            cnt = heapq.heappop(max_heap) + 1              # run the most frequent task once (negative count moves toward 0)
            if cnt:                                        # copies of this task remain
                cooldown.append((time + n, cnt))           # it returns to the heap at time + n, so it runs again at time + n + 1
        if cooldown and cooldown[0][0] <= time:            # the earliest waiting task has finished cooling down
            heapq.heappush(max_heap, cooldown.popleft()[1])    # it becomes available to run again
    return time                                            # total time units used (idle units included)

print(least_interval(["A", "A", "A", "B", "B", "B"], 2))   # sample: A B idle A B idle A B
print(least_interval(["A", "A", "A", "B", "B", "B"], 0))   # no cooldown, just 6 tasks
print(least_interval(["A"] * 6 + ["B", "C", "D", "E", "F", "G"], 2))   # many A's force idle time
```

**Output:**
```text
8
6
16
```

**Dry run for the sample (`n = 2`):**

| time | action | heap (counts) | cooling |
|---|---|---|---|
| 1 | run A (3 to 2 left) | B:3 | A ready at 3 |
| 2 | run B (3 to 2 left) | | A at 3, B at 4 |
| 3 | idle, A returns | A:2 | B at 4 |
| 4 | run A (to 1 left), B returns | B:2 | A at 6 |
| 5 | run B (to 1 left) | | A at 6, B at 7 |
| 6 | idle, A returns | A:1 | B at 7 |
| 7 | run A (done), B returns | B:1 | |
| 8 | run B (done) | | |

**Math shortcut (no heap):** `max(len(tasks), (max_count - 1) * (n + 1) + number_of_tasks_with_max_count)`. For the last test: `max(12, 5 * 3 + 1) = 16`. The heap version is what interviewers expect when they want to see the greedy simulation.

**Complexity:** Time O(T log d) where T is the total time and d is the number of distinct tasks (at most 26), Space O(d).

---

### Problem 12: Network Delay Time, Dijkstra's Algorithm (Medium / Hard)

**Question:** A network has `n` nodes (numbered `1..n`). `times[i] = [u, v, w]` means a signal from `u` reaches `v` in `w` time. A signal is sent from node `k`. Return the time for **all** nodes to receive it, or `-1` if some node is unreachable.

**Example:** `times = [[2,1,1],[2,3,1],[3,4,1]]`, `n = 4`, `k = 2` gives `2`.

**Idea:** This is **single-source shortest path with non-negative weights**, so use **Dijkstra**. The min-heap always gives the node with the **smallest known distance**, and that distance is final. The answer is the **largest** of all shortest distances. We use **lazy deletion**: if we pop a node that is already finalized, we simply skip it.

```python
import heapq                                               # import the heap library
from collections import defaultdict                        # dictionary that creates missing lists automatically

def network_delay_time(times, n, k):                       # times: edges, n: node count, k: start node
    graph = defaultdict(list)                              # adjacency list: node -> list of (neighbour, weight)
    for u, v, w in times:                                  # go through every directed edge
        graph[u].append((v, w))                            # store edge u -> v with weight w
    dist = {}                                              # finalized shortest distance of each node
    heap = [(0, k)]                                        # heap of (distance so far, node), start at k with 0
    while heap:                                            # while there are candidates
        d, node = heapq.heappop(heap)                      # candidate with the smallest distance
        if node in dist:                                   # this node was already finalized with a shorter distance
            continue                                       # lazy deletion: skip this stale entry
        dist[node] = d                                     # the first time we pop a node, d is its shortest distance
        for nei, w in graph[node]:                         # look at every outgoing edge
            if nei not in dist:                            # only useful if the neighbour is not finalized yet
                heapq.heappush(heap, (d + w, nei))         # push the distance going through this node
    return max(dist.values()) if len(dist) == n else -1    # the last node to get the signal, or -1 if some never do

print(network_delay_time([[2, 1, 1], [2, 3, 1], [3, 4, 1]], 4, 2))   # sample from the question
print(network_delay_time([[1, 2, 1]], 2, 1))               # simple two-node network
print(network_delay_time([[1, 2, 1]], 2, 2))               # node 1 can never be reached from node 2
```

**Output:**
```text
2
1
-1
```

**Dry run for the sample:** start `(0, 2)`. Pop node 2 (dist 0): push `(1, 1)` and `(1, 3)`. Pop node 1 (dist 1): no outgoing edges. Pop node 3 (dist 1): push `(2, 4)`. Pop node 4 (dist 2). Distances: `{2: 0, 1: 1, 3: 1, 4: 2}`, so the answer is `2`.

**Why Dijkstra fails with negative edges:** the "first pop is final" rule assumes that adding more edges can only make a path longer.

**Complexity:** Time O((V + E) log V), Space O(V + E).

---

### Problem 13: Merge K Sorted Linked Lists (Hard)

**Question:** You are given `k` sorted linked lists. Merge them into one sorted linked list.

**Example:** `[1->4->5, 1->3->4, 2->6]` gives `1->1->2->3->4->4->5->6`.

**Idea:** At any moment, the next smallest value must be the **head of one of the k lists**. Keep those k heads in a min-heap. Pop the smallest, attach it to the result, then push that node's **next** node. The heap never holds more than k items.

```python
import heapq                                               # import the heap library

class ListNode:                                            # one node of a singly linked list
    def __init__(self, val=0, next=None):                  # constructor with defaults
        self.val = val                                     # the value stored in the node
        self.next = next                                   # pointer to the next node (or None)

def build(values):                                         # helper: make a linked list from a Python list
    dummy = ListNode()                                     # dummy head makes appending easy
    cur = dummy                                            # pointer to the last node built so far
    for v in values:                                       # go through every value
        cur.next = ListNode(v)                             # create a node and attach it
        cur = cur.next                                     # move the pointer forward
    return dummy.next                                      # the real head is after the dummy

def to_list(node):                                         # helper: turn a linked list back into a Python list
    out = []                                               # collected values
    while node:                                            # until the end of the list
        out.append(node.val)                               # record this node's value
        node = node.next                                   # move to the next node
    return out                                             # return the values

def merge_k_lists(lists):                                  # lists: a Python list of k list heads
    heap = []                                              # min-heap of (value, list index, node)
    for i, node in enumerate(lists):                       # look at the head of each list
        if node:                                           # skip empty lists
            heapq.heappush(heap, (node.val, i, node))      # i breaks ties, so two nodes are never compared directly
    dummy = ListNode()                                     # dummy head of the merged list
    tail = dummy                                           # pointer to the last node of the merged list
    while heap:                                            # while any list still has nodes
        val, i, node = heapq.heappop(heap)                 # the smallest node among all heads
        tail.next = node                                   # attach it to the merged list
        tail = tail.next                                   # move the tail pointer forward
        if node.next:                                      # the same list has more nodes
            heapq.heappush(heap, (node.next.val, i, node.next))   # its next node becomes that list's new head
    return dummy.next                                      # the head of the merged list

lists = [build([1, 4, 5]), build([1, 3, 4]), build([2, 6])]    # three sorted lists from the question
print(to_list(merge_k_lists(lists)))                       # merge them and show the result
print(to_list(merge_k_lists([])))                          # no lists at all gives an empty result
print(to_list(merge_k_lists([None, build([1])])))          # an empty list mixed with a real one
```

**Output:**
```text
[1, 1, 2, 3, 4, 4, 5, 6]
[]
[1]
```

**Why the index `i` is needed:** if two heads have the same value, Python would try to compare the `ListNode` objects, which are not comparable and raise `TypeError` (see Section 4.6). The unique list index decides the tie first.

**Complexity:** Time O(N log k) where N is the total number of nodes, Space O(k).

---

### Problem 14: Find Median from Data Stream (Hard)

**Question:** Numbers arrive one by one. After each insertion, you must be able to return the **median** of all numbers so far.

**Example:** after adding `1, 2` the median is `1.5`; after adding `3` it is `2`.

**Idea (two heaps):**
- `small`: a **max-heap** holding the **smaller half**.
- `large`: a **min-heap** holding the **larger half**.
- Keep sizes equal (or `small` has one more). Then the median is the root of `small` (odd count) or the average of both roots (even count).

```python
import heapq                                               # import the heap library

class MedianFinder:                                        # supports add_num and find_median
    def __init__(self):                                    # constructor
        self.small = []                                    # max-heap (stored negated) for the smaller half
        self.large = []                                    # min-heap for the larger half

    def add_num(self, num):                                # insert a new number
        heapq.heappush(self.small, -num)                   # step 1: push into the small half (negated)
        heapq.heappush(self.large, -heapq.heappop(self.small))   # step 2: move the biggest of small into large
        if len(self.large) > len(self.small):              # step 3: large must never be bigger than small
            heapq.heappush(self.small, -heapq.heappop(self.large))   # move large's smallest back into small

    def find_median(self):                                 # return the current median
        if len(self.small) > len(self.large):              # odd number of elements
            return float(-self.small[0])                   # the middle one is the top of small (negate back)
        return (-self.small[0] + self.large[0]) / 2.0      # even count: average of the two middle numbers

mf = MedianFinder()                                        # create the structure
for x in [1, 2, 3]:                                        # numbers from the question
    mf.add_num(x)                                          # insert one number
    print(mf.find_median())                                # median after every insertion

mf2 = MedianFinder()                                       # a second, fresh structure
for x in [5, 15, 1, 3]:                                    # a stream that is not sorted
    mf2.add_num(x)                                         # insert one number
    print(mf2.find_median())                               # median after every insertion
```

**Output:**
```text
1.0
1.5
2.0
5.0
10.0
5.0
4.0
```

**Dry run for `5, 15, 1, 3`:** after 5: `[5]`, median 5. After 15: `[5, 15]`, median (5+15)/2 = 10. After 1: `[1, 5, 15]`, median 5. After 3: `[1, 3, 5, 15]`, median (3+5)/2 = 4.

**Why 3 steps for every insert?** Pushing into `small` first and then moving its largest to `large` guarantees that **every element of `small` is less than or equal to every element of `large`**, no matter what value arrives. Step 3 then fixes the sizes.

**Complexity:** `add_num` is O(log n), `find_median` is O(1), Space O(n).

---

### Problem 15: Sliding Window Maximum (Hard)

**Question:** Given an array and a window size `k`, return the **maximum of every window** of `k` consecutive elements as the window slides from left to right.

**Example:** `[1, 3, -1, -3, 5, 3, 6, 7]`, `k = 3` gives `[3, 3, 5, 5, 6, 7]`.

**Idea:** Use a **max-heap of `(-value, index)`**. The root is the biggest value seen so far, but it may have **slipped out of the window**. So before reading the answer, pop the root **while its index is outside the window**. This is **lazy deletion**: we never remove old elements when they leave, only when they reach the top.

```python
import heapq                                               # import the heap library
from collections import deque                              # deque is used in the optimal version below

def max_sliding_window(nums, k):                           # nums: the array, k: window size
    heap = []                                              # max-heap of (negative value, index)
    result = []                                            # maximum of each window
    for i, x in enumerate(nums):                           # i is the right end of the current window
        heapq.heappush(heap, (-x, i))                      # add the new element with its index
        if i >= k - 1:                                     # the first full window has been reached
            while heap[0][1] <= i - k:                     # root's index is left of the window start: it is stale
                heapq.heappop(heap)                        # lazily remove the stale root
            result.append(-heap[0][0])                     # root is the largest value inside the window
    return result                                          # list of window maximums

def max_sliding_window_deque(nums, k):                     # optimal O(n) version (a good follow-up answer)
    dq = deque()                                           # stores indexes; their values are in decreasing order
    result = []                                            # maximum of each window
    for i, x in enumerate(nums):                           # i is the right end of the current window
        while dq and nums[dq[-1]] <= x:                    # back elements are smaller than x: they can never be a max
            dq.pop()                                       # discard them
        dq.append(i)                                       # add the current index at the back
        if dq[0] <= i - k:                                 # front index has left the window
            dq.popleft()                                   # remove it
        if i >= k - 1:                                     # the first full window has been reached
            result.append(nums[dq[0]])                     # front of the deque is the window maximum
    return result                                          # list of window maximums

print(max_sliding_window([1, 3, -1, -3, 5, 3, 6, 7], 3))   # sample from the question (heap version)
print(max_sliding_window([1], 1))                          # single element
print(max_sliding_window([9, 8, 7, 1], 2))                 # decreasing array
print(max_sliding_window_deque([1, 3, -1, -3, 5, 3, 6, 7], 3))   # deque version gives the same answer
```

**Output:**
```text
[3, 3, 5, 5, 6, 7]
[1]
[9, 8, 7]
[3, 3, 5, 5, 6, 7]
```

**Dry run (heap version) for the sample:**

| i | x | window | heap top after cleaning | answer |
|---|---|---|---|---|
| 2 | -1 | `[1, 3, -1]` | 3 (index 1) | 3 |
| 3 | -3 | `[3, -1, -3]` | 3 (index 1, still inside) | 3 |
| 4 | 5 | `[-1, -3, 5]` | 5 (index 4) | 5 |
| 5 | 3 | `[-3, 5, 3]` | 5 (index 4) | 5 |
| 6 | 6 | `[5, 3, 6]` | 6 (index 6) | 6 |
| 7 | 7 | `[3, 6, 7]` | 7 (index 7) | 7 |

**Complexity:** heap version: Time O(n log n), Space O(n). Deque version: Time O(n), Space O(k).

---

## 8. Mistakes, cheat sheet, and extra practice

### 8.1 Top mistakes with heaps in Python

1. **Expecting a max-heap.** `heapq` is min-heap only. Push `-x` and negate when reading (Section 4.4).
2. **Reading `heap[1]` as the second smallest.** Only `heap[0]` is guaranteed. Pop, or use `nsmallest`.
3. **Printing the heap and thinking it is sorted.** Use `sorted(heap)` to look at it in order.
4. **Pushing items that cannot be compared** (dicts, nodes, objects) when priorities tie. Add a unique counter: `(priority, counter, item)`.
5. **Using `heap.sort()` or `list.pop(0)` thinking it keeps the heap valid.** Always use `heappush` and `heappop`.
6. **Building by n pushes when you already have the list.** Use `heapify` (O(n) instead of O(n log n)).
7. **Wrong Top-K direction.** k largest needs a **min**-heap, k smallest needs a **max**-heap.
8. **Forgetting to negate back.** If you stored `-x`, the answer is `-heap[0]`.
9. **Popping from an empty heap.** `heappop([])` raises `IndexError`. Check `if heap:` first.
10. **Sorting the same result again and again.** If you call `sort()` inside a loop to get the minimum, replace it with a heap.

### 8.2 Complexity cheat sheet

| Problem type | Time | Space |
|---|---|---|
| Build heap (`heapify`) | O(n) | O(1) extra |
| Push / pop | O(log n) | |
| Top-K from n items | O(n log k) | O(k) |
| Merge k sorted lists (N total items) | O(N log k) | O(k) |
| Running median | O(log n) per insert, O(1) per median | O(n) |
| Dijkstra (heap) | O((V + E) log V) | O(V + E) |
| Heap sort | O(n log n) | O(n) with `heapq`, O(1) in place |

### 8.3 Pattern templates to memorize

```text
# k largest elements   -> min-heap of size k
for x in nums:
    heapq.heappush(h, x)
    if len(h) > k:
        heapq.heappop(h)

# k smallest elements  -> max-heap (negate) of size k
for x in nums:
    heapq.heappush(h, -x)
    if len(h) > k:
        heapq.heappop(h)

# max-heap of pairs   -> negate the key
heapq.heappush(h, (-count, item))

# Dijkstra core
heap = [(0, start)]
while heap:
    d, u = heapq.heappop(heap)
    if u in dist:
        continue                  # lazy deletion
    dist[u] = d
    for v, w in graph[u]:
        if v not in dist:
            heapq.heappush(heap, (d + w, v))

# two heaps for median
small (max-heap, negated)  |  large (min-heap)
push to small, move top of small to large, rebalance sizes
```

### 8.4 Extra practice (solve after the 15 above)

| Level | Problem | Idea |
|---|---|---|
| Easy | Relative Ranks | heap or sort with indices |
| Easy | The K Weakest Rows in a Matrix | heap of (soldiers, row) |
| Medium | Sort Characters By Frequency | max-heap by count |
| Medium | Furthest Building You Can Reach | min-heap of the biggest jumps |
| Medium | Ugly Number II | min-heap or three pointers |
| Medium | Super Ugly Number | min-heap of multiples |
| Medium | Car Pooling / Maximum Performance of a Team | sort plus heap |
| Medium | Cheapest Flights Within K Stops | Dijkstra variant |
| Medium | Path With Minimum Effort | Dijkstra on a grid |
| Hard | IPO | two heaps (capital and profit) |
| Hard | Smallest Range Covering Elements from K Lists | min-heap plus tracked max |
| Hard | Trapping Rain Water II | min-heap on boundary cells |
| Hard | Sliding Window Median | two heaps with lazy deletion |

### 8.5 Readiness check

You are ready for heap questions in the coding round if you can, without help:

- Explain why the root is the minimum, and why push and pop are O(log n).
- Write `heappush`, `heappop`, `heapify`, and use negation for a max-heap.
- State the Top-K rule (k largest uses a min-heap) and write it in under a minute.
- Write the Dijkstra loop with lazy deletion.
- Draw the two-heap median structure and explain the 3 steps of `add_num`.
- Explain when to add a tie-breaker to avoid `TypeError`.

---


